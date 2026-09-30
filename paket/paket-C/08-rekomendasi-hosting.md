# 08 — Rekomendasi Hosting Paket C

> **Paket C** = cPanel Shared 🟢 PEMULA
> **Stack**: Next.js export + PHP + MySQL + cPanel

> ⚠️ **Harga dan fitur berubah sewaktu-waktu. Selalu cek situs resmi sebelum membeli.**

---

## 1. Shared Hosting cPanel Indonesia

Paket C menggunakan cPanel shared hosting karena stack-nya membutuhkan PHP, MySQL, dan phpMyAdmin. Berikut perbandingan provider Indonesia yang populer:

### DomaiNesia

- **Harga mulai**: Rp 16.500/bulan
- **Panel**: cPanel
- **Web Server**: LiteSpeed
- **Storage**: NVMe SSD
- **Database**: MySQL + PostgreSQL + phpMyAdmin
- **Fitur tambahan**: Node.js App Setup, SSH access, Git integration, AutoSSL (Let's Encrypt)
- **Catatan**: Salah satu yang paling lengkap untuk Paket C karena support Node.js di cPanel

### Niagahoster

- **Harga mulai**: Rp 19.900/bulan
- **Panel**: hPanel (**bukan cPanel** — panel custom Niagahoster/Hostinger)
- **Web Server**: LiteSpeed
- **Database**: MySQL + phpMyAdmin
- **Fitur tambahan**: SSH, Git, WordPress manager
- **Catatan**: hPanel berbeda dari cPanel. Jika butuh cPanel standar, ini bukan pilihan yang tepat

### Hostinger Indonesia

- **Harga mulai**: Rp 15.900/bulan (promo)
- **Panel**: hPanel (**bukan cPanel**)
- **Web Server**: LiteSpeed
- **Database**: MySQL + phpMyAdmin
- **Catatan**: Harga promo menarik, tapi **renewal naik 2-3x lipat**. Periksa harga perpanjangan sebelum beli

### IDCloudHost

- **Harga mulai**: Rp 15.000/bulan
- **Panel**: cPanel
- **Database**: MySQL + phpMyAdmin
- **Fitur tambahan**: Pay-as-you-go, SSH
- **Catatan**: Opsi murah dengan cPanel asli

### Dewaweb

- **Harga mulai**: Rp 40.000/bulan
- **Panel**: cPanel
- **Infrastruktur**: Google Cloud Platform
- **Database**: MySQL + phpMyAdmin
- **Sertifikasi**: ISO 27001
- **Catatan**: Lebih mahal tapi infrastruktur enterprise dan keamanan tersertifikasi

### Rumahweb

- **Harga mulai**: Rp 16.000/bulan
- **Panel**: cPanel
- **Database**: MySQL + phpMyAdmin
- **Catatan**: Provider lokal yang sudah lama beroperasi

### AntMediaHost

- **Harga mulai**: Rp 18.000/bulan
- **Panel**: cPanel
- **Storage**: NVMe SSD
- **Database**: MySQL + phpMyAdmin
- **Catatan**: Alternatif cPanel dengan storage cepat

---

## 2. Yang Harus Dicek Saat Beli Hosting cPanel

Sebelum membeli, pastikan hosting menyediakan fitur berikut:

| Fitur | Wajib? | Alasan |
|-------|--------|--------|
| **cPanel** (bukan custom panel) | ✅ Sangat dianjurkan | Standar industri, tutorial banyak, mudah migrasi |
| **phpMyAdmin** | ✅ Wajib | Untuk kelola database MySQL via browser |
| **MySQL / MariaDB** | ✅ Wajib | Database utama Paket C |
| **PostgreSQL** | ⭐ Bonus | Berguna jika ingin migrasi ke Paket A nanti |
| **Setup Node.js App** | ⭐ Bonus | Untuk menjalankan Next.js SSR di cPanel (jika tidak pakai static export) |
| **SSH Access** | ✅ Sangat dianjurkan | Untuk deploy via command line, npm install, troubleshooting |
| **AutoSSL / Let's Encrypt** | ✅ Wajib | HTTPS gratis otomatis |
| **Git Integration** | ⭐ Bonus | Deploy langsung dari repository Git |
| **Harga renewal** | ⚠️ Periksa | Jangan tertipu harga promo tahun pertama |

> ⚠️ **PENTING**: Beberapa provider (Niagahoster, Hostinger) menggunakan panel custom (hPanel), bukan cPanel. Panel custom bisa berbeda fitur dan tidak kompatibel dengan tutorial cPanel standar.

---

## 3. Database Management

### phpMyAdmin (MySQL/MariaDB)

phpMyAdmin adalah tools web untuk mengelola database MySQL yang tersedia di hampir semua hosting cPanel.

**Cara akses:**
1. Login ke cPanel
2. Cari bagian **Databases**
3. Klik **phpMyAdmin**

**Yang bisa dilakukan:**
- Buat, edit, hapus database dan tabel
- Import/export data (SQL, CSV)
- Jalankan query SQL langsung
- Kelola user dan permission database
- Backup dan restore database

### phpPgAdmin (PostgreSQL)

Jika hosting mendukung PostgreSQL (contoh: DomaiNesia), phpPgAdmin tersedia untuk mengelola database PostgreSQL dengan cara serupa phpMyAdmin.

### MySQL Database Wizard

Di cPanel, gunakan **MySQL Database Wizard** untuk:
1. Buat database baru
2. Buat user database
3. Assign user ke database dengan permission yang tepat
4. Catat nama database, username, dan password untuk konfigurasi aplikasi

### Backup Database

| Metode | Cara |
|--------|------|
| **Manual via phpMyAdmin** | Buka phpMyAdmin → pilih database → tab Export → Go |
| **Manual via cPanel** | cPanel → Backup → Download MySQL Database Backup |
| **Otomatis via cPanel** | Beberapa hosting menyediakan auto backup harian/mingguan |
| **Manual via SSH** | `mysqldump -u user -p database_name > backup.sql` |

> 💡 **Tips**: Selalu backup database **sebelum** deploy perubahan besar atau update struktur tabel.

---

## 4. Domain Murah Indonesia

| Ekstensi | Harga Estimasi | Catatan |
|----------|---------------|---------|
| `.my.id` | Rp 10.000–15.000/tahun | Domain paling murah, cocok untuk belajar dan MVP |
| `.web.id` | Rp 15.000–25.000/tahun | Alternatif murah untuk website |
| `.biz.id` | Rp 15.000–25.000/tahun | Untuk bisnis kecil |
| `.com` | $8–12/tahun | **Cloudflare Registrar** harga at cost (tanpa markup) |
| `.com` | Rp 100.000–150.000/tahun | Via registrar Indonesia (DomaiNesia, Niagahoster) |
| `.id` | Rp 200.000–300.000/tahun | Domain premium Indonesia |

> 💡 **Tips**: Untuk belajar, pakai `.my.id`. Untuk bisnis serius, beli `.com` via Cloudflare Registrar (harga paling murah).

---

## 5. Tabel Perbandingan Hosting cPanel

| Provider | Harga Mulai | Panel | LiteSpeed | NVMe | Node.js | SSH | PostgreSQL | Catatan |
|----------|------------|-------|-----------|------|---------|-----|------------|---------|
| **DomaiNesia** | Rp 16.500/bln | cPanel | ✅ | ✅ | ✅ | ✅ | ✅ | Paling lengkap |
| **Niagahoster** | Rp 19.900/bln | hPanel ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | Bukan cPanel |
| **Hostinger ID** | Rp 15.900/bln | hPanel ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | Renewal naik 2-3x |
| **IDCloudHost** | Rp 15.000/bln | cPanel | ❌ | ❌ | ❌ | ✅ | ❌ | Murah, cPanel asli |
| **Dewaweb** | Rp 40.000/bln | cPanel | ❌ | ❌ | ❌ | ✅ | ❌ | Google Cloud, ISO 27001 |
| **Rumahweb** | Rp 16.000/bln | cPanel | ❌ | ❌ | ❌ | ✅ | ❌ | Provider lama |
| **AntMediaHost** | Rp 18.000/bln | cPanel | ❌ | ✅ | ❌ | ✅ | ❌ | NVMe storage |

> 📌 **Rekomendasi Paket C**: **DomaiNesia** karena mendukung cPanel + Node.js + PostgreSQL + NVMe + LiteSpeed secara lengkap.

---

## 6. Peringatan Harga Renewal

| Provider | Harga Promo | Harga Renewal | Kenaikan |
|----------|-------------|---------------|----------|
| Hostinger ID | Rp 15.900/bln | ~Rp 40.000-50.000/bln | **2-3x lipat** |
| Niagahoster | Rp 19.900/bln | ~Rp 40.000-60.000/bln | **2-3x lipat** |
| DomaiNesia | Rp 16.500/bln | ~Rp 20.000-30.000/bln | ~1.5x |

> ⚠️ **SELALU periksa harga renewal sebelum membeli.** Harga promo tahun pertama sering sangat murah, tapi renewal bisa mengejutkan. Cari informasi di halaman harga atau tanyakan ke customer service.

---

## 7. Link Resmi

| Layanan | URL |
|---------|-----|
| DomaiNesia | [domainesia.com](https://www.domainesia.com) |
| Niagahoster | [niagahoster.co.id](https://www.niagahoster.co.id) |
| Hostinger ID | [hostinger.co.id](https://www.hostinger.co.id) |
| IDCloudHost | [idcloudhost.com](https://idcloudhost.com) |
| Dewaweb | [dewaweb.com](https://www.dewaweb.com) |
| Rumahweb | [rumahweb.com](https://www.rumahweb.com) |
| AntMediaHost | [antmediahost.com](https://www.antmediahost.com) |
| Cloudflare Registrar | [cloudflare.com/products/registrar](https://www.cloudflare.com/products/registrar/) |
| cPanel Docs | [docs.cpanel.net](https://docs.cpanel.net) |
| phpMyAdmin | [phpmyadmin.net](https://www.phpmyadmin.net) |

---

## 8. Checklist

### Sebelum Beli Hosting

- [ ] Hosting menggunakan **cPanel** (bukan panel custom)?
- [ ] Ada **phpMyAdmin** untuk kelola MySQL?
- [ ] Support **MySQL/MariaDB**?
- [ ] Ada **SSH access**?
- [ ] Ada **AutoSSL / Let's Encrypt** (HTTPS gratis)?
- [ ] Sudah periksa **harga renewal** (bukan hanya harga promo)?
- [ ] Ada **Git integration**? (bonus)
- [ ] Ada **Setup Node.js App**? (bonus, untuk SSR)
- [ ] Ada **PostgreSQL**? (bonus, untuk migrasi ke Paket A nanti)

### Setelah Beli Hosting

- [ ] Sudah login ke cPanel?
- [ ] Sudah buat database MySQL via MySQL Database Wizard?
- [ ] Sudah catat nama database, username, dan password?
- [ ] Sudah akses phpMyAdmin dan test koneksi?
- [ ] Sudah upload file PHP dan test?
- [ ] Sudah upload file Next.js export (`out/`) ke `public_html`?
- [ ] AutoSSL/HTTPS sudah aktif?

### Domain

- [ ] Sudah beli domain?
- [ ] Sudah pointing nameserver atau A record ke IP hosting?
- [ ] DNS sudah propagasi? (cek via [whatsmydns.net](https://www.whatsmydns.net))
- [ ] Custom domain sudah bisa diakses?
- [ ] HTTPS aktif di domain custom?

### Database

- [ ] Sudah backup database sebelum perubahan besar?
- [ ] Sudah test import/export via phpMyAdmin?
- [ ] Permission user database sudah benar?
- [ ] Connection string sudah dimasukkan ke konfigurasi aplikasi?

### Operasional

- [ ] Sudah test semua halaman setelah deploy?
- [ ] Sudah test dari perangkat mobile?
- [ ] Sudah setup monitoring uptime (contoh: UptimeRobot)?
- [ ] Sudah catat tanggal renewal hosting dan domain?
