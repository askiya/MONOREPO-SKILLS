# 05 — Deploy ke Vercel

## Tujuan

Aplikasi live di URL `.vercel.app`, terhubung ke database Neon produksi,
login berfungsi di internet — bukan hanya localhost.

## Sebelum Mulai

- `npm run build` sukses di localhost ([`04-preview.md`](04-preview.md))
- Project sudah di-push ke GitHub
- Akun Vercel sudah dibuat (login pakai GitHub)

Referensi:
- [`../../docs/09-deploy-gratis/01-deploy-staging.md`](../../docs/09-deploy-gratis/01-deploy-staging.md)
- [`../../deployment-examples/vercel/README.md`](../../deployment-examples/vercel/README.md)

---

## Langkah 1 — Push ke GitHub

Kalau belum ada repo:

```bash
git remote add origin https://github.com/<USERNAME>/<REPO>.git
git branch -M main
git push -u origin main
```

Kalau sudah ada repo, pastikan perubahan terbaru ter-push:

```bash
git add .
git commit -m "chore: siap deploy"
git push
```

Panduan Git dasar: [`../../docs/01-setup-ai-agent/04-git-github-dasar.md`](../../docs/01-setup-ai-agent/04-git-github-dasar.md)

---

## Langkah 2 — Import project di Vercel

1. Buka [vercel.com/new](https://vercel.com/new)
2. Klik **Import** di samping repo yang baru di-push
3. Vercel mendeteksi Next.js otomatis — jangan ubah framework preset
4. Tambahkan environment variables (**WAJIB** sebelum klik Deploy):

| Nama Variable | Nilai | Catatan |
|---|---|---|
| `DATABASE_URL` | Connection string Neon | Sama dengan `.env` lokal |
| `NEXTAUTH_SECRET` | Random string | Sama dengan yang di-generate sebelumnya |
| `NEXTAUTH_URL` | `https://nama-project.vercel.app` | Ganti nanti kalau pakai custom domain |

> ⚠️ **Jangan lewati step env vars.** Deploy tanpa env vars = build mungkin
> berhasil tapi login pasti gagal dan database tidak terhubung.

5. Klik **Deploy**
6. Tunggu build selesai (biasanya 1-3 menit)

---

## Langkah 3 — Verifikasi deploy

Setelah deploy berhasil, Vercel memberi URL `.vercel.app`.

### Cek status

```bash
curl -I https://nama-project.vercel.app
```

**Expected output:** `HTTP/2 200`

### Tes fungsionalitas

| Test | Aksi | Lolos kalau |
|---|---|---|
| Homepage | Buka URL | Halaman tampil, tidak blank |
| Register | Daftar akun baru | User masuk database Neon |
| Login | Login dengan akun tadi | Redirect ke dashboard |
| Protected route | Akses `/dashboard` | Tampil kalau login, redirect kalau tidak |
| API route | Cek Network tab | Semua response bukan 500 |

### Tes dari device lain

Buka URL dari HP (bukan laptop yang sama). Alasan:
- Memastikan CORS dan cookie berfungsi cross-device
- Memastikan responsive beneran, bukan cuma di DevTools

---

## Langkah 4 — Build settings (opsional)

Biasanya Vercel otomatis mendeteksi. Tapi kalau perlu override:

| Setting | Nilai |
|---|---|
| Framework | Next.js |
| Build Command | `npm run build` |
| Output Directory | `.next` (otomatis) |
| Install Command | `npm install` |
| Node.js Version | 18.x (atau sesuai LTS) |

### Region (opsional)

Buat file `vercel.json` di root project untuk set region Singapore:

```json
{
  "regions": ["sin1"]
}
```

Lihat contoh lengkap: [`../../deployment-examples/vercel/vercel.json`](../../deployment-examples/vercel/vercel.json)

---

## Langkah 5 — Auto-deploy branches

Vercel secara default men-deploy:
- `main` branch → URL produksi
- Setiap push / PR branch lain → preview URL unik

Manfaatkan ini: push ke branch feature, cek preview URL, merge ke main baru
jadi produksi.

---

## Kesalahan Umum

| Gejala | Penyebab | Solusi |
|---|---|---|
| Build gagal di Vercel tapi lokal sukses | Env var belum ditambahkan | Cek Settings → Environment Variables |
| `Can't reach database server` | `DATABASE_URL` salah atau belum diset | Copy ulang dari Neon, paste di env vars |
| Login gagal di production | `NEXTAUTH_URL` masih `localhost:3000` | Ganti ke URL `.vercel.app` |
| Halaman blank, 500 error | Prisma binary mismatch | Tambahkan `postinstall: "prisma generate"` di package.json scripts |
| CSS tidak muncul | Tailwind purge salah | Cek `content` di `tailwind.config.ts` |

---

## Checklist

- [ ] Project ter-push ke GitHub
- [ ] Import project di Vercel berhasil
- [ ] 3 env vars sudah diset: `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`
- [ ] Build di Vercel sukses (hijau)
- [ ] URL `.vercel.app` bisa dibuka
- [ ] `curl -I URL` → HTTP/2 200
- [ ] Register berfungsi di production
- [ ] Login berfungsi di production
- [ ] Protected route menolak akses tanpa login
- [ ] Dites dari HP / device lain
