# 01 — Deploy Gratis (Staging)

## Tujuan

Mendapat URL publik yang bisa dibagikan ke tester, tanpa bayar.

## Pilihan Platform Gratis

| Platform | Cocok untuk | Catatan |
|---|---|---|
| **Vercel** | Next.js | deploy otomatis dari GitHub, gratis untuk project pribadi |
| **Netlify** | SSG, SPA, Next.js adapter | mirip Vercel |
| **Cloudflare Pages** | SSG, SPA, Next.js via adapter | tanpa batas bandwidth tier gratis |
| **Render** | fullstack Node/Python + PostgreSQL gratis | DB gratis batas waktu (cek saat ini) |
| **Railway** | fullstack + DB | kredit gratis terbatas |
| **Fly.io** | container/Docker | mesin kecil gratis |

Harga, batas, dan kebijakan berubah. Cek halaman pricing terkini sebelum memilih.

## Deploy Next.js ke Vercel (Cara Tercepat)

### 1. Push ke GitHub

```bash
git add -A
git status      # cek tidak ada .env / secret
git commit -m "feat: MVP ready for staging"
git push origin main
```

### 2. Sambungkan Vercel

1. Buka https://vercel.com → login via GitHub.
2. **Add New Project** → pilih repo → **Import**.
3. Framework: Next.js (auto-detect).
4. Root Directory: `.` (atau `apps/web` kalau monorepo B).
5. **Environment Variables** → tambahkan dari `.env`:
   - `DATABASE_URL` — pakai DB hosted (Neon/Supabase)
   - `NEXTAUTH_SECRET`
   - `NEXTAUTH_URL` = URL deploy Vercel nanti
   - Variabel lain sesuai kebutuhan
6. **Deploy**.

### 3. Setelah Deploy

- Buka URL Vercel. Cek halaman + alur kritis.
- Cek database: pastikan migrasi berjalan (`npx prisma migrate deploy` manual
  dulu kalau DB baru).
- Vercel otomatis deploy setiap push ke `main`.

### 4. Cek Lingkungan

- API yang butuh DB harus punya URL DB hosted, bukan localhost.
- Webhook payment harus pakai URL staging sementara.
- Pastikan domain/URL di variabel auth cocok.

## Deploy ke Netlify

1. Push ke GitHub.
2. Buka Netlify → **Add new site** → import dari GitHub.
3. Build command: `npm run build`.
4. Publish dir: `.next` / `out` (tergantung mode).
5. Env: tambahkan variabel.
6. Deploy.

## Deploy ke Cloudflare Pages

1. Push ke GitHub.
2. Buka Cloudflare dashboard → **Workers & Pages** → **Create**.
3. Connect to Git → pilih repo.
4. Build command: `npx @cloudflare/next-on-pages`.
5. Tambahkan env variables.
6. Deploy.

## Deploy ke Render

1. Push ke GitHub.
2. Dashboard Render → **New Web Service** → dari GitHub.
3. Runtime: Node.
4. Build command: `npm install && npm run build`.
5. Start command: `npm start`.
6. Env variables.
7. Deploy.

Untuk database: Render → **New PostgreSQL** → salin Internal Database URL ke env.

## Bagi URL

Setelah deploy berhasil, bagikan URL staging ke 3 orang.

Format feedback:
```
Tolong buka [URL]. Coba daftar, login, lihat katalog, dan beli.
Catat hal yang membingungkan, rusak, atau lambat. Screenshot kalau bisa.
```

## Checklist

- [ ] Kode sudah lulus gate testing
- [ ] Push ke GitHub tanpa secret
- [ ] Platform staging terhubung GitHub
- [ ] Env variables terisi
- [ ] URL staging bisa diakses
- [ ] Minimal 3 orang sudah mencoba
