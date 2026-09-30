# 04 — Preview dan Testing Lokal

## Tujuan

Menjalankan seluruh stack dengan Docker Compose, menguji tiap layanan, lalu membuktikan aliran HTTP, database, cache, queue, dan worker.

## Prasyarat

Build pada [`03-build.md`](03-build.md) selesai. Docker Engine aktif.

## 1. Buat Override Lokal

`compose.override.yaml`:

```yaml
services:
  web:
    ports: ["3000:3000"]
  api:
    ports: ["4000:4000"]
```

PostgreSQL dan Redis tetap tanpa port host. Gunakan `docker compose exec` untuk inspeksi.

## 2. Validasi dan Jalankan

```bash
docker compose config --quiet
docker compose up -d --build
docker compose ps
```

**Hasil yang diharapkan:** `web`, `api`, `worker`, `postgres`, dan `redis` berstatus `Up`; layanan ber-healthcheck akhirnya `healthy`.

## 3. Test per Layanan

### PostgreSQL

```bash
docker compose exec postgres pg_isready -U app -d app
```

**Hasil yang diharapkan:** `accepting connections`.

### Redis

```bash
docker compose exec redis redis-cli ping
```

**Hasil yang diharapkan:** `PONG`.

### API

```bash
curl --fail http://localhost:4000/health
```

**Hasil yang diharapkan:** HTTP `200` dan JSON status sehat.

### Frontend

```bash
curl --fail --head http://localhost:3000
```

**Hasil yang diharapkan:** HTTP `200` atau redirect yang memang dirancang.

### Worker dan Queue

Kirim request yang membuat job:

```bash
curl --fail -X POST http://localhost:4000/v1/jobs \
  -H 'Content-Type: application/json' \
  -d '{"type":"smoke-test","payload":{"message":"halo"}}'
docker compose logs --since=2m worker
```

**Hasil yang diharapkan:** API membalas `202` dengan `jobId`; log worker memiliki `job.completed` untuk ID sama.

## 4. Jalankan Test Otomatis

```bash
npm --prefix apps/web run lint
npm --prefix apps/web run build
npm --prefix apps/api test
npm --prefix apps/api run build
npm --prefix apps/worker test
npm --prefix apps/worker run build
```

Jika script belum ada, tambahkan test kecil; jangan menghapus command dari gate.

**Hasil yang diharapkan:** semua command exit code `0`.

## 5. Uji Failure Mode

### Worker berhenti

```bash
docker compose stop worker
# kirim satu job lagi
docker compose start worker
docker compose logs --since=2m worker
```

**Hasil yang diharapkan:** job tertahan saat worker mati lalu selesai setelah worker hidup.

### Redis berhenti

```bash
docker compose stop redis
curl -i -X POST http://localhost:4000/v1/jobs -H 'Content-Type: application/json' -d '{"type":"smoke-test","payload":{}}'
docker compose start redis
```

**Hasil yang diharapkan:** API gagal terkendali dengan `503`, bukan hang atau membalas sukses palsu.

### Restart stack

```bash
docker compose restart
docker compose ps
```

**Hasil yang diharapkan:** data PostgreSQL tetap ada dan layanan kembali sehat.

## 6. Bersihkan Lab

```bash
docker compose down
```

Gunakan `docker compose down -v` hanya untuk data lokal yang boleh dihapus.

## Masalah Umum

| Gejala | Diagnosis | Perbaikan |
|---|---|---|
| container `unhealthy` | `docker compose logs SERVICE` | perbaiki dependency atau health endpoint |
| API tidak menjangkau DB | hostname memakai `localhost` | gunakan hostname `postgres` |
| job tidak diproses | nama queue producer/worker beda | satukan konstanta kontrak |
| port bentrok | port 3000/4000 terpakai | hentikan proses lama atau ubah override |
| data hilang | volume tidak terpasang | cek `docker volume ls` dan Compose |

## Checklist

- [ ] `docker compose config --quiet` lulus.
- [ ] Semua container berjalan dan healthcheck hijau.
- [ ] Web dan API dapat diakses dari host.
- [ ] PostgreSQL dan Redis hanya diuji lewat jaringan/exec internal.
- [ ] Job dari API selesai di worker dengan ID sama.
- [ ] Test web, API, dan worker lulus.
- [ ] Gangguan worker dan Redis menghasilkan perilaku yang dirancang.
- [ ] Data bertahan setelah restart.
