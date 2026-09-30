# Deploy Next.js ke Vercel

1. Push project ke GitHub.
2. Vercel → Add New → Project → Import repository.
3. Tambahkan environment variables dari `.env.example` (nama sama, nilai
   production/staging sesuai target).
4. Deploy. Vercel mendeteksi Next.js otomatis.

`vercel.json` di folder ini opsional. Copy ke root project kalau perlu region
Singapore (`sin1`). Untuk project biasa, tanpa file ini pun jalan.

Verifikasi:
```bash
curl -I https://URL-VERCEL-KAMU.vercel.app
```
Harus 200. Jangan hardcode URL localhost.
