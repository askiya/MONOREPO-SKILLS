# 07 — Maintenance: Backup, Monitoring & Update (Paket C)

## Tujuan

Website di cPanel terpantau, database terbackup, dan kamu tahu cara update
file tanpa merusak yang sudah jalan.

---

## Sebelum Mulai

- [ ] Website sudah online dengan HTTPS ([06-domain-ssl.md](06-domain-ssl.md))

---

## Referensi Utama

> Detail lengkap: [`../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md`](../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md)

---

## 1. Backup MySQL Database

### Via phpMyAdmin (Disarankan)

1. cPanel → **phpMyAdmin**
2. Pilih database kamu di sidebar kiri
3. Klik tab **Export**
4. Format: **SQL**
5. Method: **Quick** (untuk database kecil) atau **Custom** (untuk pilih tabel tertentu)
6. Klik **Go** → file `.sql` terdownload

### Via cPanel Backup Wizard

1. cPanel → **Backup Wizard**
2. **Back Up** → **MySQL Databases**
3. Klik nama database → download

### Jadwal Backup

| Frekuensi | Kapan |
|---|---|
| Mingguan | Setiap project aktif / ada transaksi |
| Setelah perubahan besar | Tambah tabel, migrasi data, update skema |
| Sebelum update file | Sebelum upload versi baru website |

Simpan backup di:
- Google Drive / cloud storage
- Komputer lokal (folder terpisah, bukan di project)
- Minimal 3 versi terakhir

> ⚠️ **Jangan simpan backup hanya di server hosting.** Kalau server rusak,
> backup ikut hilang.

---

## 2. Cek Disk Usage

1. cPanel → **Disk Usage** (sidebar atau search)
2. Lihat:
   - Total disk used vs limit
   - Folder mana yang paling besar
   - Email dan log sering makan disk tanpa sadar

### Pembersihan Rutin

```text
cPanel → Email Accounts → cek mailbox penuh
cPanel → Error Log → clear kalau sudah terlalu besar
cPanel → File Manager → hapus file backup/zip lama di public_html
```

| Batas umum | Hosting murah | Hosting menengah |
|---|---|---|
| Disk | 1-5 GB | 10-50 GB |
| Bandwidth | 10-100 GB/bulan | Unlimited (fair use) |
| MySQL size | 500 MB | 1-5 GB |

> Cek limit hosting kamu di panel provider. Angka di atas hanya gambaran umum.

---

## 3. Update File Website

### Alur Update Aman

```text
1. Edit kode di lokal
2. npm run build → pastikan berhasil
3. Test di localhost (npx serve out)
4. Backup database (kalau ada perubahan skema)
5. Upload file baru ke cPanel
6. Verifikasi website masih jalan
```

### Upload Versi Baru

#### Kalau hanya update frontend (HTML/CSS/JS):

1. Build ulang di lokal: `npm run build`
2. Zip isi folder `out/`
3. cPanel → File Manager → `public_html/`
4. Hapus folder `_next/` lama
5. Upload zip baru → Extract
6. Hapus zip

#### Kalau update PHP API:

1. cPanel → File Manager → `public_html/api/`
2. Edit file langsung di File Manager, atau
3. Upload file baru (overwrite)
4. **Jangan** overwrite `config.php` — isinya credential server

#### Kalau update database schema:

1. Backup database dulu (langkah 1 di atas)
2. phpMyAdmin → tab **SQL**
3. Jalankan query ALTER TABLE / CREATE TABLE baru
4. Verifikasi data tidak hilang

> ⚠️ **Tidak ada rollback otomatis di cPanel.**
> Kalau update rusak, kamu harus upload versi lama manual.
> Selalu backup sebelum update.

---

## 4. Monitoring Sederhana

### Cek Website Masih Online

Pakai layanan uptime gratis:
- [UptimeRobot](https://uptimerobot.com) — gratis 50 monitor
- Setup: masukkan URL, pilih interval 5 menit, aktifkan notifikasi email

### Cek Error

1. cPanel → **Errors** (Error Log)
2. Lihat error PHP terbaru
3. Kalau ada error 500 berulang → cek `api/config.php` dan permission file

### Cek Traffic

1. cPanel → **Awstats** atau **Visitors**
2. Lihat jumlah visitor per hari/bulan
3. Kalau traffic naik mendadak → pastikan bukan bot/attack

---

## 5. Perpanjangan Hosting & Domain

| Item | Cek kapan | Aksi |
|---|---|---|
| Hosting | 30 hari sebelum expired | Perpanjang di panel provider |
| Domain | 30 hari sebelum expired | Perpanjang di registrar |
| AutoSSL | Otomatis perpanjang | Cek di cPanel → SSL/TLS Status |

> ⚠️ Hosting dan domain yang expired = website mati total.
> Set reminder di kalender 30 hari sebelum expired.

---

## 6. Serah Terima ke Klien

Kalau project ini untuk klien, siapkan dokumen serah terima:

Detail: [`../../docs/20-operasional-serah-terima/02-serah-terima-klien.md`](../../docs/20-operasional-serah-terima/02-serah-terima-klien.md)

Yang diserahkan:
- [ ] Akses cPanel (URL, username, password)
- [ ] Akses registrar domain
- [ ] File backup database terakhir
- [ ] Panduan cara update konten
- [ ] Jadwal perpanjangan hosting + domain
- [ ] Kontak support kamu

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Website hilang | Hosting expired | Perpanjang hosting, minta restore dari provider |
| "Error establishing database connection" | MySQL credential salah / database dihapus | Cek `config.php`, restore dari backup |
| Disk full | Email, log, atau file backup menumpuk | Bersihkan via cPanel |
| SSL expired | AutoSSL gagal renew | cPanel → SSL/TLS Status → Run AutoSSL manual |
| Update merusak website | Tidak test di lokal dulu | Restore dari backup, test lokal sebelum upload |

---

## Checklist Maintenance Bulanan

- [ ] Backup database MySQL (download ke lokal)
- [ ] Cek disk usage (hapus file tidak perlu)
- [ ] Cek error log (selesaikan error berulang)
- [ ] Cek uptime monitor (tidak ada downtime lama)
- [ ] Cek tanggal expired hosting dan domain
- [ ] Cek AutoSSL masih aktif
- [ ] Update dependency di lokal kalau ada security patch
