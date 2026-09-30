# 02 — Hermes Agent untuk Self-hosted AI Stack

## Tujuan

Menggunakan Hermes Agent sebagai engineer pendamping tanpa mencampur data sensitif, kontrak API, evaluasi model, dan konfigurasi infrastruktur.

## Prasyarat

Dokumen [`01-pedoman.md`](01-pedoman.md) selesai. Instal/config Hermes dari dokumentasi resmi terbaru: <https://hermes-agent.nousresearch.com/docs>.

## 1. Mulai dari Root

```bash
cd ~/produk-ai
hermes
```

**Hasil yang diharapkan:** Hermes melihat `apps/web`, `apps/api`, `evals`, `docs`, dan `compose.yaml`.

## 2. Susun Konteks Advanced

Berikan berurutan:

1. `AGENTS.md` untuk aturan global.
2. `docs/PRD.md` untuk acceptance criteria AI.
3. `docs/ARCHITECTURE.md` untuk trust boundary dan RAG flow.
4. `evals/dataset.jsonl` versi sintetis/nonrahasia.
5. Hanya file komponen target.

Jangan kirim dokumen pelanggan, token, prompt produksi, atau dump DB ke agent eksternal.

Prompt awal:

```text
Baca AGENTS.md, PRD, ARCHITECTURE, dan dataset evaluasi. Fokus hanya retrieval service. Jangan ubah model, dimensi embedding, schema tenant, prompt generation, atau infra. Implementasikan acceptance criteria yang tertulis. Jalankan unit test, integration test pgvector, dan evaluasi retrieval. Laporkan metrik sebelum/sesudah; jangan menyatakan peningkatan tanpa data.
```

## 3. Peta Layanan

Tambahkan ke `AGENTS.md`:

```markdown
## Peta
- apps/web: Next.js UI; tidak memegang DB/model secret
- apps/api: FastAPI; auth, ingest, retrieval, generation
- apps/api/rag: chunk/embed/retrieve; fungsi murni bila mungkin
- evals: dataset dan runner deterministik
- postgres: metadata + pgvector
- ollama: model runtime internal
- infra: Compose/Coolify

## Gate
- python -m ruff check apps/api
- python -m pytest apps/api/tests
- npm --prefix apps/web run lint
- npm --prefix apps/web run build
- python evals/run.py --dataset evals/dataset.jsonl
- docker compose config --quiet
```

## 4. Pecah Task Berbasis Eksperimen

Satu eksperimen mengubah satu variabel:

1. Baseline dan simpan metrik.
2. Ubah chunk size **atau** top-k, bukan keduanya.
3. Jalankan dataset sama.
4. Bandingkan kualitas, latency, memory.
5. Terima atau rollback berdasarkan threshold PRD.

Dengan ini agent tidak “meningkatkan” hasil lewat perubahan tak terlacak.

## 5. Prompt Siap Pakai

### Ingest

```text
Implementasikan ingest file teks sesuai ARCHITECTURE: validasi ukuran/tipe, checksum deduplication, chunk metadata, embedding batch, transaksi status. Isolasi tenant wajib. Tambahkan test file invalid, duplicate, partial failure, dan tenant. Jangan menyentuh generation.
```

### Retrieval

```text
Implementasikan cosine retrieval pgvector dengan filter tenant sebelum ranking, top_k dan threshold dari config tervalidasi. Kembalikan source_id, chunk_id, score. Tambahkan test kebocoran lintas tenant dan no-result.
```

### Evaluasi

```text
Jalankan baseline evals/dataset.jsonl. Laporkan recall@5, MRR, no-answer precision, dan p95 retrieval latency. Jangan ubah dataset. Jika gate gagal, jelaskan sampel gagal dan usulkan satu eksperimen berikutnya.
```

## 6. Review Manual

```bash
python -m ruff check apps/api
python -m pytest apps/api/tests
npm --prefix apps/web run lint
npm --prefix apps/web run build
python evals/run.py --dataset evals/dataset.jsonl
docker compose config --quiet
```

**Hasil yang diharapkan:** command exit `0`; laporan eval memakai dataset dan model version yang tercatat.

## Masalah Umum

| Gejala | Tindakan |
|---|---|
| agent mengganti library/model | kunci dependency dan model tag; ulang task sempit |
| eval terlihat naik drastis | cek dataset tidak bocor/berubah dan cache dibersihkan |
| dimensi vector tidak cocok | hentikan; migrasi/re-index eksplisit |
| contoh berisi data pelanggan | hapus dari konteks dan rotasi secret jika ikut terpapar |
| retrieval lintas tenant | anggap insiden keamanan; blok deploy dan tambah test regresi |

## Checklist

- [ ] Hermes dibuka dari root proyek.
- [ ] Konteks memuat PRD, arsitektur, gate, dan dataset aman.
- [ ] Data pelanggan dan secret tidak dikirim ke agent.
- [ ] Task membatasi folder serta satu variabel eksperimen.
- [ ] Dataset/model/version tercatat pada hasil evaluasi.
- [ ] Test isolasi tenant dan no-answer lulus.
- [ ] Lint, test, build, eval, dan Compose config lulus.
