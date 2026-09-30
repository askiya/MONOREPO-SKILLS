# 04 — Preview: Localhost + Supabase Local Dev (Paket D)

## Tujuan

Website berjalan di localhost. Auth, database, dan storage berfungsi.
Siap untuk deploy.

---

## Sebelum Mulai

- [ ] Build berhasil ([03-build.md](03-build.md))
- [ ] Supabase project aktif di dashboard

---

## Langkah

### 1. Jalankan Development Server

```bash
npm run dev
```

Buka `http://localhost:3000`.

**Output yang benar:** website tampil tanpa error di console browser.

### 2. Test Auth Flow

Urut, satu per satu:

#### Register

1. Buka `/register`
2. Isi nama, email, password (minimal 6 karakter)
3. Klik **Daftar**
4. Kalau email confirmation aktif → cek inbox email, klik link konfirmasi
5. Kalau email confirmation mati → langsung redirect ke `/dashboard`

**Berhasil kalau:** user baru muncul di Supabase dashboard → **Authentication** → **Users**.

#### Login

1. Buka `/login`
2. Isi email + password yang baru didaftar
3. Klik **Login**
4. Redirect ke `/dashboard`

**Berhasil kalau:** halaman dashboard menampilkan nama user.

#### Logout

1. Di `/dashboard`, klik **Logout**
2. Redirect ke `/login`
3. Coba akses `/dashboard` langsung → harus redirect ke `/login`

**Berhasil kalau:** setelah logout, `/dashboard` tidak bisa diakses.

#### OAuth (Kalau Disetup)

1. Buka `/login`
2. Klik **Login dengan Google** (atau provider lain)
3. Ikuti flow OAuth
4. Redirect kembali ke website → `/dashboard`

**Berhasil kalau:** user OAuth muncul di Supabase Users.

### 3. Test Database + RLS

#### Buat Data via UI

1. Login sebagai User A
2. Buat produk/data baru via form
3. Cek di Supabase → **Table Editor** → data ada dengan `user_id` = User A

#### Verifikasi RLS

1. Login sebagai User B (buat akun baru)
2. User B **tidak boleh** bisa edit/delete data milik User A
3. Data publik (SELECT) boleh terlihat untuk semua user

**Cara cek cepat di Supabase dashboard:**

1. **SQL Editor** → jalankan:

```sql
-- Simulasikan user B mencoba update data user A
-- (ganti UUID sesuai user kamu)
SET request.jwt.claims = '{"sub":"UUID_USER_B"}';
UPDATE products SET title = 'Hacked' WHERE user_id = 'UUID_USER_A';
```

**Berhasil kalau:** 0 rows affected (RLS memblokir).

### 4. Test Storage Upload

1. Login
2. Buka halaman profil / upload
3. Pilih file gambar (< 1 MB, format JPG/PNG/WebP)
4. Klik upload
5. Gambar muncul sebagai avatar/preview

**Berhasil kalau:**
- File muncul di Supabase → **Storage** → bucket `avatars` → folder `userId/`
- Kolom `avatar_url` di tabel `profiles` terisi

**Test gagal upload:**
- Upload file > size limit → harus tampilkan error
- Upload file bukan gambar → harus tampilkan error

### 5. Test Responsive

Buka DevTools (`F12` / `Ctrl`+`Shift`+`I`) → toggle device toolbar:
- iPhone SE (375px)
- iPad (768px)
- Desktop (1280px)

Cek semua halaman: landing, login, register, dashboard, profil.

### 6. Supabase Local Development (Opsional tapi Disarankan)

Untuk development tanpa bergantung ke cloud Supabase:

```bash
# Install Supabase CLI
npm install -g supabase

# Inisialisasi
supabase init

# Start local Supabase (butuh Docker)
supabase start
```

**Output yang benar:**
```text
         API URL: http://127.0.0.1:54321
     GraphQL URL: http://127.0.0.1:54321/graphql/v1
          DB URL: postgresql://postgres:postgres@127.0.0.1:54322/postgres
      Studio URL: http://127.0.0.1:54323
    Inbucket URL: http://127.0.0.1:54324
      JWT secret: super-secret-jwt-token
        anon key: eyJ...
service_role key: eyJ...
```

Buat `.env.local.development` terpisah untuk local Supabase, atau ganti
URL di `.env.local` sementara.

> ⚠️ Supabase local butuh Docker Desktop. Kalau Docker belum terinstal
> atau komputer lambat, skip langkah ini — pakai cloud Supabase langsung.

### 7. Test Build

```bash
npm run build
```

**Output yang benar:**
```text
 ✓ Compiling ...
 ✓ Linting and checking validity of types ...
 ✓ Collecting page data ...
 ✓ Generating static pages ...
 ✓ Finalizing page optimization ...

Route (app)                    Size
┌ ○ /                          ...
├ ○ /login                     ...
├ ○ /register                  ...
├ ƒ /dashboard                 ...
├ ƒ /auth/callback             ...
...
```

`ƒ` = dynamic (server-rendered) — normal untuk halaman yang butuh auth.
`○` = static — halaman publik.

### 8. Lint dan Type Check

```bash
npm run lint
npx tsc --noEmit
```

**Berhasil kalau:** tidak ada error. Warning boleh, error tidak boleh.

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| "Invalid API key" di localhost | `.env.local` salah | Copy ulang dari Supabase dashboard |
| Register berhasil tapi tidak bisa login | Email confirmation aktif, belum konfirmasi | Cek inbox (termasuk spam), atau matikan confirmation untuk development |
| RLS tidak memblokir | RLS belum diaktifkan di tabel | `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;` |
| Upload gagal "new row violates policy" | Storage policy belum dibuat | Buat policy (lihat 03-build.md Bagian 4.2) |
| Session hilang setelah refresh | middleware.ts tidak ada/salah | Cek file middleware.ts di root project |
| `supabase start` gagal | Docker tidak jalan | Start Docker Desktop dulu, atau skip local dev |
| `npm run build` error TypeScript | Type Supabase belum di-generate | `npx supabase gen types typescript --project-id <ref> > types/supabase.ts` |

---

## Checklist

- [ ] `npm run dev` — website tampil di localhost
- [ ] Register user baru berhasil
- [ ] Login dengan user tersebut berhasil → redirect ke dashboard
- [ ] Logout berhasil → tidak bisa akses dashboard
- [ ] OAuth login berhasil (kalau disetup)
- [ ] Data yang dibuat tersimpan di Supabase dengan `user_id` benar
- [ ] RLS memblokir akses data orang lain (tested)
- [ ] Upload file ke Storage berhasil
- [ ] Upload file terlalu besar → tampilkan error
- [ ] Responsive di mobile, tablet, desktop
- [ ] `npm run build` berhasil
- [ ] `npm run lint` dan `tsc --noEmit` tidak ada error
