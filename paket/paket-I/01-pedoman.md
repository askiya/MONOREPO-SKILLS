# 01 — Pedoman Perencanaan Multi-Service

## Tujuan

Menghasilkan dokumen yang membatasi tanggung jawab layanan, kontrak jaringan, data, queue, deployment, dan operasi sebelum kode dibuat.

## Prasyarat

Baca [`README.md`](README.md). Salin template umum dari [`../../templates/`](../../templates/) bila perlu.

## 1. Tetapkan PRD

Buat `docs/PRD.md` dengan bagian berikut:

```markdown
# PRD
## Masalah dan pengguna
## Hasil bisnis dan metrik
## Fitur wajib
## Di luar cakupan
## Alur pengguna
## Kebutuhan nonfungsional
- SLO ketersediaan
- target latensi p95
- target waktu penyelesaian job
- RPO dan RTO
## Kriteria penerimaan
```

Jangan menulis “cepat” atau “aman” tanpa angka atau bukti uji. Bedakan permintaan sinkron dari pekerjaan latar belakang.

**Hasil yang diharapkan:** setiap fitur memiliki pemilik layanan dan kriteria penerimaan terukur.

## 2. Tetapkan Batas Layanan

| Layanan | Tanggung jawab | Tidak boleh |
|---|---|---|
| `web` | UI, SSR, memanggil API | akses DB/Redis langsung |
| `api` | validasi, auth, transaksi, enqueue job | menjalankan job panjang di request |
| `worker` | konsumsi BullMQ, retry, idempotensi | menerima trafik publik |
| `postgres` | data utama dan transaksi | dibuka ke internet |
| `redis` | queue, cache ber-TTL | menjadi sumber data permanen |
| `traefik` | routing HTTP, TLS | menyimpan logika bisnis |
| `uptime-kuma` | probe dan notifikasi | menggantikan backup/log |

## 3. Tulis ARCHITECTURE Multi-Service

Buat `docs/ARCHITECTURE.md` dan wajib isi:

```markdown
# ARCHITECTURE
## Context diagram
## Container diagram
## Batas kepercayaan dan aliran data
## Kontrak HTTP
## Skema database dan kepemilikan tabel
## Queue
- nama queue
- payload dan versi
- retry/backoff
- idempotency key
- dead-letter policy
## Jaringan dan port
## Healthcheck readiness/liveness
## Secret dan konfigurasi
## Backup, RPO, RTO
## Kapasitas, bottleneck, dan scaling
## Failure modes dan recovery
## ADR
```

Gunakan tiga jaringan: `proxy` untuk Traefik dan layanan HTTP, `backend` untuk API/worker/data, serta jaringan monitor bila diperlukan. Hanya `80` dan `443` dipublikasikan.

**Hasil yang diharapkan:** diagram menunjukkan dependensi dan perilaku saat Redis, DB, worker, atau API gagal.

## 4. Rancang Kontrak API dan Queue

Contoh keputusan:

```yaml
http:
  POST /v1/reports:
    success: 202
    response: { jobId: string, status: queued }
queue:
  name: report.generate.v1
  payload: { jobId: string, userId: string, reportId: string }
  attempts: 5
  backoff: exponential
  idempotencyKey: jobId
```

Validasi payload pada producer dan consumer. Simpan status job di PostgreSQL; Redis bukan sumber kebenaran.

## 5. Buat SDLC dan TASKS

Fase minimum:

1. Kontrak dan migrasi.
2. API sehat tanpa queue.
3. Producer dan worker dengan job idempoten.
4. Web terhubung API.
5. Compose lokal dan test kegagalan.
6. Staging, backup, restore.
7. Produksi, DNS, TLS, monitor.

Setiap task memuat: layanan, file, dependensi, acceptance test, risiko, dan rollback.

## 6. Tulis DESIGN dan AGENTS

`docs/DESIGN.md` menetapkan halaman, state loading/error/empty, aksesibilitas, dan responsive breakpoint. `AGENTS.md` menetapkan:

- perintah build/test per layanan;
- folder yang boleh diubah;
- kontrak lintas layanan yang tidak boleh diubah diam-diam;
- larangan secret dan port data publik;
- syarat migrasi backward-compatible;
- Definition of Done.

## 7. Review Kegagalan

| Gejala | Keputusan desain |
|---|---|
| Redis mati | API gagal enqueue dengan 503; data bisnis tidak hilang |
| Worker mati | job menunggu; alarm queue depth menyala |
| PostgreSQL mati | API tidak ready; web tampilkan gangguan |
| deploy worker baru | payload lama tetap didukung selama masa transisi |
| job terkirim dua kali | idempotency key mencegah efek ganda |

## Checklist

- [ ] PRD memiliki metrik, SLO, RPO, RTO, dan batas cakupan.
- [ ] ARCHITECTURE memuat semua layanan dan tiga aliran: HTTP, SQL, queue.
- [ ] Setiap port publik dan internal terdokumentasi.
- [ ] Kontrak API dan payload queue berversi.
- [ ] Retry, backoff, dead-letter, dan idempotensi diputuskan.
- [ ] TASKS dapat dikerjakan dan diuji per layanan.
- [ ] AGENTS memuat perintah validasi dan larangan keamanan.
- [ ] Failure mode utama memiliki perilaku dan langkah pemulihan.
