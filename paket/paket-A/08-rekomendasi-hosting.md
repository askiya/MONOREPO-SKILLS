# 08 — Rekomendasi Hosting Paket A

> **Paket A** = Vercel Starter 🟢 PEMULA
> **Stack**: Next.js + Neon PostgreSQL + Vercel

> ⚠️ **Harga dan fitur berubah sewaktu-waktu. Selalu cek situs resmi sebelum membeli.**

---

## 1. Frontend Hosting

### Rekomendasi Utama: Vercel

Vercel adalah platform hosting yang dibuat oleh tim pembuat Next.js — jadi kompatibilitas terjamin 100%.

| Plan | Biaya | Deploy/Hari | Bandwidth | Catatan |
|------|-------|-------------|-----------|---------|
| **Hobby** | Gratis | 100 | 100 GB/bulan | Hanya untuk proyek **non-komersial** (personal, belajar, portofolio) |
| **Pro** | $20/user/bulan | 6.000 | 1 TB/bulan | Boleh untuk proyek **komersial**, analytics, password protection |

**Fitur Vercel yang penting:**
- Automatic HTTPS (SSL gratis)
- Preview deployment untuk setiap branch/PR
- Edge Functions & Serverless Functions
- Integrasi langsung dengan GitHub/GitLab
- Custom domain gratis di semua plan
- Automatic image optimization (next/image)

### Alternatif Frontend Hosting

| Provider | Biaya | Kelebihan | Kekurangan |
|----------|-------|-----------|------------|
| **Cloudflare Pages** | Gratis | 500 build/bulan, unlimited bandwidth, CDN global | Perlu adapter khusus untuk Next.js SSR |
| **Netlify** | Gratis (300 credit/bulan) | Form handling, identity, serverless | Next.js support terbatas dibanding Vercel |
| **GitHub Pages** | Gratis | 100 GB bandwidth/bulan, mudah setup | Hanya **static site** (tidak support SSR/API Routes) |
| **Railway** | Usage-based (~$5/bulan) | Full server, Docker support | Bukan edge deployment, latency lebih tinggi |

---

## 2. Database Hosting

### Rekomendasi Utama: Neon PostgreSQL

Neon adalah serverless PostgreSQL yang bisa **scale-to-zero** — database mati saat tidak dipakai, hemat biaya.

| Plan | Biaya | Storage | Compute | Restore | Catatan |
|------|-------|---------|---------|---------|---------|
| **Free** | $0 | 0.5 GB | 0.25 CU shared | 6 jam | Maks 100 project, cukup untuk development & MVP |
| **Launch** | Usage-based (~$0.10/CU-hour) | 10 GB included | Maks 16 CU | 7 hari | Untuk production kecil-menengah |
| **Scale** | Usage-based (~$0.16/CU-hour) | 50 GB included | Maks 56 CU | 30 hari | Untuk production besar |

**Fitur Neon yang penting:**
- Scale-to-zero (hemat biaya saat idle)
- Branching database (seperti Git branch untuk database)
- Integrasi langsung dengan Vercel (Neon integration di dashboard Vercel)
- Connection pooling built-in (PgBouncer)
- Serverless driver (`@neondatabase/serverless`)

### Alternatif Database Hosting

| Provider | Biaya | Storage | Kelebihan | Kekurangan |
|----------|-------|---------|-----------|------------|
| **Supabase Free** | $0 | 500 MB | Auth, Realtime, Storage built-in, 2 project gratis | **Pause otomatis** setelah 1 minggu idle |
| **Supabase Pro** | $25/bulan | 8 GB | Tidak pause, daily backup | Lebih mahal untuk DB saja |
| **Railway PostgreSQL** | Usage-based (~$1-4/bulan) | Sesuai usage | Mudah setup, Docker support | Tidak ada scale-to-zero |
| **Render PostgreSQL** | Gratis 30 hari → paid | 1 GB (free) | Managed PostgreSQL | Free plan **expired** setelah 30 hari, paid mulai ~$7/bulan |

---

## 3. Domain & DNS

### DNS (Gratis)

| Provider | Biaya | Fitur |
|----------|-------|-------|
| **Cloudflare DNS** | Gratis | Proxy (CDN + DDoS protection), SSL, analytics, caching rules |
| **Vercel DNS** | Gratis | Otomatis saat pakai Vercel, nameserver Vercel |

### Beli Domain

| Ekstensi | Harga Estimasi | Provider Rekomendasi |
|----------|---------------|---------------------|
| `.com` | $8-12/tahun | **Cloudflare Registrar** (at cost, tanpa markup) |
| `.com` | Rp 100-150rb/tahun | Niagahoster, DomaiNesia, IDCloudHost |
| `.my.id` | Rp 10-15rb/tahun | PANDI via registrar (DomaiNesia, Rumahweb) |
| `.web.id` | Rp 15-25rb/tahun | PANDI via registrar |
| `.id` | Rp 200-300rb/tahun | PANDI via registrar |

> 💡 **Tips**: Untuk belajar/MVP, pakai `.my.id` (sangat murah). Untuk bisnis serius, pakai `.com` via Cloudflare Registrar (harga termurah).

---

## 4. Tabel Perbandingan Lengkap

### Frontend

| Provider | Biaya | Cocok Untuk | Link Resmi |
|----------|-------|-------------|------------|
| Vercel Hobby | Gratis | Belajar, portofolio, MVP non-komersial | [vercel.com](https://vercel.com) |
| Vercel Pro | $20/user/bulan | Produksi komersial, tim | [vercel.com/pricing](https://vercel.com/pricing) |
| Cloudflare Pages | Gratis | Static site, Astro, alternatif murah | [pages.cloudflare.com](https://pages.cloudflare.com) |
| Netlify | Gratis (300 credit) | Proyek kecil, form handling | [netlify.com](https://www.netlify.com) |
| GitHub Pages | Gratis | Static site sederhana | [pages.github.com](https://pages.github.com) |

### Database

| Provider | Biaya | Cocok Untuk | Link Resmi |
|----------|-------|-------------|------------|
| Neon Free | $0 | Development, MVP, belajar | [neon.tech](https://neon.tech) |
| Neon Launch | ~$0.10/CU-hour | Production kecil-menengah | [neon.tech/pricing](https://neon.tech/pricing) |
| Supabase Free | $0 | Proyek dengan auth + realtime | [supabase.com](https://supabase.com) |
| Supabase Pro | $25/bulan | Production dengan Supabase ecosystem | [supabase.com/pricing](https://supabase.com/pricing) |
| Railway | ~$1-4/bulan | DB sederhana, cepat setup | [railway.app](https://railway.app) |

---

## 5. Peringatan

1. **Harga bisa berubah** — semua harga di dokumen ini adalah estimasi per September 2026. Selalu cek situs resmi.
2. **Vercel Hobby = non-komersial** — jika menjual produk/jasa, wajib upgrade ke Pro.
3. **Neon Free = 0.5 GB** — cukup untuk ribuan record teks, tapi perhatikan jika menyimpan banyak data.
4. **Supabase Free pause** — database otomatis pause setelah 1 minggu tidak diakses. Harus manual unpause.
5. **Harga renewal domain** — beberapa registrar kasih harga promo tahun pertama, renewal bisa 2-3x lipat.
6. **Backup database** — selalu backup manual sebelum deploy perubahan besar, jangan hanya andalkan auto-backup.

---

## 6. Link Resmi

| Layanan | URL |
|---------|-----|
| Vercel | [vercel.com](https://vercel.com) |
| Vercel Pricing | [vercel.com/pricing](https://vercel.com/pricing) |
| Neon | [neon.tech](https://neon.tech) |
| Neon Pricing | [neon.tech/pricing](https://neon.tech/pricing) |
| Cloudflare Pages | [pages.cloudflare.com](https://pages.cloudflare.com) |
| Cloudflare Registrar | [cloudflare.com/products/registrar](https://www.cloudflare.com/products/registrar/) |
| Supabase | [supabase.com](https://supabase.com) |
| Netlify | [netlify.com](https://www.netlify.com) |
| Railway | [railway.app](https://railway.app) |
| DomaiNesia | [domainesia.com](https://www.domainesia.com) |
| Niagahoster | [niagahoster.co.id](https://www.niagahoster.co.id) |

---

## 7. Checklist Pemilihan Hosting

### Frontend
- [ ] Apakah proyek ini komersial? → Jika ya, Vercel Pro. Jika tidak, Vercel Hobby.
- [ ] Apakah butuh SSR (Server-Side Rendering)? → Jika ya, Vercel atau Railway. Jika tidak, Cloudflare Pages juga bisa.
- [ ] Apakah bandwidth 100 GB/bulan cukup? → Untuk MVP/awal biasanya cukup.
- [ ] Sudah hubungkan repository GitHub ke Vercel?

### Database
- [ ] Apakah 0.5 GB storage cukup? → Untuk awal biasanya cukup.
- [ ] Sudah buat project di Neon dan dapatkan connection string?
- [ ] Sudah pasang `DATABASE_URL` di environment variables Vercel?
- [ ] Sudah test koneksi database dari lokal dan dari Vercel?

### Domain & DNS
- [ ] Sudah beli domain? → Jika budget terbatas, mulai dengan `.my.id`.
- [ ] Sudah pointing DNS ke Vercel? (CNAME `cname.vercel-dns.com`)
- [ ] SSL/HTTPS aktif? (Vercel otomatis)
- [ ] Sudah test akses via domain custom?

### Umum
- [ ] Sudah baca Terms of Service provider yang dipilih?
- [ ] Sudah catat tanggal renewal domain dan hosting?
- [ ] Sudah setup monitoring uptime (contoh: UptimeRobot gratis)?
- [ ] Sudah backup database sebelum go-live?
