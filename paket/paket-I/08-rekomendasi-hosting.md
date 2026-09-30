# I8 — Rekomendasi Hosting Multi-Service Docker

> 🔴 EXPERT · Stack: Next.js + NestJS/Fastify + PostgreSQL + Redis + BullMQ + Docker Compose + Traefik + Uptime Kuma

## Tujuan

Memilih VPS, menentukan ukuran resource, dan menyusun strategi database, monitoring, backup, serta object storage untuk stack multi-service yang berjalan pada satu node.

> Harga, kuota, spesifikasi, dan kebijakan provider dapat berubah. Periksa halaman resmi provider sebelum membeli.

## Prasyarat

- `docker compose config` valid di lokal, semua container sehat.
- Proyeksi beban puncak (concurrent users, jobs per menit, ukuran DB) sudah diperkirakan.
- SSH key tersedia, domain aktif di Cloudflare, email operasional untuk ACME.
- Rencana backup off-site dan penanggung jawab operasi teridentifikasi.

## Arsitektur Target

```text
Internet ── HTTPS ──► Traefik :80/:443 (ACME/Let's Encrypt)
                         ├── app.example.com  → web  (Next.js :3000)
                         ├── api.example.com  → api  (NestJS :4000)
                         └── status.example.com → Uptime Kuma :3001

api ── SQL ──► PostgreSQL (internal)
  │── cache ──► Redis (internal)
  └── job ──► BullMQ/Redis ◄── worker

Semua berjalan di satu VPS. Hanya Traefik membuka port publik.
PostgreSQL, Redis, dan worker tidak punya port publik.
```

## 1. VPS — Pilihan dan Ukuran

### Mengapa VPS

Multi-service membutuhkan proses persistent (API, worker, Redis, PostgreSQL, Traefik, Kuma). Platform serverless dan shared hosting tidak mendukung pola ini. VPS memberikan kontrol penuh, tetapi tim bertanggung jawab atas patch OS, firewall, backup, monitoring, dan pemulihan insiden.

### Provider dan perkiraan harga

| Provider | Seri/Paket | vCPU | RAM | Disk | Perkiraan harga/bulan | Catatan |
|---|---|---:|---:|---:|---:|---|
| Hetzner | CAX21 (ARM) | 4 | 8 GB | 80 GB | Sekitar €7–8 | ARM; pastikan image kompatibel |
| Hetzner | CPX31 (x86) | 4 | 8 GB | 160 GB | Sekitar €14 | x86 standar |
| DigitalOcean | Premium Intel 4 GB | 2 | 4 GB | 80 GB | Sekitar US$14 | Minimum untuk lab; produksi perlu lebih |
| DigitalOcean | Premium Intel 8 GB | 4 | 8 GB | 160 GB | Sekitar US$48 | Produksi awal |
| Vultr | Cloud Compute 4 GB | 2 | 4 GB | 100 GB | Sekitar US$24 | Periksa region terdekat |
| Hostinger | KVM 8 GB | 4–6 | 8 GB | 200 GB | Sekitar US$9 | Periksa uptime SLA dan dukungan |

### Rekomendasi ukuran

| Kebutuhan | RAM minimum | Alasan |
|---|---:|---|
| Lab/belajar | 4 GB | Cukup untuk semua service tanpa beban; swap aktif |
| Produksi ringan (≤ ratusan user aktif) | 8 GB | Ruang untuk DB, cache, worker, dan headroom upgrade |
| Produksi sedang hingga berat | 16 GB+ atau dedicated | Kueri berat, banyak job, atau data besar |

**Peringatan:** Docker mengonsumsi RAM cepat. Setiap container membawa overhead image dan runtime. PostgreSQL dan Redis memerlukan alokasi RAM tersendiri. Ukur dengan `docker stats` dan `free -h` pada beban nyata, bukan hanya saat idle.

### Tips pemilihan

1. Pilih region terdekat dengan mayoritas pengguna.
2. Pastikan provider mendukung snapshot/backup otomatis dan biayanya jelas.
3. Periksa kebijakan bandwidth/egress; beberapa provider membatasi transfer bulanan.
4. Mulai dari spesifikasi yang cukup, lalu upgrade berdasarkan metrik `docker stats`, bukan estimasi.
5. Pilih AMD64/x86 bila ada image Docker yang belum mendukung ARM.
6. Pertimbangkan dedicated server bila beban konsisten dan biaya VPS setara sudah lebih mahal.

## 2. Database

### PostgreSQL

| Opsi | Kelebihan | Kekurangan | Cocok untuk |
|---|---|---|---|
| Container self-managed | Gratis, kontrol penuh, data di volume lokal | Tim bertanggung jawab backup, patch, pemulihan | Tim yang mampu operasi DB |
| Hetzner Cloud Database | Managed, dekat VPS Hetzner, backup otomatis | Biaya tambahan, periksa harga terbaru | Produksi yang ingin delegasi operasi DB |
| DigitalOcean Managed Database | Managed, dashboard terintegrasi | Biaya terpisah dari VPS, periksa region | Produksi di ekosistem DigitalOcean |

Untuk self-managed PostgreSQL:

1. Jalankan sebagai container di Docker Compose.
2. Jangan publikasikan port ke host; gunakan jaringan internal Docker.
3. Gunakan named volume atau bind mount untuk data persisten.
4. Atur backup terjadwal:

```bash
docker compose exec postgres pg_dump -Fc -U <ISI_SENDIRI> <ISI_SENDIRI> > /backups/db-$(date +%Y%m%d-%H%M%S).dump
```

5. Simpan backup terenkripsi ke storage off-site.
6. Uji restore secara berkala; backup yang belum diuji bukan backup.

### Redis

Jalankan Redis sebagai container. Jangan publikasikan port ke host.

Konfigurasi minimum:

- Set `maxmemory` sesuai alokasi RAM yang telah ditentukan.
- Gunakan `maxmemory-policy allkeys-lru` atau sesuai pola akses.
- Aktifkan `requirepass` meskipun jaringan internal; gunakan password acak kuat.

### GUI Database (Opsional)

Jika perlu akses visual:

- **pgAdmin** atau **phpMyAdmin** dapat dijalankan sebagai container dan diakses melalui Traefik dengan autentikasi.
- Jangan ekspos GUI database tanpa autentikasi ke publik.
- Pertimbangkan akses melalui SSH tunnel sebagai alternatif yang lebih aman.

## 3. Monitoring

| Tool | Tipe | Biaya | Fungsi utama |
|---|---|---|---|
| Uptime Kuma | Self-hosted | Gratis | Cek ketersediaan endpoint; notifikasi ke Telegram/email/webhook |
| Grafana + Loki | Self-hosted | Gratis | Dashboard metrik dan log terpusat; perlu resource tambahan di VPS |
| Better Stack | Managed | Free tier tersedia, periksa batas | Uptime, log, incident; tidak menambah beban VPS |

### Rekomendasi

1. **Wajib:** Uptime Kuma untuk status endpoint. Pasang sebagai container, rutekan lewat Traefik, dan hubungkan notifikasi ke kanal tim.
2. **Disarankan:** Grafana + Loki bila tim mampu mengelola stack observability tambahan. Perlu RAM ekstra; hitung sebelum menginstal.
3. **Alternatif:** Better Stack atau layanan serupa bila tim tidak ingin mengelola monitoring sendiri.

Pastikan alarm monitoring sampai ke orang yang bertanggung jawab dan diuji sebelum produksi.

## 4. Object Storage untuk Backup

| Provider | Layanan | Perkiraan biaya | Catatan |
|---|---|---|---|
| Cloudflare | R2 | Gratis egress; penyimpanan berbayar sesuai volume | Tidak mengenakan biaya egress |
| Backblaze | B2 | Gratis 10 GB; penyimpanan dan egress murah | Egress gratis melalui Cloudflare CDN |
| DigitalOcean | Spaces | Mulai sekitar US$5/bulan | 250 GB termasuk, periksa biaya transfer |
| Hetzner | Storage Box | Mulai sekitar €3–4/bulan | SFTP/rsync, cocok untuk backup sederhana |

### Strategi backup

1. Dump PostgreSQL terjadwal (misalnya setiap 6 jam atau sesuai RPO).
2. Enkripsi dump sebelum upload.
3. Upload ke object storage off-site menggunakan `rclone`, `s3cmd`, atau CLI provider.
4. Simpan setidaknya 7 hari retensi.
5. Uji restore dari object storage ke environment terpisah secara berkala.
6. Pastikan credential object storage tersimpan sebagai secret, bukan di repo.

## 5. Domain dan DNS

Gunakan Cloudflare DNS. Semua hostname mengarah ke IP VPS yang sama.

| Hostname | Tipe | Target | Tujuan |
|---|---|---|---|
| `app.example.com` | A | IP VPS | Next.js frontend |
| `api.example.com` | A | IP VPS | NestJS/Fastify API |
| `status.example.com` | A | IP VPS | Uptime Kuma |

Traefik membedakan service berdasarkan `Host()` rule di label Docker. Satu IP melayani semua hostname. TLS diterbitkan otomatis melalui Let's Encrypt HTTP-01 challenge.

- Gunakan **DNS only** selama verifikasi awal.
- Setelah sertifikat valid, aktifkan proxy Cloudflare dan set SSL/TLS ke **Full (strict)**.
- Jangan gunakan **Flexible**; menyebabkan redirect loop.

Konfigurasi lengkap tersedia di [`06-domain-ssl.md`](06-domain-ssl.md).

## 6. Tabel VPS Sizing untuk Multi-Service

| Komponen | RAM perkiraan idle | RAM perkiraan beban ringan | Catatan |
|---|---:|---:|---|
| Next.js (web) | 150–250 MB | 300–500 MB | Bergantung jumlah route dan SSR |
| NestJS/Fastify (api) | 100–200 MB | 200–400 MB | Bergantung middleware dan koneksi |
| Worker (BullMQ) | 80–150 MB | 150–300 MB | Bergantung jumlah dan jenis job |
| PostgreSQL | 200–400 MB | 500 MB–2 GB | Bergantung shared_buffers dan kueri |
| Redis | 50–100 MB | 100–500 MB | Bergantung dataset dan maxmemory |
| Traefik | 30–50 MB | 50–80 MB | Ringan |
| Uptime Kuma | 80–120 MB | 100–150 MB | Bergantung jumlah monitor |
| **Total perkiraan** | **~700 MB – 1.3 GB** | **~1.4 – 3.9 GB** | Belum termasuk OS dan overhead |

Angka di atas adalah perkiraan kasar. Ukur dengan `docker stats` pada beban nyata. Sisakan minimal 1–2 GB untuk OS, buffer I/O, dan headroom darurat. VPS 4 GB cukup untuk lab; **8 GB minimum untuk produksi ringan**.

## 7. Peringatan Penting

1. **Docker mengonsumsi RAM cepat.** Setiap container membawa overhead. Jangan menambah service tanpa menghitung dampak pada total RAM.
2. **Self-managed berarti tanggung jawab penuh.** Patch OS, update Docker, rotasi log, alarm disk, backup, dan restore adalah tugas tim — bukan provider.
3. **Jangan skip backup drill.** Backup otomatis yang belum pernah di-restore bukan jaminan keamanan data.
4. **Jangan ekspos port internal.** PostgreSQL, Redis, dan worker hanya boleh diakses melalui jaringan internal Docker.
5. **Monitor sebelum produksi.** Pastikan alarm berfungsi dan sampai ke orang yang bertanggung jawab sebelum pengguna pertama masuk.
6. **Satu node bukan HA.** Paket ini tidak mencakup failover otomatis. Siapkan prosedur manual untuk downtime dan komunikasikan ke stakeholder.

## Checklist

- [ ] VPS dipilih dengan RAM minimal 4 GB (lab) atau 8 GB (produksi).
- [ ] Region VPS dekat dengan mayoritas pengguna.
- [ ] Snapshot/backup VPS aktif dan biaya diketahui.
- [ ] PostgreSQL berjalan tanpa port publik; backup terjadwal aktif.
- [ ] Redis berjalan tanpa port publik; `maxmemory` dan `requirepass` dikonfigurasi.
- [ ] Uptime Kuma atau monitoring lain aktif dan alarm terhubung ke kanal tim.
- [ ] Backup database terenkripsi tersimpan di object storage off-site.
- [ ] Restore drill pernah dijalankan dan berhasil.
- [ ] DNS Cloudflare mengarah ke IP VPS; SSL/TLS Full (strict) bila proxy aktif.
- [ ] Traefik menangani TLS untuk semua hostname publik.
- [ ] RAM terukur dengan `docker stats` pada beban nyata; headroom tersisa.
- [ ] Prosedur rollback dan eskalasi insiden tertulis.
- [ ] Harga serta batas paket sudah diperiksa pada dokumentasi resmi terbaru.

➡️ Kembali ke **[README.md](README.md)** untuk peta lengkap paket.
