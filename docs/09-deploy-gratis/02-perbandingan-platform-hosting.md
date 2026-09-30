# 02 — Perbandingan Platform Hosting

## Tujuan

Memilih hosting berdasar bentuk aplikasi, bukan ikut tren.

> Harga, kuota, dan free tier sering berubah. Tabel ini menjelaskan jenis
> layanan; cek pricing resmi sebelum membeli.

## A. Static Site / SPA

Cocok untuk HTML/CSS/JS, Astro, Vite, React SPA, Next.js static export.

| Platform | Git deploy | Custom domain | Server API | Cocok untuk |
|---|---:|---:|---:|---|
| GitHub Pages | ya | ya | tidak | dokumentasi, landing statis |
| Cloudflare Pages | ya | ya | Functions | static/SPA global |
| Netlify | ya | ya | Functions | landing, SPA, form |
| Vercel | ya | ya | Functions | Next.js/static |
| Firebase Hosting | CLI/GitHub | ya | via Functions | SPA/Firebase app |
| Surge | CLI | ya | tidak | preview cepat |

## B. Frontend SSR / Fullstack Serverless

| Platform | Stack unggulan | Database built-in | Catatan |
|---|---|---:|---|
| Vercel | Next.js | integrasi pihak lain | paling mulus untuk Next.js |
| Netlify | Next.js/Astro | tidak | Functions + Edge |
| Cloudflare Workers/Pages | Workers runtime | D1/KV/R2 | API Node tertentu perlu adaptasi |
| Firebase App Hosting | web framework | Firestore/SQL integrasi | ekosistem Google |
| AWS Amplify | React/Next.js | AWS ecosystem | kuat, konfigurasi lebih banyak |

Serverless punya batas waktu request, cold start, dan filesystem sementara.
Jangan pilih untuk worker panjang tanpa mengecek batas platform.

## C. Backend / Container PaaS

| Platform | Deploy | Database | Cocok untuk |
|---|---|---|---|
| Render | Git/Docker | PostgreSQL | Node, Python, worker |
| Railway | Git/Docker | PostgreSQL/MySQL/Redis | prototipe fullstack |
| Fly.io | Docker | opsi managed | app dekat pengguna |
| Koyeb | Git/Docker | eksternal | API/container |
| Northflank | Git/Docker | addon | app + jobs |
| Google Cloud Run | container | Cloud SQL | autoscale ke nol |
| Azure App Service | Git/container | Azure DB | ekosistem Microsoft |
| AWS App Runner | container | RDS | ekosistem AWS |
| DigitalOcean App Platform | Git/container | managed DB | PaaS sederhana |

## D. VPS / Cloud Server

| Pilihan | Kontrol | Operasional | Cocok untuk |
|---|---:|---:|---|
| Hetzner/Contabo | penuh | tinggi | biaya rendah, region luar |
| DigitalOcean/Linode/Vultr | penuh | tinggi | docs lengkap |
| AWS Lightsail | penuh | tinggi | VPS sederhana di AWS |
| Google Compute Engine | penuh | tinggi | kebutuhan GCP |
| Azure VM | penuh | tinggi | kebutuhan Azure |
| Biznet/IDCloudHost/Dewaweb | penuh | tinggi | region Indonesia |
| Tencent/Alibaba Cloud | penuh | tinggi | region Asia/cloud besar |

Di VPS kamu mengurus patch OS, firewall, Docker, SSL, database, backup,
monitoring, dan insiden. Panel Coolify/CapRover/Dokku mempermudah deploy, bukan
menghilangkan tanggung jawab keamanan.

## E. Shared Hosting / cPanel

Cocok untuk:
- WordPress/PHP,
- website statis,
- Node.js kecil bila fitur tersedia,
- email domain dan file hosting sederhana.

Tidak cocok untuk:
- Docker,
- worker berat,
- WebSocket tanpa dukungan provider,
- Next.js SSR besar,
- kontrol versi system package.

## F. Database Managed

| Database | Opsi populer |
|---|---|
| PostgreSQL | Neon, Supabase, Railway, Render, Aiven, cloud provider |
| MySQL | PlanetScale/alternatif managed MySQL, Railway, cloud provider |
| Redis | Upstash, Redis Cloud, cloud provider |
| SQLite edge | Cloudflare D1, Turso |
| Document | MongoDB Atlas, Firestore |

## G. Object Storage

Jangan simpan upload user di filesystem serverless/container ephemeral.

| Pilihan | Kompatibilitas |
|---|---|
| Cloudflare R2 | S3 API, egress menarik |
| AWS S3 | standar industri |
| Backblaze B2 | S3 API |
| DigitalOcean Spaces | S3 API |
| Supabase Storage | terintegrasi Supabase |
| Firebase Storage | terintegrasi Firebase |

## Decision Tree

```text
Hanya HTML/CSS/JS?
  Ya → Cloudflare Pages / GitHub Pages / Netlify
  Tidak → Butuh Next.js SSR?
    Ya → Vercel dulu; VPS/Coolify saat butuh kontrol
    Tidak → Backend container?
      Ya → Render/Railway/Cloud Run atau VPS
      Tidak → PHP/WordPress?
        Ya → cPanel
        Tidak → evaluasi runtime khusus
```

## Checklist

- [ ] Runtime aplikasi cocok dengan platform
- [ ] Batas free tier/pricing dicek hari ini
- [ ] Database dan storage dipilih terpisah bila perlu
- [ ] Batas timeout, RAM, disk, dan bandwidth dipahami
- [ ] Jalur pindah platform tersedia
