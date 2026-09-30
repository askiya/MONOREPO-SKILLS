# 08 — Rekomendasi Hosting Santriverse Hybrid

> 🟠 LANJUTAN · Stack: Hermes Agent + GitHub + Cloudflare frontend + Coolify VPS backend/PostgreSQL

> **Ini stack yang Santriverse benar-benar pakai:** Hermes Agent untuk workflow agent, GitHub sebagai sumber kode, Cloudflare untuk frontend, serta Tencent Cloud VPS + Coolify untuk backend dan PostgreSQL.

## Tujuan

Menjalankan frontend dan backend pada dua pipeline deployment yang terpisah, dengan GitHub sebagai sumber kode dan Cloudflare sebagai pengelola domain.

> Harga, kuota, dan fitur provider dapat berubah. Angka di dokumen ini adalah perkiraan awal per September 2026. Periksa halaman harga dan dokumentasi resmi sebelum membeli.

## Arsitektur yang Direkomendasikan

```text
Hermes Agent
    └── mengerjakan dan memverifikasi kode lokal
             └── GitHub private repository
                    ├── Cloudflare Pages/Workers → app.domainmu.com
                    └── Coolify pada VPS
                           ├── backend → api.domainmu.com
                           └── PostgreSQL → network internal

Backup PostgreSQL → Cloudflare R2 atau Backblaze B2
Monitoring → Uptime Kuma + monitoring bawaan Coolify
```

Pisahkan domain, environment variable, log, dan prosedur rollback frontend dari backend. GitHub menjadi sumber kode; database dan secret tidak masuk repository.

## 1. Frontend: Cloudflare Pages atau Workers

Cloudflare cocok untuk frontend karena CDN global, domain custom, SSL, dan integrasi Git. Pilih Pages untuk static site atau alur Pages yang sudah ada. Pilih Workers bila framework atau runtime membutuhkan integrasi Workers.

| Paket | Biaya | Build | Batas file | Bandwidth | Cocok untuk |
|---|---:|---:|---:|---|---|
| Gratis | US$0 | Sampai 500 build/bulan | Sampai 20.000 file per situs | Unlimited sesuai kebijakan platform | Belajar, staging, MVP, frontend produksi ringan |
| Pro | Sekitar US$20/bulan | Sampai 5.000 build/bulan | Periksa batas terbaru | Periksa ketentuan terbaru | Tim dengan frekuensi build lebih tinggi dan fitur Cloudflare tambahan |

Kuota dapat berbeda antara Pages, Workers, dan fitur berbayar lain. Periksa produk yang dipakai; biaya Workers dapat mengikuti request dan waktu CPU.

### Konfigurasi minimum

1. Hubungkan repository GitHub.
2. Pilih branch produksi dan folder frontend.
3. Isi perintah build serta output directory sesuai framework.
4. Set URL API publik, misalnya:

```dotenv
NEXT_PUBLIC_API_URL=https://api.domainmu.com
```

5. Jangan memasukkan secret backend ke variable berawalan `NEXT_PUBLIC_`.
6. Uji preview deployment sebelum mempromosikan ke produksi.
7. Pasang `app.domainmu.com` dan verifikasi HTTPS.

### Alternatif: Vercel

Gunakan Vercel bila frontend memakai Next.js dan membutuhkan integrasi framework serta preview deployment yang lebih langsung. Periksa syarat penggunaan paket gratis untuk proyek komersial dan biaya paket tim terbaru.

Link resmi: [vercel.com](https://vercel.com/) dan [vercel.com/pricing](https://vercel.com/pricing).

## 2. Backend + PostgreSQL: VPS dan Coolify

Coolify gratis untuk self-hosted dan mengelola deployment aplikasi, database, domain, TLS, environment variable, health check, serta log. Biaya utama berasal dari VPS, backup, storage, dan bandwidth.

### Ukuran awal

Untuk Coolify + satu backend + PostgreSQL kecil:

- 2 vCPU;
- 4 GB RAM;
- 40–60 GB SSD;
- Ubuntu LTS;
- swap untuk lonjakan build, bukan pengganti RAM permanen.

Naikkan ukuran berdasarkan metrik CPU, RAM, disk, koneksi DB, dan waktu build. Jangan membuka PostgreSQL ke internet.

### Pilihan VPS

| Provider | Perkiraan biaya awal | Kelebihan | Perhatian | Link resmi |
|---|---:|---|---|---|
| Hetzner Cloud | Mulai sekitar €4/bulan | Harga per resource kompetitif | Region utama Eropa/AS; ukur latency pengguna Indonesia | [hetzner.com/cloud](https://www.hetzner.com/cloud/) |
| DigitalOcean Droplets | Mulai sekitar US$6/bulan | Dokumentasi dan dashboard mudah | Resource murah mungkin belum cukup untuk build + DB | [digitalocean.com/pricing/droplets](https://www.digitalocean.com/pricing/droplets) |
| Vultr Cloud Compute | Mulai sekitar US$6/bulan | Banyak pilihan region | Periksa bandwidth, IPv4, backup, dan harga region | [vultr.com/pricing](https://www.vultr.com/pricing/) |
| Tencent Cloud | Periksa harga region | Region Asia dan opsi yang dipakai Santriverse | Harga promo serta renewal dapat berbeda | [tencentcloud.com/products/cvm](https://www.tencentcloud.com/products/cvm) |
| IDCloudHost | Periksa paket terbaru | Provider Indonesia, pembayaran lokal | Bandingkan resource, SLA, backup, dan support | [idcloudhost.com](https://idcloudhost.com/) |
| Biznet Gio | Periksa paket terbaru | Infrastruktur dan region Indonesia | Periksa jenis instance serta biaya tambahan | [biznetgio.com](https://www.biznetgio.com/) |
| Jagoan Hosting | Periksa paket terbaru | Pembayaran dan dukungan lokal | Pastikan paket memberi akses root dan mendukung Docker | [jagoanhosting.com](https://www.jagoanhosting.com/) |
| DomaiNesia | Periksa paket terbaru | Domain dan layanan server dalam satu provider | Pastikan resource cukup untuk Coolify | [domainesia.com](https://www.domainesia.com/) |
| Dewaweb | Periksa paket terbaru | Dukungan lokal dan pilihan cloud server | Bandingkan SLA, akses root, dan biaya backup | [dewaweb.com](https://www.dewaweb.com/) |

Harga termurah belum tentu memenuhi ukuran awal 2 vCPU/4 GB. Pilih paket berdasarkan spesifikasi, bukan angka “mulai dari”.

### Resource Coolify

Pisahkan minimal dua resource:

1. **Backend** dari repository/branch produksi.
2. **PostgreSQL** dengan volume persisten dan network internal.

Tambahkan:

- health check backend, misalnya `/up`;
- domain `api.domainmu.com`;
- `DATABASE_URL`, `AUTH_SECRET`, dan `FRONTEND_ORIGIN` melalui secret UI;
- backup eksternal;
- resource limit dan alarm disk.

Coolify dapat menjalankan pgAdmin atau tool administrasi database lain. phpMyAdmin khusus MySQL/MariaDB, bukan PostgreSQL. Untuk stack ini gunakan pgAdmin bila UI database diperlukan. Jangan mengekspos pgAdmin ke publik tanpa autentikasi kuat dan pembatasan akses.

## 3. Source Control: GitHub Private

GitHub Free mendukung repository private tanpa batas jumlah repository sesuai kebijakan layanan saat ini.

Aturan minimum:

- aktifkan 2FA;
- lindungi branch produksi;
- wajibkan CI hijau sebelum merge;
- batasi GitHub App Coolify dan Cloudflare hanya ke repository yang diperlukan;
- jangan commit `.env`, credential, backup, atau dump database;
- pisahkan branch/environment staging dan produksi bila keduanya digunakan.

Link resmi: [github.com/pricing](https://github.com/pricing).

## 4. Domain dan DNS: Cloudflare

Gunakan domain frontend dan API terpisah:

| Hostname | Tujuan | Variable terkait |
|---|---|---|
| `app.domainmu.com` | Cloudflare Pages/Workers | URL publik frontend |
| `api.domainmu.com` | Proxy Coolify backend | `NEXT_PUBLIC_API_URL` |
| `status.domainmu.com` | Uptime Kuma opsional | URL status/monitoring |

Urutan aman:

1. Daftarkan custom domain pada resource tujuan.
2. Salin target DNS dari dashboard provider.
3. Buat record DNS Cloudflare.
4. Mulai dengan **DNS only** untuk verifikasi origin.
5. Setelah sertifikat origin valid, gunakan SSL/TLS **Full (strict)** bila proxy aktif.
6. Set `FRONTEND_ORIGIN=https://app.domainmu.com` di backend.
7. Uji preflight dan request dari browser.

Jangan memakai wildcard CORS untuk endpoint yang membawa cookie atau credential.

## 5. Backup PostgreSQL ke R2 atau B2

Jalankan `pg_dump` terjadwal, enkripsi backup bila berisi data sensitif, lalu kirim ke bucket privat Cloudflare R2 atau Backblaze B2.

Contoh alur:

```text
PostgreSQL Coolify
   └── pg_dump format custom
          └── kompresi/enkripsi
                 └── bucket privat R2/B2
                        └── lifecycle retention
```

Kebijakan awal yang masuk akal:

- backup harian;
- retensi 7 backup harian dan 4 backup mingguan;
- minimal satu salinan di luar VPS;
- notifikasi bila job gagal;
- restore drill berkala ke database terpisah.

Status “upload sukses” belum membuktikan backup dapat dipulihkan. Simpan versi PostgreSQL, perintah restore, checksum, dan hasil restore drill.

Link resmi:

- [Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/)
- [Backblaze B2](https://www.backblaze.com/cloud-storage)
- [Dokumentasi pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)

## 6. Monitoring

Gunakan dua lapisan:

| Alat | Fungsi | Penempatan |
|---|---|---|
| Uptime Kuma | Memeriksa frontend, endpoint `/up`, sertifikat, dan notifikasi downtime | Resource terpisah; idealnya dari server berbeda agar tetap memberi alarm saat VPS utama mati |
| Monitoring bawaan Coolify | Status resource, deploy, log, dan pemakaian dasar | VPS Coolify |

Pantau minimal:

- HTTP status dan latency frontend/backend;
- sisa masa berlaku TLS;
- CPU, RAM, swap, disk, dan inode VPS;
- status container serta restart count;
- koneksi, ukuran, dan query lambat PostgreSQL;
- kegagalan backup serta umur backup terakhir;
- kegagalan build Cloudflare dan deploy Coolify.

Link resmi: [uptime.kuma.pet](https://uptime.kuma.pet/) dan [coolify.io/docs](https://coolify.io/docs/).

## 7. Tabel Komponen, Provider, dan Biaya

| Komponen | Provider rekomendasi | Perkiraan biaya | Link resmi |
|---|---|---:|---|
| Workflow agent | Hermes Agent | Sesuai model/provider yang dikonfigurasi | [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs) |
| Source control | GitHub private | Gratis untuk kebutuhan dasar | [github.com/pricing](https://github.com/pricing) |
| Frontend | Cloudflare Pages/Workers | Gratis; Pro sekitar US$20/bulan | [pages.cloudflare.com](https://pages.cloudflare.com/) |
| Alternatif frontend | Vercel | Gratis atau paket berbayar | [vercel.com/pricing](https://vercel.com/pricing) |
| Backend + Coolify | Tencent Cloud VPS | Sesuai region dan paket | [tencentcloud.com/products/cvm](https://www.tencentcloud.com/products/cvm) |
| Alternatif VPS | Hetzner | Mulai sekitar €4/bulan | [hetzner.com/cloud](https://www.hetzner.com/cloud/) |
| Alternatif VPS | DigitalOcean | Mulai sekitar US$6/bulan | [digitalocean.com/pricing/droplets](https://www.digitalocean.com/pricing/droplets) |
| Alternatif VPS | Vultr | Mulai sekitar US$6/bulan | [vultr.com/pricing](https://www.vultr.com/pricing/) |
| Orkestrasi | Coolify self-hosted | Gratis; biaya VPS tetap berlaku | [coolify.io](https://coolify.io/) |
| Database | PostgreSQL container | Termasuk resource VPS | [postgresql.org](https://www.postgresql.org/) |
| Backup | Cloudflare R2 atau Backblaze B2 | Berdasarkan storage, operasi, dan egress | [R2](https://www.cloudflare.com/developer-platform/products/r2/) · [B2](https://www.backblaze.com/cloud-storage) |
| DNS/SSL | Cloudflare | Gratis untuk kebutuhan dasar | [cloudflare.com/plans](https://www.cloudflare.com/plans/) |
| Monitoring | Uptime Kuma | Gratis self-hosted; biaya server tetap berlaku | [uptime.kuma.pet](https://uptime.kuma.pet/) |

Total minimum bukan penjumlahan harga “mulai dari” semata. Hitung paket VPS yang benar-benar memenuhi 2 vCPU/4 GB, object storage, backup, domain, pajak, dan biaya model AI untuk Hermes Agent.

## 8. Catatan Stack Santriverse

Santriverse memakai:

- **Hermes Agent** untuk workflow agent;
- **GitHub** sebagai sumber kode;
- **Cloudflare** untuk frontend, domain, DNS, dan proxy;
- **Tencent Cloud VPS** sebagai server;
- **Coolify** untuk deploy backend dan mengelola PostgreSQL.

Pilihan ini bukan aturan untuk semua proyek. Pilih provider lain bila region, SLA, biaya, dukungan, atau kemampuan operasi lebih sesuai. Pertahankan bentuk arsitekturnya: frontend terpisah, backend/DB terisolasi, source of truth jelas, dan backup berada di luar VPS.

## 9. Peringatan Dua Pipeline Deployment

Frontend Cloudflare dan backend Coolify adalah dua pipeline berbeda. Keduanya dapat gagal secara independen.

Kegagalan umum:

| Gejala | Penyebab umum | Tindakan |
|---|---|---|
| Frontend baru memanggil endpoint lama | `NEXT_PUBLIC_API_URL` salah atau build lama | Periksa variable environment lalu build ulang frontend |
| Request diblokir browser | `FRONTEND_ORIGIN` atau CORS salah | Izinkan hanya origin frontend yang benar dan uji preflight |
| Frontend sukses, API gagal | Deploy Coolify gagal atau health check merah | Rollback frontend atau backend ke pasangan versi kompatibel |
| Backend sukses, migrasi gagal | Urutan deploy salah atau schema tidak kompatibel | Gunakan migrasi backward-compatible dan backup sebelum perubahan destruktif |
| Staging mengubah data produksi | DB atau secret tercampur | Pisahkan resource, credential, domain, dan database |
| Secret terlihat di browser | Secret dimasukkan ke variable publik/build frontend | Rotasi secret dan pindahkan ke backend |

Setiap rilis harus mencatat pasangan versi frontend/backend. Untuk perubahan API, deploy kompatibilitas backend lebih dahulu, lalu frontend. Hapus endpoint lama setelah semua frontend sudah berpindah.

## 10. Verifikasi Setelah Deploy

```bash
curl --fail --head https://app.domainmu.com
curl --fail https://api.domainmu.com/up
curl -i -X OPTIONS https://api.domainmu.com/endpoint \
  -H "Origin: https://app.domainmu.com" \
  -H "Access-Control-Request-Method: GET"
```

Lanjutkan dari browser:

1. Buka frontend produksi.
2. Jalankan alur yang memanggil API.
3. Pastikan request menuju `https://api.domainmu.com`, bukan `localhost` atau staging.
4. Pastikan tidak ada error CORS, mixed content, atau cookie domain.
5. Periksa log Cloudflare dan Coolify.
6. Pastikan monitor Uptime Kuma hijau.
7. Pastikan backup terbaru ada, checksum cocok, dan restore pernah lulus.

## Checklist

### Akun dan source

- [ ] GitHub repository private dan 2FA aktif.
- [ ] Branch produksi dilindungi dan CI wajib hijau.
- [ ] Akses GitHub App dibatasi ke repository yang diperlukan.
- [ ] `.env`, credential, dan dump database tidak masuk repository.

### Frontend Cloudflare

- [ ] Folder, perintah build, dan output directory benar.
- [ ] Variable publik mengarah ke API environment yang tepat.
- [ ] Tidak ada secret di variable frontend.
- [ ] Preview dan produksi berhasil dibuka melalui HTTPS.
- [ ] Kuota build, file, request, dan CPU sudah diperiksa pada dokumentasi terbaru.

### Backend dan database Coolify

- [ ] VPS cukup untuk Coolify, build, backend, dan PostgreSQL.
- [ ] Backend `/up` mengembalikan HTTP 200.
- [ ] PostgreSQL memakai volume persisten dan tidak terbuka ke internet.
- [ ] Secret hanya berada di Coolify.
- [ ] pgAdmin, bila dipasang, dilindungi dan tidak terbuka bebas.
- [ ] Resource limit, alarm disk, dan log aktif.

### Domain dan integrasi

- [ ] `app.domainmu.com` mengarah ke frontend Cloudflare.
- [ ] `api.domainmu.com` mengarah ke backend Coolify.
- [ ] SSL/TLS memakai Full (strict) bila proxy Cloudflare aktif.
- [ ] `FRONTEND_ORIGIN` dan CORS hanya mengizinkan origin yang benar.
- [ ] Cookie, auth callback, dan redirect memakai domain produksi.
- [ ] Pasangan versi frontend/backend tercatat dan rollback pernah diuji.

### Backup dan monitoring

- [ ] `pg_dump` berjalan terjadwal ke R2/B2.
- [ ] Bucket backup privat, terenkripsi bila perlu, dan memiliki retention.
- [ ] Restore drill ke database terpisah pernah lulus.
- [ ] Uptime Kuma memantau frontend, backend, dan TLS.
- [ ] Monitoring Coolify memantau resource dan kegagalan deploy.
- [ ] Harga, batas paket, tanggal renewal, dan budget alert sudah diperiksa.
