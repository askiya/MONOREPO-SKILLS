# H8 — Rekomendasi Hosting Monorepo Turborepo

> 🟠 LANJUTAN · Stack: Next.js (`apps/web`) + Express (`apps/api`) + `packages/shared`

## Tujuan

Memilih kombinasi hosting yang sesuai untuk frontend, backend, dan PostgreSQL tanpa memaksa semua workspace berjalan pada satu platform.

> Harga, kuota, kebijakan free tier, dan tampilan dashboard dapat berubah. Periksa halaman harga serta dokumentasi resmi provider sebelum membeli.

## Prasyarat

- `npx turbo run lint test build` sukses dari root monorepo.
- Kebutuhan trafik, penyimpanan, lokasi pengguna, dan anggaran bulanan sudah dicatat.
- Repo tersedia di provider Git yang didukung platform tujuan.
- Secret tidak tersimpan dalam repo.
- Domain aktif dan DNS dapat dikelola melalui Cloudflare.

## Arsitektur yang Direkomendasikan

```text
Pengguna
   ├── https://app.domainmu.com
   │      └── Vercel → apps/web
   └── https://api.domainmu.com
          └── Railway/Render → apps/api → PostgreSQL terkelola

Satu repo:
apps/web + apps/api + packages/shared
```

Pisahkan target deploy berdasarkan karakter workload:

- Vercel menjalankan Next.js dan menangani CDN, preview deployment, serta integrasi framework.
- Railway atau Render menjalankan proses Express yang persistent.
- PostgreSQL terkelola mengurangi pekerjaan patch, backup, dan pemulihan database.
- Cloudflare mengelola DNS untuk dua subdomain.

## 1. Frontend `apps/web`: Vercel

Vercel menjadi pilihan utama karena dukungan Next.js, preview deployment, CDN, dan integrasi monorepo.

| Paket | Perkiraan biaya | Cocok untuk | Catatan |
|---|---:|---|---|
| Hobby | Gratis | Belajar, demo, proyek pribadi | Periksa batas penggunaan dan ketentuan penggunaan komersial terbaru |
| Pro | Sekitar US$20/pengguna/bulan | Tim dan aplikasi produksi | Biaya pemakaian tambahan dapat berlaku |

### Konfigurasi minimum

1. Impor repo ke Vercel.
2. Pilih **Framework Preset: Next.js**.
3. Set **Root Directory** ke `apps/web`.
4. Pastikan dependency workspace dari `packages/shared` tetap tersedia saat instalasi.
5. Set `NEXT_PUBLIC_API_URL=https://api.domainmu.com`.
6. Deploy, lalu verifikasi URL `*.vercel.app` sebelum memasang domain.

### Hindari build yang tidak perlu

Aktifkan **Ignored Build Step** dengan `turbo-ignore` agar perubahan yang tidak memengaruhi frontend dapat melewati build:

```bash
npx turbo-ignore
```

Uji aturan ini pada pull request yang hanya mengubah `apps/api`. Jangan melewati build bila perubahan `packages/shared` memengaruhi `apps/web`.

## 2. Backend `apps/api`: Railway atau Render

### Railway

| Item | Keterangan |
|---|---|
| Paket awal | Hobby sekitar US$5/bulan ditambah pemakaian sesuai kebijakan terbaru |
| Kelebihan | Setup cepat, service dan database mudah dihubungkan, cocok untuk eksperimen hingga aplikasi kecil |
| Perhatian | Biaya mengikuti resource; pasang batas anggaran dan pantau pemakaian |

Pilih Railway bila tim mengutamakan setup cepat dan ingin API serta PostgreSQL dalam satu dashboard.

### Render

| Item | Keterangan |
|---|---|
| Paket awal | Starter sekitar US$7/bulan menurut harga yang berlaku saat dokumen ditulis |
| Kelebihan | Biaya service lebih mudah diperkirakan, health check dan deploy Git sederhana |
| Perhatian | Periksa region, batas resource, penyimpanan, dan kebijakan sleep pada paket yang dipilih |

Pilih Render bila tim mengutamakan biaya bulanan service yang lebih tetap.

### Alternatif

- **DigitalOcean App Platform**, mulai sekitar US$5/bulan bergantung komponen dan resource. Cocok bila layanan lain sudah berada di ekosistem DigitalOcean.
- **VPS + Coolify**, cocok bila tim siap mengelola OS, firewall, patch, backup, monitoring, dan insiden. Jangan memilih opsi ini hanya untuk menghemat biaya kecil.

### Konfigurasi backend minimum

1. Hubungkan repo dan pilih branch produksi.
2. Pastikan build berjalan dengan konteks monorepo agar `packages/shared` tersedia.
3. Gunakan perintah build yang telah lulus lokal, misalnya:

```bash
npx turbo run build --filter=@santriverse/api
```

4. Set `NODE_ENV=production`, `DATABASE_URL`, dan `CORS_ORIGIN=https://app.domainmu.com`.
5. Jangan meneruskan `DATABASE_URL` ke frontend.
6. Aktifkan health check pada `/health`.
7. Verifikasi API mengembalikan HTTP 200 sebelum frontend diarahkan ke URL produksi.

## 3. Database PostgreSQL

Pilihan utama:

| Provider | Cocok untuk | Perhatian |
|---|---|---|
| Neon | PostgreSQL serverless, development branch, beban yang dapat berubah | Periksa batas koneksi, cold start, compute, dan penyimpanan |
| Supabase | PostgreSQL dengan dashboard dan fitur platform tambahan | Jangan mengaktifkan fitur tambahan tanpa kebutuhan; periksa batas paket |
| Railway PostgreSQL | Setup dekat dengan Railway API | Pantau volume, backup, egress, dan total biaya proyek |

Untuk tim kecil, PostgreSQL terkelola biasanya lebih aman daripada database self-hosted karena provider menangani banyak tugas infrastruktur. Tanggung jawab tim tetap mencakup migrasi, kontrol akses, retensi backup, uji restore, dan perlindungan credential.

Sebelum memilih:

1. Pilih region sedekat mungkin dengan API.
2. Pastikan koneksi TLS didukung.
3. Periksa backup otomatis dan point-in-time recovery.
4. Periksa batas koneksi dan gunakan connection pooling bila diperlukan.
5. Lakukan restore drill; status “backup sukses” belum membuktikan data dapat dipulihkan.

## 4. Domain dan Cloudflare DNS

Gunakan dua hostname:

| Hostname | Target |
|---|---|
| `app.domainmu.com` | Project Vercel `apps/web` |
| `api.domainmu.com` | Service Railway/Render `apps/api` |

Alur konfigurasi:

1. Daftarkan custom domain pada Vercel dan Railway/Render.
2. Salin target DNS yang diberikan masing-masing platform.
3. Buat record CNAME di Cloudflare.
4. Gunakan **DNS only** selama verifikasi awal.
5. Setelah HTTPS origin valid, gunakan Cloudflare **Full (strict)** bila proxy diaktifkan.
6. Perbarui `NEXT_PUBLIC_API_URL` dan `CORS_ORIGIN`, lalu redeploy.

Jangan menebak target CNAME. Gunakan nilai yang muncul pada dashboard provider. Langkah lengkap tersedia di [`06-domain-ssl.md`](06-domain-ssl.md).

## 5. Tabel Kombinasi Hosting

| Kombinasi | Perkiraan biaya awal/bulan | Kelebihan | Kekurangan | Rekomendasi |
|---|---:|---|---|---|
| Vercel Hobby + Railway Hobby + Railway PostgreSQL | Mulai sekitar US$5 + pemakaian DB/resource | Setup paling cepat, dashboard sederhana | Biaya dapat naik mengikuti pemakaian | Belajar, demo, MVP kecil |
| Vercel Hobby/Pro + Render Starter + Neon | Mulai sekitar US$7, belum termasuk paket Vercel berbayar | Komponen terpisah, biaya API lebih mudah diperkirakan | Tiga dashboard; perlu mengelola koneksi lintas provider | MVP produksi dengan biaya terkendali |
| Vercel Pro + Railway + Neon/Supabase | Mulai sekitar US$20/pengguna + backend/DB | Workflow tim dan preview deployment kuat | Biaya per pengguna dan pemakaian lebih tinggi | Tim kecil yang aktif merilis |
| Vercel + DigitalOcean App Platform + Managed PostgreSQL | Mulai sekitar US$5 untuk komponen app, DB terpisah | Ekosistem infra lebih lengkap | Managed DB dapat menjadi komponen termahal | Produk yang akan tumbuh di DigitalOcean |
| Vercel + VPS/Coolify + PostgreSQL terkelola | Bergantung ukuran VPS dan DB | Kontrol backend lebih besar | Tim menanggung operasi server | Tim yang sudah mampu mengelola VPS |

Angka di atas bukan penawaran tetap. Hitung total biaya dari compute, database, storage, bandwidth/egress, backup, seat tim, domain, dan pajak.

## 6. Cara Memilih

1. Pilih **Vercel + Railway** bila prioritas utama adalah cepat live.
2. Pilih **Vercel + Render** bila biaya API yang lebih tetap lebih penting.
3. Pilih **Vercel + DigitalOcean** bila resource lain sudah berada di DigitalOcean.
4. Pilih **VPS + Coolify** hanya bila ada penanggung jawab patch, backup, alarm, dan pemulihan.
5. Pilih database managed untuk tim kecil kecuali ada kemampuan operasional dan alasan kuat untuk self-host.
6. Mulai dari resource terkecil yang memenuhi pengujian beban, lalu naikkan berdasarkan metrik, bukan tebakan.

## 7. Verifikasi Setelah Deploy

```bash
curl --fail --head https://app.domainmu.com
curl --fail https://api.domainmu.com/health
```

Lanjutkan dengan uji dari browser:

1. Buka frontend produksi.
2. Jalankan satu alur yang memanggil API.
3. Pastikan request menuju `https://api.domainmu.com`, bukan `localhost`.
4. Pastikan tidak ada error CORS.
5. Periksa log backend dan metrik database.

## Checklist

- [ ] Vercel memakai Root Directory `apps/web`.
- [ ] `turbo-ignore` diuji tanpa melewatkan perubahan `packages/shared` yang relevan.
- [ ] Backend memakai Railway, Render, DigitalOcean, atau VPS sesuai kemampuan operasi tim.
- [ ] Build backend dapat mengakses `packages/shared`.
- [ ] PostgreSQL berada di region dekat API.
- [ ] Backup database aktif dan restore pernah diuji.
- [ ] `DATABASE_URL` hanya tersedia pada backend.
- [ ] `app.domainmu.com` mengarah ke Vercel dan HTTPS valid.
- [ ] `api.domainmu.com` mengarah ke backend dan health check memberi HTTP 200.
- [ ] Cloudflare memakai Full (strict) bila proxy aktif.
- [ ] `CORS_ORIGIN` hanya mengizinkan origin frontend produksi.
- [ ] Anggaran dan alarm pemakaian aktif pada semua provider.
- [ ] Harga serta batas paket sudah diperiksa pada dokumentasi resmi terbaru.
