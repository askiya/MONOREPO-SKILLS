# 04 — Preview dan Evaluasi Lokal

## Tujuan

Menjalankan stack AI lokal, menguji API, membuktikan retrieval menemukan sumber benar, dan mengukur perilaku no-answer.

## Prasyarat

Build [`03-build.md`](03-build.md) selesai. Model sudah diunduh; RAM/disk cukup.

## 1. Jalankan Compose

Tambahkan override lokal:

```yaml
services:
  web:
    ports: ["3000:3000"]
  api:
    ports: ["8000:8000"]
  ollama:
    ports: ["11434:11434"]
```

Port Ollama hanya untuk localhost saat lab; jangan publikasikan di produksi.

```bash
docker compose config --quiet
docker compose up -d --build
docker compose ps
```

**Hasil yang diharapkan:** web, API, PostgreSQL, dan Ollama `Up`; DB/API sehat.

## 2. Test Infrastruktur

```bash
curl --fail http://localhost:8000/health
curl --fail http://localhost:11434/api/tags
docker compose exec postgres psql -U aiapp -d aiapp -c "SELECT extversion FROM pg_extension WHERE extname='vector';"
```

**Hasil yang diharapkan:** health `200`, model muncul, dan pgvector punya versi.

## 3. Ingest Dokumen Uji

Buat `fixtures/pedoman.txt` berisi fakta sintetis dengan ID sumber. Kemudian:

```bash
curl --fail -X POST http://localhost:8000/v1/documents \
  -H 'X-Tenant-ID: 00000000-0000-0000-0000-000000000001' \
  -F 'file=@fixtures/pedoman.txt'
```

**Hasil yang diharapkan:** respons memiliki `document_id` dan status `indexed` atau job status yang dapat dipantau.

## 4. Test Retrieval

```bash
curl --fail -X POST http://localhost:8000/v1/retrieve \
  -H 'Content-Type: application/json' \
  -H 'X-Tenant-ID: 00000000-0000-0000-0000-000000000001' \
  -d '{"query":"Apa kebijakan backup?","top_k":5}'
```

**Hasil yang diharapkan:** source/chunk relevan berada pada hasil atas, score tercantum, dan tidak ada dokumen tenant lain.

Uji tenant kedua dengan ID berbeda. **Hasil yang diharapkan:** nol hasil untuk dokumen tenant pertama.

## 5. Test API AI

```bash
curl --fail -X POST http://localhost:8000/v1/ask \
  -H 'Content-Type: application/json' \
  -H 'X-Tenant-ID: 00000000-0000-0000-0000-000000000001' \
  -d '{"question":"Apa kebijakan backup?"}'
```

**Hasil yang diharapkan:** jawaban merujuk fakta dokumen dan response memuat citation source/chunk.

No-answer:

```bash
curl --fail -X POST http://localhost:8000/v1/ask \
  -H 'Content-Type: application/json' \
  -H 'X-Tenant-ID: 00000000-0000-0000-0000-000000000001' \
  -d '{"question":"Siapa juara turnamen yang tidak dibahas?"}'
```

**Hasil yang diharapkan:** “Informasi tidak ditemukan”, tanpa citation palsu.

## 6. Jalankan Evaluasi RAG

```bash
python evals/run.py --dataset evals/dataset.jsonl --base-url http://localhost:8000
```

Laporan minimum:

```text
model_chat=...
model_embedding=...
dataset_version=...
recall_at_5=...
mrr=...
no_answer_precision=...
p95_retrieval_ms=...
p95_total_ms=...
```

Bandingkan dengan gate PRD. Jangan mengubah dataset setelah melihat output agar nilai tampak bagus.

## 7. Test Otomatis dan Beban Ringan

```bash
python -m pytest apps/api/tests
npm --prefix apps/web run lint
npm --prefix apps/web run build
```

Kirim concurrency kecil sesuai target lab; pantau:

```bash
docker stats --no-stream
docker compose logs --since=10m api ollama
```

**Hasil yang diharapkan:** tidak ada OOM/restart; latency masih dalam gate.

## Masalah Umum

| Gejala | Diagnosis | Perbaikan |
|---|---|---|
| embedding lambat | CPU penuh/model besar | batch lebih kecil atau gunakan GPU/model kecil |
| hasil tak relevan | chunk/top-k/model | ubah satu variabel dan eval ulang |
| dimensi error | model berbeda dari schema | pakai model benar atau re-index |
| jawaban tanpa sumber | threshold/prompt longgar | blok generation saat bukti kurang |
| OOM | context/model/concurrency tinggi | turunkan limit atau tambah resource |

## Checklist

- [ ] Compose dan health endpoint sehat.
- [ ] pgvector dan dua model tersedia.
- [ ] Dokumen sintetis berhasil di-index.
- [ ] Retrieval menemukan source benar dan mengisolasi tenant.
- [ ] Ask menghasilkan citation yang dapat diverifikasi.
- [ ] Pertanyaan di luar konteks ditolak.
- [ ] Eval mencatat model, dataset, kualitas, dan latency.
- [ ] Unit test, lint, build, dan observasi resource lulus.
