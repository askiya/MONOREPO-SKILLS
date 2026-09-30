# 07 — Maintenance: Log, Backup, Keamanan, dan Update

## Tujuan

Stack Docker tetap sehat, data aman, dan rilis baru tidak merusak produksi.

## Prasyarat

- Deploy, Nginx, dan TLS aktif dari bab 05 dan 06.
- Health endpoint merespons.

> Rujukan: [`../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md`](../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md)

## 1. Monitoring dan Log

### Cek Status Container

```bash
docker compose ps
```

**Hasil yang diharapkan:** semua service `Up`; database `healthy`. Service yang `Restarting` harus segera diinvestigasi.

### Baca Log

```bash
docker compose logs --tail=200 web
docker compose logs --tail=200 api
docker compose logs --tail=200 db
```

Untuk log real-time:

```bash
docker compose logs -f api
```

### Health Check

```bash
curl -fsS http://127.0.0.1:3000 > /dev/null && echo "web ok" || echo "web down"
curl -fsS http://127.0.0.1:4000/health && echo "" || echo "api down"
```

### Monitoring Resource Server

```bash
df -h         # disk
free -m       # RAM
uptime        # load average
```

Jika disk penuh: image, volume, dan log lama kemungkinan menumpuk. Lihat bagian image prune.

### Uptime Monitor Eksternal

Gunakan UptimeRobot, Better Uptime, atau Healthchecks.io untuk monitoring 24/7. Jangan hanya mengandalkan cek manual.

## 2. Backup PostgreSQL

### Backup Manual

```bash
cd /opt/myapp
docker compose exec db pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} > backup_$(date +%Y%m%d).sql
ls -lh backup_*.sql
```

**Hasil yang diharapkan:** file SQL berukuran masuk akal (bukan 0 byte).

### Backup Otomatis via Cron

Buat script `/opt/myapp/backup.sh`:

```bash
#!/bin/bash
set -e
BACKUP_DIR="/opt/myapp/backups"
mkdir -p "$BACKUP_DIR"
FILENAME="$BACKUP_DIR/backup_$(date +%Y%m%d_%H%M%S).sql"
docker compose -f /opt/myapp/docker-compose.yml exec -T db pg_dump -U "${POSTGRES_USER}" "${POSTGRES_DB}" > "$FILENAME"
# Hapus backup lebih dari 14 hari
find "$BACKUP_DIR" -name "*.sql" -mtime +14 -delete
echo "Backup selesai: $FILENAME"
```

```bash
chmod +x /opt/myapp/backup.sh
```

Tambahkan ke crontab user deploy:

```bash
crontab -e
```

Tambah baris:

```
0 3 * * * /opt/myapp/backup.sh >> /opt/myapp/backups/cron.log 2>&1
```

Backup berjalan setiap hari jam 03:00.

### Simpan di Luar VPS

Backup di VPS yang sama bukan backup. Kirim salinan ke tempat lain:

```bash
scp /opt/myapp/backups/backup_20260930.sql user@server-lain:/backups/
```

Atau upload ke object storage (S3-compatible, Backblaze B2).

### Uji Restore

Backup tanpa uji restore bukan backup, hanya file yang memberi rasa aman palsu.

```bash
docker compose exec -T db psql -U ${POSTGRES_USER} -c 'CREATE DATABASE restore_test;'
docker compose exec -T db psql -U ${POSTGRES_USER} restore_test < backup_20260930.sql
docker compose exec db psql -U ${POSTGRES_USER} restore_test -c 'SELECT COUNT(*) FROM "Product";'
docker compose exec db psql -U ${POSTGRES_USER} -c 'DROP DATABASE restore_test;'
```

**Hasil yang diharapkan:** jumlah baris sesuai harapan.

Lakukan restore drill minimal per kuartal.

## 3. Image Prune

Docker menyimpan setiap layer lama. Setelah beberapa deploy, disk bisa penuh.

```bash
docker system df
docker image prune -af --filter "until=168h"
docker builder prune -af
```

Jangan jalankan `docker system prune -a` tanpa pikir — perintah itu menghapus volume yang tidak sedang dipakai, termasuk volume database yang mungkin terhubung ke compose lain.

## 4. Update Server

### Update OS

```bash
sudo apt update && sudo apt upgrade -y
```

Reboot setelah update kernel. Jadwalkan bulanan.

### Update Docker Engine

Docker dari `get.docker.com` di-update melalui apt:

```bash
sudo apt update && sudo apt upgrade docker-ce docker-ce-cli containerd.io -y
docker --version
```

### Update Nginx

```bash
sudo apt update && sudo apt upgrade nginx -y
sudo nginx -t
sudo systemctl reload nginx
```

### Update Dependency Aplikasi

```bash
cd /opt/myapp
npm outdated --workspaces
npm audit
```

Periksa advisory. Update satu per satu, test lokal, lalu push:

```bash
npm run lint --workspaces
npm test --workspaces
npm run build --workspaces
```

Push setelah test lulus. GitHub Actions mengirim deploy.

## 5. Security Update

- [ ] Cek advisory npm dan GitHub Dependabot.
- [ ] Update base image Docker (`node:22-alpine`) bila ada patch keamanan.
- [ ] Rebuild image setelah update base:

```bash
docker compose build --pull --no-cache
docker compose up -d
```

- [ ] Cek `sudo journalctl -u ssh --since "30 days ago"` untuk login mencurigakan.
- [ ] Cek akun admin yang masih aktif, hapus yang tidak dipakai.

## 6. Checklist Maintenance Bulanan

### Keamanan

- [ ] Dependency npm diperiksa dan advisory ditinjau.
- [ ] Base image Docker terbaru.
- [ ] Akun admin tidak terpakai dihapus.
- [ ] 2FA pemilik aktif.
- [ ] Log SSH diperiksa.

### Infrastruktur

- [ ] `docker compose ps` semua sehat.
- [ ] `df -h` disk tidak penuh.
- [ ] Domain expiry dicek.
- [ ] SSL expiry dicek (`sudo certbot certificates`).
- [ ] Health endpoint sehat.

### Database dan Backup

- [ ] Backup otomatis berjalan (cek `cron.log`).
- [ ] Ukuran backup masuk akal.
- [ ] Satu backup disimpan di luar VPS.
- [ ] Restore drill per kuartal.

### Aplikasi

- [ ] Lint, test, build lulus.
- [ ] Alur utama diuji manual.
- [ ] Email transaksional sampai inbox.

### Log Maintenance

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
| Disk 100% | Image dan log menumpuk | `docker system df`, prune, rotate log |
| Container restart loop setelah update | Dependency breaking change | Rollback ke commit sebelumnya, debug lokal |
| SSL expired | Certbot renew gagal | `sudo certbot renew`, cek port 80 dan DNS |
| Backup 0 byte | Credential berubah atau container mati | Samakan env, pastikan container db hidup |
| Deploy gagal setelah Docker upgrade | API Docker Compose berubah | `docker compose version`, sesuaikan syntax |

## Checklist

- [ ] Monitoring container dan resource server aktif.
- [ ] Uptime monitor eksternal terpasang.
- [ ] Backup cron berjalan harian dan tersimpan di luar VPS.
- [ ] Restore drill pernah berhasil.
- [ ] Image prune dijadwalkan.
- [ ] Update OS, Docker, Nginx dijadwalkan bulanan.
- [ ] Update dependency melalui test sebelum deploy.
- [ ] Catatan maintenance bulanan dilakukan.
