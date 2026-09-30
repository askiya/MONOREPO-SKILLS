# 08 — Rekomendasi Hosting Paket E

> ⚠️ **Harga dan fitur berubah sewaktu-waktu.** Angka di bawah adalah patokan awal, bukan penawaran tetap. Periksa halaman harga resmi sebelum membeli.

Paket E memakai **Next.js standalone + PostgreSQL di Coolify + VPS**. Ini stack yang dipakai Santriverse. Stack ini memberi kontrol lebih besar daripada platform serverless, tetapi pemilik wajib mengurus keamanan, backup, pembaruan, dan kapasitas server.

## Arsitektur yang Direkomendasikan

```text
Pengguna
   │ HTTPS
   ▼
Cloudflare DNS + Proxy
   │ Full (strict)
   ▼
VPS Ubuntu LTS + Coolify
   ├── Next.js standalone
   ├── PostgreSQL
   └── Backup terjadwal
          │
          ▼
   R2 / Backblaze B2

GitHub ── webhook ──> Coolify ── build dan deploy otomatis
```

## 1. VPS Provider untuk Coolify

### Hetzner Cloud

- Mulai sekitar €4 per bulan untuk CAX11 ARM, 2 vCPU dan 4 GB RAM.
- Menjadi pilihan populer di komunitas Coolify karena rasio harga dan resource.
- Bandwidth hingga sekitar 20 TB pada paket atau region tertentu.
- Region Eropa, Amerika Serikat, dan Singapura tersedia sesuai stok.
- Arsitektur ARM dapat menimbulkan masalah bila image atau dependency hanya mendukung x86-64.
- Harga diumumkan naik mulai April 2026; periksa harga terbaru sebelum membeli.

### DigitalOcean

- Basic Droplet mulai sekitar US$6 per bulan untuk 1 vCPU dan 1 GB RAM.
- Premium sekitar US$14 per bulan untuk 2 vCPU dan 4 GB RAM.
- Dokumentasi, marketplace, firewall, monitoring, dan ekosistem matang.
- DigitalOcean Spaces tersedia untuk object storage kompatibel S3.
- Paket 1 GB tidak cukup untuk Coolify + build Next.js + PostgreSQL.

### Vultr

- Cloud Compute mulai sekitar US$6 per bulan.
- High Frequency sekitar US$12 per bulan dengan CPU 3 GHz atau lebih.
- Lebih dari 32 lokasi memberi banyak pilihan region.
- Periksa batas bandwidth dan biaya transfer keluar tiap paket.

### Hostinger VPS

- Mulai sekitar US$6 per bulan pada promo tertentu.
- Menyediakan template pemasangan Coolify sekali klik.
- Tier populer dapat menawarkan RAM hingga 8 GB.
- Perhatikan harga perpanjangan setelah periode promo, bukan hanya harga awal.

### Linode / Akamai

- Mulai sekitar US$5 per bulan.
- Dokumentasi baik dan lokasi server beragam.
- Pilih paket minimal 4 GB RAM untuk penggunaan Paket E yang stabil.

### VPS Indonesia

Pilihan lokal berguna bila mayoritas pengguna berada di Indonesia atau ada kebutuhan lokasi data:

- IDCloudHost VPS
- Biznet Gio
- Jagoan Hosting VPS
- Dewaweb Cloud Server
- DomaiNesia Cloud VPS, mulai sekitar Rp80.000 per bulan

Bandingkan vCPU, RAM, storage NVMe, bandwidth, IP publik, snapshot, SLA, support, dan harga perpanjangan. Label “cloud” tidak selalu berarti snapshot dan backup sudah termasuk.

### Tencent Cloud

Santriverse memakai Tencent Cloud. Region Singapura dan Hong Kong dapat memberi latensi baik untuk Asia Tenggara, serta promo pengguna baru sering tersedia. Periksa harga normal setelah promo habis dan pastikan produk yang dipilih menyediakan IP publik, disk memadai, serta bandwidth yang cukup.

## 2. Coolify

Coolify adalah platform deployment open source yang dapat dipasang gratis di VPS. Coolify Cloud terkelola mulai sekitar US$5 per bulan, di luar biaya server atau resource terkait.

Fitur utama:

- deploy aplikasi dari GitHub;
- auto-deploy melalui webhook;
- reverse proxy dan HTTPS otomatis;
- pengelolaan environment variables;
- database serta service berbasis container;
- log build dan log runtime dari panel.

### Kebutuhan Server

Kebutuhan minimum resmi menjadi titik awal, bukan rekomendasi produksi:

- 2 vCPU;
- 2 GB RAM;
- 30 GB disk.

Untuk Next.js + PostgreSQL, gunakan minimal realistis:

- 2 vCPU;
- **4 GB RAM**;
- 40–60 GB SSD/NVMe;
- Ubuntu LTS;
- swap 2 GB untuk menahan lonjakan build.

Coolify sendiri memakai sekitar 300–400 MB RAM dalam kondisi umum. Build Next.js, container, database, dan cache dapat mendorong VPS 2 GB kehabisan memori. Pilih 8 GB RAM bila menjalankan beberapa aplikasi, worker, atau build bersamaan.

## 3. Database di Coolify

### PostgreSQL Container

PostgreSQL dapat dibuat sebagai resource Coolify tanpa lisensi tambahan.

- Gunakan jaringan internal Coolify untuk koneksi aplikasi.
- Jangan membuka port PostgreSQL ke internet kecuali ada kebutuhan khusus dan pembatasan firewall/IP.
- Simpan `DATABASE_URL` di environment variables Coolify, bukan repository.
- Jadwalkan backup dan simpan salinan di luar VPS.
- Jalankan `prisma migrate deploy` untuk migrasi produksi.

### Alat Administrasi Database

`phpMyAdmin` dapat dipasang sebagai resource terpisah di Coolify, tetapi **phpMyAdmin hanya untuk MySQL/MariaDB, bukan PostgreSQL**. Untuk PostgreSQL gunakan pgAdmin, Adminer yang mendukung PostgreSQL, atau `psql`.

Panel admin database tidak perlu dibuka untuk publik. Bila harus tersedia, lindungi dengan HTTPS, autentikasi kuat, pembatasan IP, atau VPN.

### Alternatif Managed

Gunakan database managed bila tidak ingin mengurus update, high availability, dan backup database sendiri:

- Neon untuk PostgreSQL serverless;
- Supabase untuk PostgreSQL plus Auth, Storage, dan API;
- Aiven untuk PostgreSQL managed dengan pilihan region dan fitur operasional.

Database managed menambah biaya, tetapi mengurangi risiko operasional pada VPS tunggal.

## 4. Domain dan DNS: Cloudflare

1. Tambahkan domain ke Cloudflare.
2. Buat record `A` menuju IP publik VPS.
3. Tambahkan domain pada resource aplikasi di Coolify.
4. Tunggu Coolify menerbitkan sertifikat origin.
5. Aktifkan proxy Cloudflare setelah origin dapat diakses.
6. Gunakan mode SSL/TLS **Full (strict)**.
7. Uji HTTPS dan redirect HTTP ke HTTPS.

Jangan gunakan mode Flexible. Mode itu tidak mengenkripsi koneksi Cloudflare ke VPS dan sering memicu redirect loop.

Panduan lengkap tersedia di [`06-domain-ssl.md`](06-domain-ssl.md).

## 5. Backup

Backup PostgreSQL minimum memakai `pg_dump`, lalu hasilnya dikirim ke object storage di luar VPS, misalnya Cloudflare R2 atau Backblaze B2.

```bash
pg_dump "$DATABASE_URL" --format=custom --file="backup_$(date +%Y%m%d_%H%M%S).dump"
```

Prinsip wajib:

- backup otomatis setiap hari;
- enkripsi saat transit dan saat tersimpan;
- retensi harian, mingguan, dan bulanan sesuai kebutuhan;
- salinan berada di luar VPS utama;
- akses object storage memakai credential khusus dengan izin minimum;
- uji restore berkala, bukan hanya memeriksa file backup ada.

Contoh uji restore ke database kosong:

```bash
pg_restore --clean --if-exists --no-owner --dbname="$DATABASE_URL_UJI" backup_YYYYMMDD_HHMMSS.dump
```

Jangan menjalankan uji restore ke database production. Prosedur rinci tersedia di [`07-maintenance.md`](07-maintenance.md).

## Tabel Perbandingan

### VPS Provider

| Provider | Harga awal indikatif | Paket relevan | Kelebihan | Perhatian |
|---|---:|---|---|---|
| Hetzner Cloud | ~€4/bulan | CAX11, 2 vCPU/4 GB | Resource besar, bandwidth tinggi | ARM compatibility, harga naik April 2026 |
| DigitalOcean | US$6/bulan | Basic 1 vCPU/1 GB | Dokumentasi dan ekosistem kuat | Tier awal tidak cukup; 4 GB sekitar US$14 |
| Vultr | US$6/bulan | Cloud Compute | Banyak lokasi | Cek bandwidth dan biaya egress |
| Hostinger VPS | ~US$6/bulan | Tier promo | Template Coolify, tier RAM besar | Harga perpanjangan |
| Linode/Akamai | US$5/bulan | Shared CPU | Stabil, dokumentasi baik | Pilih 4 GB atau lebih |
| VPS Indonesia | ~Rp80.000/bulan | Tergantung provider | Latensi lokal dan pembayaran rupiah | Bandingkan SLA, backup, bandwidth |
| Tencent Cloud | Tergantung promo | Region SG/HK | Cocok untuk Asia; dipakai Santriverse | Cek harga normal setelah promo |

### Ukuran VPS

| Beban | vCPU | RAM | Disk | Rekomendasi |
|---|---:|---:|---:|---|
| Uji Coolify saja | 2 | 2 GB | 30 GB | Bukan untuk production |
| 1 app Next.js + PostgreSQL | 2 | 4 GB | 40–60 GB | Minimum realistis |
| Beberapa app atau worker | 4 | 8 GB | 80 GB+ | Pilihan aman untuk berkembang |
| Trafik/data tinggi | 4+ | 16 GB+ | Sesuai metrik | Ukur CPU, RAM, IOPS, dan DB |

## Rekomendasi Praktis

- **Biaya terendah:** Hetzner 4 GB bila ARM didukung aplikasi.
- **Pengalaman paling mudah:** DigitalOcean atau Hostinger dengan dokumentasi/template kuat.
- **Pengguna mayoritas Indonesia:** bandingkan region Singapura dengan VPS Indonesia memakai pengukuran latensi nyata.
- **Stack Santriverse:** Tencent Cloud + Coolify + Next.js standalone + PostgreSQL.
- **Data penting tetapi tim kecil:** pertimbangkan VPS untuk aplikasi dan PostgreSQL managed terpisah.

## Peringatan Biaya, Keamanan, dan Operasional

- Harga promo dapat naik tajam saat perpanjangan. Catat harga normal sebelum membeli.
- Satu VPS adalah single point of failure. Kerusakan disk, salah konfigurasi, atau akun dibajak dapat mematikan app dan DB sekaligus.
- Snapshot VPS bukan pengganti `pg_dump`; snapshot yang korup atau tidak konsisten dapat gagal dipulihkan.
- Jangan mengekspos port PostgreSQL, Redis, Docker, atau panel admin database ke internet.
- Gunakan SSH key, matikan login root/password setelah akses key teruji, aktifkan firewall, dan pasang Fail2ban.
- Perbarui Ubuntu, Coolify, image container, dan dependency secara terjadwal.
- Pantau CPU, RAM, disk, IOPS, status container, sertifikat, log error, dan uptime eksternal.
- Simpan secret hanya di Coolify. Jangan menaruh `.env`, backup, atau private key di repository.
- Sediakan rencana rollback sebelum deploy dan backup sebelum migrasi destruktif.

## Link Resmi

- [Dokumentasi Coolify](https://coolify.io/docs/)
- [Harga Coolify](https://coolify.io/pricing/)
- [Hetzner Cloud](https://www.hetzner.com/cloud/)
- [Harga DigitalOcean Droplets](https://www.digitalocean.com/pricing/droplets)
- [Harga Vultr](https://www.vultr.com/pricing/)
- [Hostinger VPS](https://www.hostinger.com/vps-hosting)
- [Akamai Cloud / Linode](https://www.linode.com/pricing/)
- [Tencent Cloud](https://www.tencentcloud.com/)
- [Cloudflare R2](https://developers.cloudflare.com/r2/)
- [Backblaze B2](https://www.backblaze.com/cloud-storage/pricing)
- [Dokumentasi PostgreSQL Backup](https://www.postgresql.org/docs/current/backup.html)

## Checklist

- [ ] Region VPS dipilih berdasarkan lokasi pengguna dan kebutuhan data.
- [ ] Harga normal, harga promo, bandwidth, storage, snapshot, dan egress dibandingkan.
- [ ] VPS memiliki minimal 2 vCPU, 4 GB RAM, dan 40 GB disk.
- [ ] Arsitektur CPU sesuai dengan seluruh image dan dependency aplikasi.
- [ ] Ubuntu LTS diperbarui sebelum Coolify dipasang.
- [ ] SSH key, user non-root, firewall, dan Fail2ban aktif.
- [ ] Coolify berjalan dan akun admin pertama sudah diamankan.
- [ ] GitHub App hanya mendapat akses ke repository yang diperlukan.
- [ ] PostgreSQL hanya dapat diakses lewat jaringan internal.
- [ ] Secret tersimpan di Coolify, bukan repository atau log.
- [ ] Domain Cloudflare memakai mode Full (strict).
- [ ] Auto-deploy berhasil diuji.
- [ ] Backup `pg_dump` harian terkirim ke R2 atau B2.
- [ ] Satu uji restore ke database terpisah berhasil.
- [ ] Monitoring CPU, RAM, disk, uptime, dan log aktif.
- [ ] Harga dan spesifikasi diperiksa ulang di link resmi sebelum membeli.
