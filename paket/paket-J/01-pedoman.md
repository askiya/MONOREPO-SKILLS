# 01 — Pedoman Perencanaan Produk AI dan RAG

## Tujuan

Menghasilkan PRD dan arsitektur yang menetapkan model, data, RAG pipeline, evaluasi, keamanan, resource, serta batas tanggung jawab produk.

## Prasyarat

Baca [`README.md`](README.md). Pastikan hak pemrosesan dokumen dan lisensi model diketahui.

## 1. Tulis PRD AI

Buat `docs/PRD.md`:

```markdown
# PRD Produk AI
## Masalah dan pengguna
## Keputusan yang dibantu AI
## Input dan output
## Model
- model chat dan versi
- model embedding dan dimensi
- lisensi serta batas penggunaan
- bahasa/context window
## RAG pipeline
- sumber dan hak data
- parsing, chunking, embedding, indexing
- retrieval, filter, prompt, citation
## Target nonfungsional
- latency p50/p95
- concurrency
- kualitas minimum
- biaya per permintaan
## Evaluasi dan acceptance set
## Privasi, retensi, penghapusan
## Risiko dan mitigasi
## Di luar cakupan
```

**Hasil yang diharapkan:** “jawaban bagus” diganti metrik seperti recall@k, citation correctness, groundedness, latency, dan refusal rate.

## 2. Pilih Runtime Model

| Pilihan | Cocok | Trade-off |
|---|---|---|
| Ollama | lab, satu node, setup mudah | throughput dan kontrol serving terbatas |
| vLLM | GPU, concurrency/throughput tinggi | setup serta tuning lebih rumit |
| API terkelola | traffic awal tidak pasti | data, biaya variabel, vendor dependency |

Catat keputusan di ADR. Jangan memilih model hanya dari ukuran; uji bahasa Indonesia, lisensi, memori, context, dan kualitas pada acceptance set.

## 3. Rancang RAG Pipeline

```text
INGEST
Dokumen → validasi tipe/ukuran → ekstraksi teks → normalisasi
→ chunk + metadata → embedding → PostgreSQL/pgvector

QUERY
Pertanyaan → validasi → embedding → filter tenant/access
→ top-k vector search → rerank opsional → prompt berkonteks
→ model → jawaban + citation → log evaluasi aman
```

Keputusan wajib:

- ukuran chunk dan overlap;
- model embedding serta dimensinya;
- cosine distance, inner product, atau L2;
- `top_k` dan threshold minimum;
- metadata tenant, source, version, checksum;
- strategi re-index saat model embedding berubah;
- perilaku saat tidak ada bukti cukup.

## 4. ARCHITECTURE

Buat `docs/ARCHITECTURE.md` dengan:

```markdown
# ARCHITECTURE
## Context dan container diagram
## Trust boundary
## API ingest/query
## Schema: documents, chunks, embeddings, evaluations
## Isolasi tenant dan authorization
## Model serving dan resource
## RAG data flow
## Observability
## Failure modes
## Backup, RPO, RTO
## Threat model AI
## ADR
```

Ancaman minimum: prompt injection dalam dokumen, kebocoran lintas tenant, file berbahaya, exfiltration melalui prompt, denial of wallet/resource, PII pada log, dan output berbahaya.

## 5. Skema Data Minimum

| Entitas | Kolom penting |
|---|---|
| `documents` | `id`, `tenant_id`, `source`, `checksum`, `version`, `status` |
| `chunks` | `id`, `document_id`, `content`, `metadata`, `embedding` |
| `queries` | `id`, `tenant_id`, `latency_ms`, `model`, `created_at` |
| `evaluations` | `query_id`, `expected_sources`, `retrieved_sources`, `scores` |

Tambahkan filter `tenant_id` pada retrieval. Authorization terjadi sebelum query vektor, bukan setelah hasil didapat.

## 6. Dataset Evaluasi

Siapkan minimal 20 pertanyaan representatif untuk lab:

- jawaban eksplisit;
- jawaban lintas dua chunk;
- pertanyaan di luar dokumen;
- istilah Indonesia/asing;
- dokumen lama versus versi baru;
- upaya prompt injection.

Simpan expected answer/source, bukan hanya jawaban model. Jalankan baseline sebelum tuning.

## 7. TASKS dan AGENTS

Pecah pekerjaan: schema/extension, ingest, embedding, retrieval, generation, UI, evaluation, deployment, observability. Setiap task menyebut acceptance test dan resource ceiling.

`AGENTS.md` wajib melarang:

- data produksi dalam prompt agent;
- mengganti model/dimensi tanpa migrasi re-index;
- query tanpa filter tenant;
- klaim kualitas tanpa evaluasi;
- log raw document/prompt berisi PII.

## Checklist

- [ ] PRD menyebut model, embedding, lisensi, resource, latency, dan biaya.
- [ ] RAG ingest dan query flow terdokumentasi.
- [ ] Chunking, distance metric, top-k, threshold, dan citation diputuskan.
- [ ] Dataset evaluasi mencakup no-answer dan prompt injection.
- [ ] Schema menyimpan tenant, source, version, checksum, dan model embedding.
- [ ] Threat model serta retensi/penghapusan data tertulis.
- [ ] Re-index strategy tersedia untuk perubahan embedding.
- [ ] TASKS dan AGENTS memiliki acceptance gate objektif.
