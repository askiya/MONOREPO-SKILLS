# 05 — Deploy VPS, Docker Compose, dan Coolify

## Tujuan

Men-deploy stack AI pada VPS CPU atau GPU melalui Coolify, membatasi resource, menarik model, dan memverifikasi persistence.

## Prasyarat

- VPS Linux dan domain aktif.
- RAM/disk dihitung dari model, context, concurrency, DB, serta headroom.
- GPU opsional: driver NVIDIA dan Container Toolkit kompatibel telah diuji.
- Coolify dipasang dari dokumentasi resmi terbaru: <https://coolify.io/docs>.

## 1. Pilih CPU atau GPU

| Mode | Cocok | Risiko |
|---|---|---|
| CPU | demo, embedding, model kecil, traffic rendah | latency tinggi |
| GPU | chat interaktif/concurrency lebih tinggi | biaya, driver, VRAM, thermal |
| API eksternal | traffic tidak pasti | biaya per token, data keluar |

Uji model nyata. Jangan menyewa GPU hanya berdasarkan nama model.

## 2. Amankan VPS

Sebelum mengubah SSH:

1. Pastikan login SSH key berhasil pada terminal kedua.
2. Izinkan port SSH, HTTP, dan HTTPS di firewall.
3. Aktifkan firewall.
4. Uji login key lagi.
5. Baru nonaktifkan password dan root login.

Ikuti kebutuhan port resmi Coolify; jangan membuka PostgreSQL (`5432`) atau Ollama (`11434`) ke internet.

## 3. Buat Resource Compose di Coolify

1. Buat project dan environment produksi.
2. Tambahkan resource Docker Compose dari repo atau konfigurasi terkontrol.
3. Pasang volume persisten PostgreSQL dan Ollama.
4. Expose hanya `web:3000` dan `api:8000` melalui proxy Coolify.
5. Tambahkan healthcheck.
6. Pasang resource limit dan reservation.

Contoh limit awal lab, lalu ukur:

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          memory: 2G
  ollama:
    deploy:
      resources:
        limits:
          memory: 8G
```

Pastikan mode deployment Coolify menghormati field resource yang digunakan. Validasi dari metrik runtime.

## 4. Environment Variables

Set melalui secret UI Coolify:

```dotenv
DATABASE_URL=postgresql://aiapp:<ISI_SENDIRI>@postgres:5432/aiapp
POSTGRES_PASSWORD=<ISI_SENDIRI>
OLLAMA_BASE_URL=http://ollama:11434
CHAT_MODEL=llama3.2:3b
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_DIMENSION=768
RAG_TOP_K=5
RAG_SCORE_THRESHOLD=<HASIL_EVALUASI>
PUBLIC_API_URL=https://api.example.com
```

`NEXT_PUBLIC_API_URL` boleh berisi URL publik, tidak boleh secret. Validasi rentang `TOP_K`, threshold, timeout, dan ukuran upload saat startup.

## 5. Deploy dan Migrasi

Urutan:

1. Deploy PostgreSQL dan Ollama.
2. Tunggu healthcheck.
3. Jalankan migrasi pgvector satu kali.
4. Pull model.
5. Deploy API.
6. Jalankan smoke test API.
7. Deploy web.

Melalui terminal resource Ollama:

```bash
ollama pull nomic-embed-text
ollama pull llama3.2:3b
ollama list
```

**Hasil yang diharapkan:** tag model yang dipin muncul. Volume membuat model tetap ada setelah restart.

## 6. GPU Opsional

Verifikasi host:

```bash
nvidia-smi
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

**Hasil yang diharapkan:** GPU terlihat dari host dan container. Pin CUDA/image sesuai driver; jangan menebak kompatibilitas.

Pasang reservasi GPU pada Ollama sesuai dokumentasi Compose/Coolify. Setelah deploy, cek pemakaian VRAM ketika request aktif.

## 7. Verifikasi Produksi

```bash
curl --fail https://api.example.com/health
curl --fail --head https://app.example.com
```

Lakukan ingest fixture nonrahasia, retrieve, ask, dan no-answer. Restart resource lalu ulangi.

**Hasil yang diharapkan:** model serta vector persistence bertahan, health hijau, dan response memenuhi gate evaluasi.

## 8. Rollback

- Pin image API/web dan model tag.
- Migrasi schema harus backward-compatible.
- Simpan backup sebelum perubahan embedding/model.
- Rollback aplikasi dahulu; jika embedding berubah, aktifkan indeks versi lama.
- Jangan melakukan downgrade DB tanpa restore drill.

## Masalah Umum

| Gejala | Perbaikan |
|---|---|
| deploy timeout saat model pull | pull setelah runtime hidup; gunakan volume persisten |
| Ollama tak terlihat API | gunakan hostname service, bukan `localhost` |
| OOM kill | turunkan model/context/concurrency atau tambah RAM/VRAM |
| GPU tak terlihat | cek driver, toolkit, runtime, dan reservation |
| data hilang setelah redeploy | periksa mount volume dan kebijakan resource Coolify |

## Checklist

- [ ] Mode CPU/GPU dipilih dari benchmark.
- [ ] Port DB dan Ollama tidak publik.
- [ ] Volume PostgreSQL dan model persisten.
- [ ] Secret hanya berada di secret manager Coolify.
- [ ] Resource limit terpasang dan terukur.
- [ ] Migrasi berjalan satu kali sebelum API.
- [ ] Model dipin, berhasil ditarik, dan tercatat.
- [ ] Health, ingest, retrieve, ask, dan no-answer lulus.
- [ ] Restart dan rollback diuji.
