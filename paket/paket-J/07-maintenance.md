# 07 — Maintenance Model, Vector DB, GPU, Backup, dan Biaya

## Tujuan

Menjaga kualitas model/RAG, kesehatan pgvector, resource CPU/GPU, backup, dan biaya sepanjang umur produk.

## Prasyarat

Tentukan owner model, owner data, owner infra, acceptance dataset, RPO/RTO, dan anggaran bulanan.

## 1. Pantau Service dan Resource

```bash
docker compose ps
docker stats --no-stream
df -h
docker compose logs --since=30m api ollama postgres
```

GPU:

```bash
nvidia-smi
nvidia-smi dmon -s pucvmet
```

Pantau:

- request rate, error rate, p50/p95 latency;
- token/input length dan queue/concurrency;
- CPU, RAM, GPU utilization, VRAM, suhu, power;
- disk DB/model dan restart/OOM;
- retrieval score, no-answer rate, citation feedback.

**Hasil yang diharapkan:** alarm diterima sebelum disk/VRAM habis atau SLO gagal.

## 2. Update Model dengan Gate

Jangan mengganti tag model langsung di produksi.

1. Catat model lama, digest, parameter, prompt version.
2. Pull model baru di staging.
3. Jalankan dataset evaluasi sama.
4. Bandingkan kualitas, latency, RAM/VRAM, throughput, lisensi.
5. Canary traffic nonkritis.
6. Promosikan jika semua gate lulus.
7. Simpan rollback ke model lama.

```bash
ollama pull MODEL:TAG
ollama list
python evals/run.py --dataset evals/dataset.jsonl
```

Jika model embedding berubah, buat namespace/kolom indeks versi baru, re-embed seluruh corpus, verifikasi, lalu switch atomik. Jangan mencampur vector dari dua model/dimensi.

## 3. Maintenance pgvector

```bash
docker compose exec postgres psql -U aiapp -d aiapp -c "SELECT pg_size_pretty(pg_database_size('aiapp'));"
docker compose exec postgres psql -U aiapp -d aiapp -c "ANALYZE chunks;"
```

Pantau bloat, dead tuples, query plan, index size, dan recall. `VACUUM` rutin melalui autovacuum; `REINDEX`/`VACUUM FULL` memerlukan rencana downtime dan disk ekstra.

Uji query:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id FROM chunks
WHERE tenant_id = '00000000-0000-0000-0000-000000000001'
ORDER BY embedding <=> '[...]'::vector
LIMIT 5;
```

Pastikan indeks dan filter tenant bekerja pada skala produksi.

## 4. Backup dan Restore

Backup wajib mencakup:

- PostgreSQL: metadata, chunks, embeddings, evaluasi;
- dokumen sumber asli di object storage/off-site;
- manifest model/digest dan konfigurasi prompt;
- migrasi serta dataset evaluasi.

```bash
docker compose exec -T postgres pg_dump -U aiapp -d aiapp -Fc > backups/aiapp-$(date +%F-%H%M).dump
```

Enkripsi, kirim off-site, dan uji restore:

```bash
docker compose exec -T postgres createdb -U aiapp aiapp_restore_test
docker compose exec -T postgres pg_restore -U aiapp -d aiapp_restore_test --clean --if-exists < backups/NAMA_FILE.dump
docker compose exec -T postgres psql -U aiapp -d aiapp_restore_test -c 'SELECT count(*) FROM chunks;'
```

**Hasil yang diharapkan:** jumlah dokumen/chunk cocok; sampel retrieval dari restore menghasilkan sumber yang sama.

Model binary dapat diunduh ulang bila digest tersedia; jangan mengandalkan nama tag bergerak.

## 5. Kontrol Biaya

Catat per hari/bulan:

| Komponen | Driver biaya | Kendali |
|---|---|---|
| VPS/GPU | jam aktif, tipe instance | schedule nonproduksi, right-size |
| inference | token, context, concurrency | limit panjang, cache aman, model kecil |
| embedding | jumlah chunk/re-index | checksum, batch, hindari re-index palsu |
| DB | volume vector dan IOPS | retention, chunk tuning, archive |
| egress/backup | ukuran dokumen/log | kompresi, retention, region |
| waktu operasi | insiden dan update | otomatisasi gate/runbook |

Pasang alert anggaran provider. Ukur biaya per 1.000 request dan per dokumen terindeks.

## 6. Retensi dan Penghapusan

Penghapusan tenant harus menghapus dokumen, chunk/vector, cache, log terkait sesuai kebijakan, dan menunggu expiry backup yang terdokumentasi. Audit dengan ID, bukan menyalin isi sensitif ke log.

## 7. Runbook

| Gejala | Periksa | Tindakan awal |
|---|---|---|
| kualitas turun | model/prompt/data version | rollback versi; jalankan eval |
| retrieval kosong | ingest status, dimensi, filter tenant | hentikan generation; perbaiki index |
| GPU OOM | context/concurrency/VRAM | kurangi beban; restart terkontrol |
| DB lambat | query plan, bloat, disk | ANALYZE; scale/tune terukur |
| biaya melonjak | traffic, re-index, GPU idle | rate limit; hentikan job; right-size |
| data lintas tenant | auth/filter/query | blok layanan, respons insiden, audit |

## Checklist

- [ ] Dashboard dan alarm meliputi aplikasi, DB, model, disk, serta GPU.
- [ ] Model/prompt/dataset version tercatat pada setiap evaluasi.
- [ ] Update model melewati staging, eval, canary, dan rollback.
- [ ] Perubahan embedding memakai re-index versi terpisah.
- [ ] pgvector dianalisis dan query plan ditinjau berkala.
- [ ] Backup DB, dokumen, manifest model, dan konfigurasi tersimpan off-site.
- [ ] Restore dan retrieval dari hasil restore lulus dalam RTO.
- [ ] Anggaran dan biaya per unit dipantau.
- [ ] Penghapusan data serta insiden lintas tenant punya runbook.
