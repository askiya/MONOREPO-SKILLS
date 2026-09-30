# 08 — Rekomendasi Hosting VPS untuk Docker Manual

> 🟠 LANJUTAN · Stack: Next.js + Express/Fastify + PostgreSQL + Docker Compose + Nginx + Certbot

## Tujuan

Memilih VPS dan layanan pendukung yang cocok untuk menjalankan stack Paket F secara manual, dengan kapasitas cukup, backup di luar server, serta biaya yang dapat diperkirakan.

## Prasyarat

- Sudah memahami Dockerfile, Docker Compose, jaringan container, volume, dan log.
- Siap mengelola update OS, firewall, SSH, Nginx, TLS, database, backup, dan pemulihan sendiri.
- Sudah memperkirakan lokasi mayoritas pengguna, kebutuhan RAM, ruang disk, dan anggaran bulanan.
- Sudah membaca [`05-deploy.md`](05-deploy.md) dan [`07-maintenance.md`](07-maintenance.md).

> Harga, paket, kuota, performa, lokasi pusat data, dan tampilan dashboard dapat berubah. Periksa halaman resmi provider sebelum membeli. Angka pada dokumen ini hanya titik awal perencanaan, bukan janji harga.

## 1. Spesifikasi Awal yang Disarankan

Untuk satu aplikasi Next.js, satu API Express/Fastify, PostgreSQL, Nginx, dan proses backup:

| Beban | vCPU | RAM | Disk | Cocok untuk |
|---|---:|---:|---:|---|
| Belajar/staging | 2 | 4 GB | 40–80 GB SSD/NVMe | Pengguna sedikit, build jarang |
| Produksi kecil | 2–4 | 4–8 GB | 80–160 GB NVMe | Toko kecil, dashboard internal, SaaS awal |
| Build/CI berat | 4+ | 8–16 GB | 160+ GB NVMe | Build image sering, worker, traffic bertambah |

Jangan memilih RAM berdasarkan aplikasi saja. Sisakan ruang untuk PostgreSQL, page cache Linux, proses build, log, dan lonjakan penggunaan. VPS 2 GB dapat berjalan untuk eksperimen, tetapi mudah kehabisan memori saat `next build` dan PostgreSQL aktif bersamaan.

## 2. Rekomendasi Provider VPS

| Provider | Kelebihan utama | Kekurangan/perhatian | Pilih ketika |
|---|---|---|---|
| **Hetzner Cloud** | Harga terhadap performa kuat; NVMe; pilihan x86 dan CAX berbasis ARM | Region lebih terbatas; ARM wajib memakai image multi-arsitektur | Mengejar efisiensi biaya dan nyaman mengelola VPS sendiri |
| **DigitalOcean Droplets** | Dokumentasi ramah; UI sederhana; integrasi Spaces | Harga per resource biasanya lebih tinggi dibanding opsi hemat | Mengutamakan dokumentasi dan pengalaman operasional yang mudah |
| **Vultr High Frequency** | CPU berfrekuensi tinggi, 3 GHz+ pada kelas terkait; banyak lokasi | Harga dan spesifikasi berbeda per region | Build image dan CI/CD sering menjadi bottleneck CPU |
| **Hostinger KVM VPS** | Paket RAM besar relatif murah, termasuk opsi 8 GB | Periksa detail CPU, disk, backup, dan lokasi pada paket terkini | Membutuhkan RAM besar dengan anggaran terbatas |
| **Linode/Akamai** | Dokumentasi matang; tersedia kelas dedicated CPU | Dedicated CPU sekitar US$28/bulan sebagai titik awal historis; cek harga terbaru | Beban CPU stabil dan perlu resource lebih konsisten |
| **Contabo** | Kapasitas RAM/disk besar dengan harga rendah | Performa CPU dan I/O dapat bervariasi; lakukan benchmark setelah provisioning | Kapasitas lebih penting daripada performa konsisten |
| **Provider Indonesia** | Latensi rendah ke pengguna Indonesia; pembayaran dan dukungan lokal | Harga per resource bisa lebih tinggi; kualitas tiap produk berbeda | Mayoritas pengguna di Indonesia atau perlu invoice/dukungan lokal |

### Hetzner CAX ARM

Instance CAX dapat memberi rasio harga/performa bagus untuk Docker. Pada workload build yang kompatibel, ARM dapat mencatat build sekitar 18–22% lebih cepat dibanding kelas pembanding tertentu. Angka ini bukan jaminan universal: hasil bergantung pada jenis instance, cache, dependency, dan arsitektur image.

Sebelum memilih ARM, pastikan:

1. Base image memiliki varian `linux/arm64`.
2. Dependency native Node.js mendukung ARM64.
3. Image pihak ketiga, agent monitoring, dan tool backup mendukung ARM64.
4. Jalankan build dan smoke test nyata sebelum produksi.

Jika salah satu dependency hanya tersedia untuk `amd64`, pilih VPS x86 atau bangun image multi-arsitektur.

### Provider VPS Indonesia

Bandingkan penawaran dari:

- **IDCloudHost**
- **Biznet Gio**
- **Jagoan Hosting**
- **DomaiNesia**
- **Dewaweb**

Jangan memilih hanya berdasarkan label “cloud” atau jumlah RAM. Periksa:

1. Lokasi pusat data dan hasil latensi dari jaringan pengguna.
2. Jenis virtualisasi, kelas CPU, dan apakah vCPU shared atau dedicated.
3. Jenis disk, batas IOPS, bandwidth, dan biaya trafik keluar.
4. Ketersediaan snapshot, backup, rescue console, dan IPv4.
5. SLA, dukungan 24/7, invoice pajak, dan prosedur pemulihan akun.

## 3. Cara Memilih Provider

### Langkah 1 — Tentukan region

Pilih region dekat mayoritas pengguna. Uji latensi dari beberapa ISP, bukan hanya dari laptop sendiri. Untuk aplikasi Indonesia, VPS lokal biasanya memberi latensi lebih rendah; Singapura sering menjadi kompromi antara pilihan provider dan jarak.

### Langkah 2 — Tentukan arsitektur CPU

- Pilih **x86/amd64** untuk kompatibilitas paling luas.
- Pilih **ARM64** jika seluruh image sudah kompatibel dan hasil benchmark lebih baik.
- Pilih **dedicated CPU** bila penggunaan CPU tinggi terus-menerus dan performa shared vCPU tidak stabil.

### Langkah 3 — Uji VPS sebelum migrasi produksi

Setelah provisioning, periksa resource:

```bash
lscpu
free -h
df -h
```

Jalankan build aplikasi dan catat durasi:

```bash
time docker compose build --no-cache
```

Periksa kondisi container setelah dijalankan:

```bash
docker compose up -d
docker compose ps
docker stats --no-stream
```

**Hasil yang diharapkan:** tidak ada container restart berulang, RAM masih memiliki ruang, disk cukup, dan build selesai tanpa `out of memory`.

### Langkah 4 — Mulai bulanan

Gunakan tagihan bulanan saat validasi. Jangan langsung membayar satu atau dua tahun sebelum performa, dukungan, backup, dan alur restore terbukti sesuai.

## 4. Semua Dikelola Manual — Tanpa Coolify

Paket F tidak memakai Coolify. Komponen yang wajib kamu kelola:

| Komponen | Tanggung jawab |
|---|---|
| Docker Engine/Compose | Instalasi, update, jaringan, volume, restart policy |
| Next.js dan API | Dockerfile, health check, build, environment variable |
| PostgreSQL | Versi image, volume, user, tuning, migrasi, backup, restore |
| Nginx | Reverse proxy, header, batas upload, timeout, log |
| Certbot | Penerbitan dan perpanjangan sertifikat TLS |
| Ubuntu/VPS | Patch keamanan, SSH, UFW, Fail2ban, kapasitas disk |
| Deployment | CI/CD, migrasi aman, smoke test, rollback |
| Monitoring | Uptime, CPU, RAM, disk, log, alert |

Jika tanggung jawab ini terlalu berat, pindah ke Paket E (VPS + Coolify) atau Paket G (Railway). Jangan memasang panel tambahan tanpa kebutuhan jelas karena setiap panel menambah service, port, update, dan permukaan serangan.

## 5. PostgreSQL dalam Container

PostgreSQL berjalan sebagai container self-managed dengan persistent volume. Jangan mengekspos port `5432` ke internet. API harus terhubung melalui network internal Docker Compose.

Minimum produksi:

- Gunakan versi image PostgreSQL yang dipatok, bukan `latest`.
- Simpan data di named volume atau bind mount yang jelas.
- Simpan credential sebagai environment secret di server, bukan di Git.
- Batasi resource agar build tidak menghabiskan seluruh RAM.
- Jalankan `pg_dump` terjadwal.
- Salin backup ke lokasi di luar VPS.
- Uji restore berkala ke database terpisah.

Contoh backup manual mengikuti nama service `db`:

```bash
docker compose exec -T db pg_dump -U "$POSTGRES_USER" -Fc "$POSTGRES_DB" > "backup-$(date +%F-%H%M).dump"
```

Verifikasi daftar isi:

```bash
pg_restore --list "backup-YYYY-MM-DD-HHMM.dump"
```

Backup yang hanya berada pada disk VPS tidak melindungi dari kerusakan disk, penghapusan VPS, atau akun provider yang diambil alih.

## 6. Object Storage untuk Backup

| Layanan | Titik awal | Kelebihan | Perhatian |
|---|---|---|---|
| **Cloudflare R2** | 10 GB gratis menurut kuota yang umum dipublikasikan | API kompatibel S3; tanpa biaya egress ke internet pada model R2 | Cek batas operasi dan kuota terbaru |
| **Backblaze B2** | 10 GB gratis menurut kuota yang umum dipublikasikan | Murah dan matang untuk backup | Cek biaya download dan transaksi |
| **Hetzner Storage Box** | Berbayar sesuai kapasitas | Cocok untuk `rsync`, SFTP, Borg, dan backup server | Region dan jalur akses perlu diperiksa |
| **DigitalOcean Spaces** | Sekitar US$5/bulan untuk 250 GB sebagai titik awal | S3-compatible; integrasi mudah dengan ekosistem DigitalOcean | Cek biaya transfer dan harga terbaru |

Pilih satu tujuan di luar provider VPS jika memungkinkan. Terapkan retensi, misalnya harian 7–14 hari dan bulanan 3–6 bulan, sesuai nilai data dan aturan bisnis.

Backup sebaiknya:

1. Dibuat otomatis oleh cron.
2. Dienkripsi sebelum dikirim jika mengandung data sensitif.
3. Dikirim memakai credential khusus dengan izin minimum.
4. Menghasilkan alert bila gagal.
5. Diuji restore, bukan hanya diuji upload.

## 7. pgAdmin atau GUI Database

PostgreSQL tidak memakai phpMyAdmin. Gunakan **pgAdmin** untuk PostgreSQL; phpMyAdmin hanya untuk MySQL/MariaDB.

pgAdmin dapat dijalankan sebagai container bila GUI memang diperlukan. Aturan aman:

- Jangan membuka pgAdmin langsung ke internet tanpa autentikasi tambahan.
- Lebih aman akses melalui SSH tunnel, VPN, atau Cloudflare Access.
- Jangan mengekspos PostgreSQL publik hanya agar GUI bisa terhubung.
- Matikan container GUI saat tidak digunakan.
- Jangan menyimpan password produksi di browser komputer bersama.

Untuk operasi rutin dan troubleshooting, `psql`, `pg_dump`, dan `pg_restore` lebih ringan daripada menjalankan GUI permanen.

## 8. Domain, DNS, dan HTTPS

Gunakan Cloudflare untuk DNS dan proteksi dasar:

1. Tambahkan domain ke Cloudflare.
2. Arahkan record `A` root dan subdomain ke IPv4 VPS.
3. Gunakan record `AAAA` hanya jika IPv6 VPS dan firewall sudah benar.
4. Saat validasi Certbot HTTP-01, pastikan port 80 dapat diakses.
5. Pasang Nginx sebagai reverse proxy.
6. Terbitkan sertifikat dengan Certbot.
7. Uji perpanjangan otomatis:

```bash
sudo certbot renew --dry-run
```

8. Setelah origin HTTPS valid, gunakan mode Cloudflare **Full (strict)**. Jangan gunakan **Flexible** karena koneksi Cloudflare ke origin tidak terenkripsi.

Rujukan langkah lengkap: [`06-domain-ssl.md`](06-domain-ssl.md).

## 9. Rekomendasi Cepat

| Kondisi | Rekomendasi |
|---|---|
| Harga/performa prioritas, nyaman troubleshooting | Hetzner x86 atau CAX ARM setelah uji kompatibilitas |
| Dokumentasi dan UI prioritas | DigitalOcean |
| Build Docker/CI sering dan CPU-bound | Vultr High Frequency atau kelas dedicated CPU |
| Perlu RAM 8 GB dengan anggaran ketat | Bandingkan Hostinger KVM dan Contabo; benchmark dahulu |
| Perlu CPU lebih konsisten | Linode/Akamai dedicated CPU atau kelas dedicated provider lain |
| Pengguna dominan Indonesia | Uji IDCloudHost, Biznet Gio, Jagoan Hosting, DomaiNesia, dan Dewaweb |
| Tidak siap kelola server manual | Jangan pilih Paket F; gunakan Paket E atau G |

## 10. Peringatan Sebelum Membeli

- VPS murah tidak termasuk pekerjaan sysadmin. Waktu patching, monitoring, backup, dan insiden tetap menjadi biaya.
- Snapshot provider bukan pengganti `pg_dump` dan backup eksternal.
- RAID bukan backup.
- Jangan mengekspos Docker socket, PostgreSQL, pgAdmin, atau dashboard internal ke internet.
- Jangan menjalankan seluruh stack sebagai `root` bila image mendukung user non-root.
- Jangan memilih ARM hanya dari klaim benchmark; uji dependency dan image milik sendiri.
- Jangan menaruh backup pada volume yang sama dengan database.
- Pastikan ada ruang disk untuk image lama, layer build, WAL PostgreSQL, log, dan file backup sementara.
- Aktifkan MFA pada akun VPS, DNS, GitHub, dan object storage.
- Simpan kode pemulihan MFA secara offline.

## Checklist Akhir

- [ ] Region dipilih berdasarkan lokasi pengguna dan hasil uji latensi.
- [ ] VPS minimal 2 vCPU, 4 GB RAM, dan disk SSD/NVMe memadai.
- [ ] Arsitektur `amd64` atau `arm64` sudah diuji dengan semua image.
- [ ] Build Docker nyata lulus tanpa kehabisan RAM.
- [ ] Provider menyediakan rescue console dan metode snapshot/backup yang dipahami.
- [ ] PostgreSQL hanya dapat diakses melalui network internal.
- [ ] Persistent volume database sudah ditetapkan.
- [ ] `pg_dump` otomatis berjalan melalui cron.
- [ ] Backup tersalin ke R2, B2, Storage Box, Spaces, atau lokasi eksternal lain.
- [ ] Restore backup sudah diuji.
- [ ] Nginx dan Certbot dikelola manual tanpa Coolify.
- [ ] DNS dikelola di Cloudflare.
- [ ] Mode SSL/TLS Cloudflare memakai **Full (strict)** setelah origin valid.
- [ ] MFA aktif pada semua akun infrastruktur.
- [ ] Biaya VPS, object storage, backup, IPv4, dan trafik keluar sudah dihitung.
