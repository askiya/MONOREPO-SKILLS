# 07 — Maintenance: Monitoring, Backup, dan Update

## Tujuan

Server dan aplikasi tetap sehat, data aman, dan update tidak merusak produksi.

## Prasyarat

- Deploy selesai, HTTPS aktif, health endpoint merespons.

> Rujukan: [`../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md`](../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md)

## 1. Monitoring Coolify

Coolify menyediakan log, status resource, dan metrik dasar langsung dari panel.

### Yang harus diperiksa rutin:

| Apa | Cara cek | Kapan |
|---|---|---|
| Status resource | Panel → Resource → status Running | Setiap hari |
| Log build terakhir | Panel → Resource → Deployments | Setelah setiap deploy |
| Disk VPS | `df -h` via SSH | Mingguan |
| RAM dan CPU | `htop` atau `top` via SSH | Mingguan |
| Health endpoint | `curl -s https://domain/api/health` | Harian atau via uptime monitor |

### Uptime Monitor Eksternal

Jangan hanya mengandalkan cek manual. Gunakan layanan monitor gratis:

- UptimeRobot (free tier)
- Better Uptime
- Healthchecks.io (untuk cron/background job)

Set monitor ke health endpoint. Terima notifikasi saat down.

## 2. Backup PostgreSQL

### Backup Manual

```bash
# Dari server, masuk ke container database Coolify
# Nama container bisa dilihat di panel Coolify atau:
docker ps | grep postgres

# Export
docker exec NAMA_CONTAINER pg_dump -U USERNAME NAMA_DB > backup_$(date +%Y%m%d).sql
```

### Backup Otomatis di Coolify

1. Panel Coolify → resource PostgreSQL → tab Backups.
2. Aktifkan backup terjadwal.
3. Tentukan frekuensi (minimal harian).
4. Pastikan lokasi penyimpanan backup tidak berada di disk yang sama dengan database.

### Simpan Backup di Luar VPS

VPS mati = backup di dalamnya juga hilang. Kirim satu salinan ke tempat lain:

```bash
scp backup_20260930.sql user@server-lain:/backups/
# Atau upload ke object storage (S3-compatible, Backblaze B2, dll)
```

### Uji Restore

Backup yang tidak pernah diuji restore bukan backup — hanya file yang memberi rasa aman palsu.

```bash
# Buat database uji (jangan restore ke database produksi)
docker exec -i NAMA_CONTAINER psql -U USERNAME -c 'CREATE DATABASE restore_test;'
docker exec -i NAMA_CONTAINER psql -U USERNAME restore_test < backup_20260930.sql

# Verifikasi data ada
docker exec NAMA_CONTAINER psql -U USERNAME restore_test -c 'SELECT COUNT(*) FROM "Product";'

# Bersihkan setelah uji
docker exec NAMA_CONTAINER psql -U USERNAME -c 'DROP DATABASE restore_test;'
```

Lakukan restore drill minimal satu kali per kuartal (3 bulan).

## 3. Update Server

### Update OS

```bash
sudo apt update && sudo apt upgrade -y
```

Jadwalkan bulanan. Reboot setelah update kernel jika diminta.

### Update Coolify

Ikuti panduan update di dokumentasi resmi Coolify versi yang kamu pakai. Baca changelog sebelum update — ada versi yang memerlukan migrasi manual.

Backup konfigurasi Coolify sebelum update besar. Jangan update Coolify dan deploy aplikasi pada hari yang sama.

### Update Dependency Aplikasi

```bash
npm outdated
npm audit
```

Periksa advisory sebelum update. Update satu per satu, bukan `npm update` sekaligus. Test setiap update:

```bash
npm run lint
npm test
npm run build
```

Commit dan push setelah test lulus. Coolify otomatis deploy.

## 4. Checklist Maintenance Bulanan

Jalankan checklist berikut sebulan sekali:

### Keamanan

- [ ] `npm outdated` / Dependabot diperiksa.
- [ ] Advisory keamanan ditinjau.
- [ ] Akun admin yang tidak dipakai dihapus.
- [ ] 2FA pemilik/admin aktif.
- [ ] Log login server diperiksa (`sudo journalctl -u ssh --since "30 days ago"`).

### Infrastruktur

- [ ] Disk, RAM, CPU, bandwidth diperiksa.
- [ ] Domain expiry dan auto-renew dicek (jangan sampai domain terlupakan).
- [ ] SSL expiry dicek (`curl -vI https://domain 2>&1 | grep "expire date"`).
- [ ] Health endpoint sehat.

### Database dan Backup

- [ ] Backup otomatis masih berjalan.
- [ ] Ukuran backup masuk akal (bukan 0 byte).
- [ ] Satu backup diunduh ke lokasi terpisah.
- [ ] Restore drill dijalankan per kuartal.

### Aplikasi

- [ ] `npm run lint && npm test && npm run build` lulus.
- [ ] Alur utama diuji manual (login, fitur inti).
- [ ] Email transaksional sampai inbox (bukan spam).

### Log Maintenance

Buat catatan setiap kali menjalankan checklist:

```text
Tanggal:
Pemeriksa:
Masalah ditemukan:
Tindakan:
Review berikutnya:
```

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Disk penuh, build gagal | Image lama dan log menumpuk | Prune image dan rotate log |
| Health endpoint error setelah update | Dependency berubah perilaku | Rollback deploy, uji lokal, deploy ulang |
| Sertifikat expired | Renewal gagal tanpa notifikasi | Cek log Coolify, pastikan port 80 terbuka |
| Backup 0 byte | Credential DB berubah | Samakan credential di perintah pg_dump |
| Domain expired | Auto-renew gagal karena pembayaran | Set reminder manual selain auto-renew |

## Checklist

- [ ] Monitoring aktif: panel Coolify, SSH resource, uptime external.
- [ ] Backup PostgreSQL terjadwal dan tersimpan di luar VPS.
- [ ] Restore drill pernah berhasil.
- [ ] Update OS dijadwalkan bulanan.
- [ ] Update dependency melalui test sebelum deploy.
- [ ] Catatan maintenance bulanan dilakukan.
