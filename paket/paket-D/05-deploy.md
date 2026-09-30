# 05 — Deploy: Vercel + Supabase Hosted (Paket D)

## Tujuan

Aplikasi Next.js online di Vercel, terhubung ke Supabase hosted, semua
environment variables terisi dengan aman.

---

## Sebelum Mulai

- [ ] Semua tes localhost lulus ([04-preview.md](04-preview.md))
- [ ] `npm run build` berhasil
- [ ] Project sudah di-push ke GitHub
- [ ] Akun Vercel aktif
- [ ] Supabase project aktif

---

## Langkah

### 1. Pastikan Secret Tidak Masuk Git

```bash
git status --short
git check-ignore .env.local
```

**Output yang benar:**
- `.env.local` TIDAK muncul di `git status`
- `git check-ignore` menampilkan `.env.local`

> Kalau `.env.local` pernah ter-commit: hapus dari history, lalu rotate semua
> key di Supabase. Jangan hanya menghapus file — key sudah bocor.

### 2. Import Repository ke Vercel

1. Buka [vercel.com](https://vercel.com)
2. Login pakai GitHub
3. Klik **Add New** → **Project**
4. Pilih repository project kamu
5. Vercel otomatis mendeteksi framework: **Next.js**
6. Jangan klik Deploy dulu — isi env vars di langkah 3

Referensi config: [`../../deployment-examples/vercel/`](../../deployment-examples/vercel/)

### 3. Tambahkan Environment Variables

Di halaman import project → **Environment Variables**:

| Name | Value | Environment |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | dari Supabase Settings → API | Production, Preview, Development |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | dari Supabase Settings → API | Production, Preview, Development |
| `SUPABASE_SERVICE_ROLE_KEY` | dari Supabase Settings → API | **Production saja** (kalau dipakai) |

> ⚠️ `SUPABASE_SERVICE_ROLE_KEY` bypass RLS. Jangan tambahkan kalau aplikasi
> tidak membutuhkannya. Prinsip least privilege: key yang tidak dipakai = tidak perlu ada.

### 4. Deploy

Klik **Deploy**. Tunggu build selesai.

**Output yang benar:**
```text
✓ Build Completed
✓ Deployment Ready
```

Vercel memberi URL seperti:
```text
https://my-supabase-app-abc123.vercel.app
```

### 5. Update Supabase Auth URL Configuration

Di Supabase dashboard → **Authentication** → **URL Configuration**:

| Setting | Nilai |
|---|---|
| Site URL | `https://my-supabase-app-abc123.vercel.app` |
| Redirect URLs | `https://my-supabase-app-abc123.vercel.app/**` |

Tambahkan juga preview pattern:
```text
https://*-username.vercel.app/**
```

> Format wildcard Vercel bisa berubah. Cek dokumentasi Supabase Redirect URLs
> terbaru. Untuk production, selalu pakai URL exact.

### 6. Update OAuth Provider (Kalau Dipakai)

Di Google Cloud Console / GitHub OAuth App:
- Authorized redirect URI **tetap** ke Supabase:
  `https://<PROJECT_REF>.supabase.co/auth/v1/callback`
- Tidak perlu tambahkan Vercel URL di provider OAuth
- Vercel URL ditambahkan di Supabase Redirect URLs (langkah 5)

### 7. Test Production

Buka URL Vercel dan test lengkap:

```text
1. Register user baru (pakai email berbeda dari localhost test)
2. Login
3. Akses dashboard
4. Buat data → cek masuk Supabase
5. Upload avatar → cek masuk Storage
6. Logout → dashboard tidak bisa diakses
7. Test OAuth login (kalau ada)
```

Verifikasi via command:

```bash
curl -I https://my-supabase-app-abc123.vercel.app
```

**Output yang benar:** `HTTP/2 200`

### 8. Cek Function Logs

Kalau ada error:
1. Vercel Dashboard → Project → **Logs**
2. Filter: Errors
3. Buka request yang gagal
4. Jangan tampilkan nilai secret di log

### 9. Config Opsional: vercel.json

Untuk region Singapore, copy config dari deployment example:

```bash
cp ../../deployment-examples/vercel/vercel.json ./vercel.json
```

Isi:

```json
{
  "regions": ["sin1"]
}
```

> Opsional. Untuk project biasa tanpa kebutuhan latency khusus, Vercel otomatis
> pilih region. Region Supabase sebaiknya sama (Singapore) untuk mengurangi latency.

---

## Environment per Tahap

| Environment | Supabase Project | Vercel URL | Kegunaan |
|---|---|---|---|
| Local | Local Supabase / dev project | `localhost:3000` | Coding harian |
| Preview | Staging Supabase (opsional) | `*.vercel.app` | Review PR |
| Production | Production Supabase | custom domain | User asli |

> Untuk MVP kecil, boleh satu Supabase project untuk preview + production.
> Setelah ada user asli, pisahkan staging agar test tidak merusak data production.

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Build Vercel gagal "Missing Supabase URL" | Env vars belum diisi | Vercel → Settings → Environment Variables → tambah, redeploy |
| Login berhasil lalu redirect ke localhost | Site URL Supabase masih localhost | Update Authentication → URL Configuration |
| OAuth error "redirect_uri_mismatch" | Provider OAuth callback salah | Set callback ke URL Supabase `/auth/v1/callback` |
| Production login tidak persisten | Cookie middleware tidak jalan | Cek `middleware.ts` ada di root dan matcher benar |
| Data kosong di production | Terhubung ke Supabase project berbeda | Cek `NEXT_PUBLIC_SUPABASE_URL` di Vercel |
| 500 tapi lokal jalan | Secret server tidak ada di Vercel | Tambahkan env var, redeploy |
| Perubahan env tidak berlaku | Belum redeploy | Vercel → Deployments → Redeploy |

---

## Checklist

- [ ] `.env.local` tidak ada di Git history
- [ ] Repository berhasil diimport ke Vercel
- [ ] `NEXT_PUBLIC_SUPABASE_URL` terisi di Vercel
- [ ] `NEXT_PUBLIC_SUPABASE_ANON_KEY` terisi di Vercel
- [ ] Service role key hanya ditambahkan kalau benar-benar dipakai server-side
- [ ] Build Vercel selesai tanpa error
- [ ] Vercel URL ditambahkan ke Supabase Redirect URLs
- [ ] Register, login, logout berfungsi di production
- [ ] Database read/write berfungsi di production
- [ ] Storage upload berfungsi di production
- [ ] OAuth berfungsi di production (kalau dipakai)
- [ ] `curl -I` mengembalikan HTTP 200
