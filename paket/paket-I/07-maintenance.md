# 07 — Maintenance, Monitoring, Backup, dan Keamanan

## Tujuan

Menjaga stack dapat diamati, dipulihkan, diperbarui, dan dibersihkan tanpa menghapus data produksi.

## Prasyarat

Produksi dan TLS sehat. Tentukan PIC alarm, jadwal maintenance, RPO, dan RTO.

## 1. Operasikan Log Docker

```bash
docker compose ps
docker compose logs --since=30m api
docker compose logs --since=30m worker
docker stats --no-stream
df -h
docker system df
```

**Hasil yang diharapkan:** semua layanan sehat; tidak ada restart loop; disk dan memori di bawah ambang tim.

Atur rotasi log:

```yaml
logging:
  driver: json-file
  options:
    max-size: 10m
    max-file: "5"
```

Log harus terstruktur dan tidak memuat password, token, cookie, atau payload sensitif.

## 2. Pasang Uptime Kuma

Monitor minimum:

| Monitor | Probe | Interval | Alarm |
|---|---|---|---|
| Frontend | HTTPS `/` | 60 detik | 2 kegagalan |
| API | HTTPS `/health` | 30 detik | 2 kegagalan |
| Worker heartbeat | push monitor | 60 detik | terlambat 2 interval |
| TLS | certificate expiry | harian | sebelum masa aman tim |

Kirim notifikasi uji ke kanal tim.

**Hasil yang diharapkan:** alarm test diterima dan PIC tahu runbook yang harus dibuka.

## 3. Grafana dan Loki Opsional

Tambahkan hanya jika log Docker tidak cukup. Loki mengumpulkan log; Grafana melakukan query/dashboard. Batasi retention dan jangan mempublikasikan UI admin tanpa auth kuat/VPN.

Wajib sebelum adopsi:

- estimasi disk dan retention;
- label cardinality rendah;
- redaksi secret/PII;
- dashboard API error, queue failure, restart, dan latency.

## 4. Backup PostgreSQL

```bash
mkdir -p backups
docker compose exec -T postgres pg_dump -U app -d app -Fc > backups/app-$(date +%F-%H%M).dump
```

Enkripsi dan kirim ke storage off-site. Retensi contoh: harian, mingguan, bulanan sesuai kebijakan. Redis queue bukan pengganti database backup.

Uji restore pada database terpisah:

```bash
docker compose exec -T postgres createdb -U app app_restore_test
docker compose exec -T postgres pg_restore -U app -d app_restore_test --clean --if-exists < backups/NAMA_FILE.dump
docker compose exec -T postgres psql -U app -d app_restore_test -c 'SELECT current_database();'
```

**Hasil yang diharapkan:** query mengembalikan `app_restore_test`; data sampel dan jumlah record diverifikasi.

## 5. Update Aman

1. Baca changelog dan advisory.
2. Ambil backup serta uji restore terbaru.
3. Uji versi image di staging.
4. Jalankan migrasi backward-compatible.
5. Deploy satu rilis terpin.
6. Jalankan smoke test dan pantau error/queue.
7. Rollback bila acceptance gate gagal.

```bash
docker compose pull
docker compose build --pull
docker compose up -d
docker compose ps
```

## 6. Prune Aman

Lihat dampak lebih dulu:

```bash
docker system df
docker image prune
docker builder prune
```

Jangan menjalankan `docker system prune --volumes` di produksi. Volume mungkin memuat PostgreSQL, Redis, atau sertifikat ACME.

## 7. Baseline Keamanan

- Patch OS dan Docker berkala.
- SSH key saja; fail2ban opsional; port data tidak publik.
- Image non-root, read-only filesystem bila kompatibel, `no-new-privileges`.
- Secret dirotasi dan akses dibatasi.
- Scan dependency dan image; pin versi.
- Dashboard admin dilindungi auth kuat atau VPN.
- Audit restore, akses, dan incident drill per kuartal.

## Runbook Cepat

| Gejala | Periksa | Tindakan awal |
|---|---|---|
| HTTP 502 | log Traefik dan health API/web | rollback image atau pulihkan dependency |
| queue menumpuk | worker restart, Redis, failure rate | hentikan producer bila backlog membahayakan |
| disk > 85% | log, image, backup lokal | rotasi log; pindah backup; prune image aman |
| DB lambat | koneksi, query lambat, disk | kurangi beban; analisis query; scale terencana |
| sertifikat gagal renew | log ACME, DNS, port 80/443 | perbaiki sebelum kedaluwarsa; jangan spam retry |

## Checklist

- [ ] Uptime Kuma memantau web, API, worker, dan TLS.
- [ ] Notifikasi uji diterima PIC.
- [ ] Log terotasi dan bebas secret.
- [ ] Backup PostgreSQL otomatis, terenkripsi, dan off-site.
- [ ] Restore drill lulus dalam RTO.
- [ ] Update diuji di staging dan memiliki rollback.
- [ ] Prune tidak menyentuh volume produksi.
- [ ] Patch, rotasi secret, dan audit memiliki jadwal.
- [ ] Runbook insiden dapat diakses saat aplikasi mati.
