# 03 — Build: Next.js + Supabase (Auth, Storage, RLS) (Paket D)

## Tujuan

Project Next.js terhubung ke Supabase. Auth (register/login/logout) berfungsi.
Database punya RLS. Storage bucket siap terima upload.

---

## Sebelum Mulai

- [ ] Cursor sudah diberi konteks arsitektur ([02-ai-agent.md](02-ai-agent.md))
- [ ] Dokumen perencanaan sudah lengkap ([01-pedoman.md](01-pedoman.md))

---

## Bagian 1: Setup Project

### 1. Buat Project Next.js

```bash
npx create-next-app@latest my-supabase-app --typescript --tailwind --app --no-src-dir
cd my-supabase-app
```

### 2. Install Supabase SDK

```bash
npm install @supabase/supabase-js @supabase/ssr
```

> ⚠️ **JANGAN** install `@supabase/auth-helpers-nextjs` — paket itu deprecated.
> Gunakan `@supabase/ssr` untuk Next.js App Router.

### 3. Buat Project Supabase

1. Buka [supabase.com/dashboard](https://supabase.com/dashboard)
2. Klik **New Project**
3. Isi:
   - Name: nama project
   - Database Password: password kuat (simpan di tempat aman)
   - Region: **Southeast Asia (Singapore)** — paling dekat Indonesia
4. Klik **Create new project**
5. Tunggu setup selesai (1-2 menit)

### 4. Copy Credential ke .env.local

Di Supabase dashboard → **Settings** → **API**:

```bash
# Buat .env.local di root project
```

Isi:

```env
NEXT_PUBLIC_SUPABASE_URL=<DARI_DASHBOARD>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<DARI_DASHBOARD>
SUPABASE_SERVICE_ROLE_KEY=<DARI_DASHBOARD>
```

> ⚠️ `SUPABASE_SERVICE_ROLE_KEY` **BYPASS RLS**. Hanya diakses di server.
> Jangan beri prefix `NEXT_PUBLIC_`. Jangan commit file ini.

### 5. Buat Supabase Client Utilities

Buat `utils/supabase/client.ts`:

```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

Buat `utils/supabase/server.ts`:

```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Dari Server Component — abaikan, middleware handle ini
          }
        },
      },
    }
  )
}
```

Buat `utils/supabase/middleware.ts`:

```typescript
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function updateSession(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) =>
            request.cookies.set(name, value)
          )
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  // Refresh session — PENTING: jangan hapus baris ini
  await supabase.auth.getUser()

  return supabaseResponse
}
```

Buat `middleware.ts` di **root project** (bukan di `app/`):

```typescript
import { type NextRequest } from 'next/server'
import { updateSession } from '@/utils/supabase/middleware'

export async function middleware(request: NextRequest) {
  return await updateSession(request)
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

> Pattern di atas menang di semua tutorial resmi Supabase × Next.js.
> Cek dokumentasi resmi untuk versi terbaru kalau ada perubahan.

---

## Bagian 2: Database + RLS

### 1. Buat Tabel Profiles

Di Supabase dashboard → **SQL Editor**, jalankan:

```sql
-- Tabel profiles (auto-create saat user register)
CREATE TABLE public.profiles (
  id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  full_name TEXT,
  avatar_url TEXT,
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Aktifkan RLS
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

-- Policy: user bisa lihat semua profil (publik)
CREATE POLICY "Profil bisa dilihat semua"
  ON public.profiles FOR SELECT
  USING (true);

-- Policy: user hanya bisa update profilnya sendiri
CREATE POLICY "User update profil sendiri"
  ON public.profiles FOR UPDATE
  USING (auth.uid() = id);

-- Trigger: auto-create profil saat register
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, full_name)
  VALUES (NEW.id, NEW.raw_user_meta_data->>'full_name');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

### 2. Buat Tabel Aplikasi Sesuai PRD

Contoh (sesuaikan dengan PRD kamu):

```sql
CREATE TABLE public.products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  price BIGINT NOT NULL DEFAULT 0,
  image_url TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE public.products ENABLE ROW LEVEL SECURITY;

-- Semua orang bisa lihat produk (publik)
CREATE POLICY "Produk bisa dilihat semua"
  ON public.products FOR SELECT
  USING (true);

-- User hanya bisa insert produk miliknya
CREATE POLICY "User insert produk sendiri"
  ON public.products FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- User hanya bisa update/delete produknya sendiri
CREATE POLICY "User update produk sendiri"
  ON public.products FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "User delete produk sendiri"
  ON public.products FOR DELETE
  USING (auth.uid() = user_id);
```

> ⚠️ **Setiap tabel baru WAJIB:**
> 1. `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`
> 2. Minimal 1 policy
>
> Tabel tanpa RLS = data bisa dibaca siapa saja yang punya anon key.

### 3. Verifikasi RLS Aktif

Di Supabase dashboard → **Table Editor** → klik tabel → kolom **RLS**
harus menunjukkan ✅ Enabled.

---

## Bagian 3: Auth

### 1. Setup Auth Provider

Di Supabase dashboard → **Authentication** → **Providers**:

**Email (default, sudah aktif):**
- Pastikan **Enable Email** aktif
- Opsional: aktifkan **Confirm email** (user harus verifikasi email)

**OAuth (opsional tapi disarankan):**
- Contoh: Google OAuth
- Buat credential di [Google Cloud Console](https://console.cloud.google.com/)
- Copy Client ID + Client Secret ke Supabase
- Redirect URL: `https://<PROJECT_REF>.supabase.co/auth/v1/callback`

> Detail setup OAuth sering berubah. Ikuti panduan terbaru di
> dokumentasi resmi Supabase dan provider OAuth.

### 2. Buat Halaman Auth

Minta Cursor:

```text
@codebase

Buat halaman auth:
1. app/login/page.tsx — form email + password, tombol login, link ke register
2. app/register/page.tsx — form nama + email + password, tombol daftar
3. app/auth/callback/route.ts — handle OAuth callback

Gunakan Server Actions untuk submit form.
Pakai createClient dari utils/supabase/server.ts untuk server-side.
Setelah login/register sukses, redirect ke /dashboard.

Jangan gunakan @supabase/auth-helpers-nextjs.
```

### 3. Buat Protected Route

```text
Buat app/dashboard/page.tsx yang:
1. Cek auth di server component pakai supabase.auth.getUser()
2. Kalau tidak login → redirect ke /login
3. Kalau login → tampilkan nama user dan tombol logout

Logout pakai Server Action yang memanggil supabase.auth.signOut()
lalu redirect ke /login.
```

### 4. Setup Redirect URLs

Di Supabase dashboard → **Authentication** → **URL Configuration**:

| Setting | Nilai |
|---|---|
| Site URL | `http://localhost:3000` (development) |
| Redirect URLs | `http://localhost:3000/**`, `https://domainmu.com/**` |

> Tambahkan URL Vercel nanti setelah deploy.

---

## Bagian 4: Storage

### 1. Buat Storage Bucket

Di Supabase dashboard → **Storage** → **New Bucket**:

| Setting | Nilai |
|---|---|
| Name | `avatars` |
| Public | ❌ No (private — akses via signed URL) |
| File size limit | 1 MB (atau sesuai kebutuhan) |
| Allowed MIME types | `image/jpeg, image/png, image/webp` |

### 2. Buat Storage Policy

```sql
-- Policy: user bisa upload ke folder user_id sendiri
CREATE POLICY "User upload avatar sendiri"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'avatars'
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

-- Policy: user bisa update/delete file sendiri
CREATE POLICY "User update avatar sendiri"
  ON storage.objects FOR UPDATE
  USING (
    bucket_id = 'avatars'
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

CREATE POLICY "User delete avatar sendiri"
  ON storage.objects FOR DELETE
  USING (
    bucket_id = 'avatars'
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

-- Policy: avatar bisa dilihat semua (public read)
CREATE POLICY "Avatar bisa dilihat semua"
  ON storage.objects FOR SELECT
  USING (bucket_id = 'avatars');
```

### 3. Buat Komponen Upload

```text
Buat komponen upload avatar yang:
1. Terima file dari <input type="file" accept="image/*">
2. Upload ke Supabase Storage bucket 'avatars' folder userId/avatar.ext
3. Update kolom avatar_url di tabel profiles
4. Tampilkan preview gambar
5. Tampilkan error kalau file terlalu besar atau format salah
```

---

## Struktur Project Akhir

```text
my-supabase-app/
├── app/
│   ├── page.tsx              ← landing page publik
│   ├── layout.tsx
│   ├── login/page.tsx        ← form login
│   ├── register/page.tsx     ← form register
│   ├── auth/callback/route.ts ← OAuth callback
│   └── dashboard/
│       └── page.tsx           ← protected, setelah login
├── components/
│   ├── Header.tsx
│   ├── AvatarUpload.tsx
│   └── ...
├── utils/supabase/
│   ├── client.ts             ← browser client
│   ├── server.ts             ← server client
│   └── middleware.ts          ← session refresh
├── middleware.ts              ← root middleware
├── .env.local                 ← TIDAK di-commit
├── .env.example               ← template tanpa nilai
├── .cursorrules               ← aturan agent
├── PRD.md
├── ARCHITECTURE.md
└── ...
```

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| "Invalid API key" | `.env.local` salah atau belum ada | Cek SUPABASE_URL dan ANON_KEY dari dashboard |
| "new row violates row-level security" | RLS policy terlalu ketat atau belum ada INSERT policy | Cek policy di SQL Editor |
| Data bisa dilihat semua user | RLS tidak diaktifkan | `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` |
| Login berhasil tapi session hilang | middleware.ts tidak ada | Buat middleware.ts di root (langkah Bagian 1.5) |
| OAuth redirect error | Redirect URL tidak cocok | Tambahkan URL di Supabase Authentication → URL Config |
| Upload gagal "policy" | Storage policy belum dibuat | Buat policy di SQL Editor (Bagian 4.2) |
| `@supabase/auth-helpers-nextjs` error | Paket deprecated | Ganti ke `@supabase/ssr`, ikuti contoh di atas |
| TypeScript error `any` | Tabel belum di-generate | `npx supabase gen types typescript --project-id <ref> > types/supabase.ts` |

---

## Checklist

- [ ] Project Next.js + TypeScript + Tailwind dibuat
- [ ] `@supabase/supabase-js` dan `@supabase/ssr` terinstal
- [ ] Supabase project dibuat di dashboard
- [ ] `.env.local` terisi URL + anon key + service role key
- [ ] `.env.local` ada di `.gitignore`
- [ ] Supabase client utilities dibuat (client, server, middleware)
- [ ] `middleware.ts` ada di root project
- [ ] Tabel `profiles` dibuat dengan trigger auto-create
- [ ] RLS aktif di semua tabel (`ENABLE ROW LEVEL SECURITY`)
- [ ] Policy dibuat di setiap tabel
- [ ] Auth berfungsi: register → login → dashboard → logout
- [ ] Storage bucket `avatars` dibuat dengan policy
- [ ] Upload avatar berfungsi
- [ ] `npm run build` berhasil tanpa error
