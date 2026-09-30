# 08 — Rekomendasi Hosting Self-hosted AI Stack

> 🔴 EXPERT · Stack: Next.js + FastAPI + PostgreSQL + pgvector + Ollama/vLLM + Docker Compose + Coolify

## Tujuan

Memilih infrastruktur yang cukup untuk model, API, database vektor, dan dokumen RAG tanpa menyewa GPU sebelum kebutuhan terbukti.

> Harga, stok GPU, kurs, kuota, dan spesifikasi provider dapat berubah. Angka di dokumen ini adalah perkiraan awal per September 2026. Periksa situs resmi sebelum membeli.

## Prasyarat

- Model, quantization, context window, dan target concurrency sudah ditentukan.
- Kebutuhan RAM/VRAM diuji dengan model dan prompt nyata.
- Ukuran dokumen, pertumbuhan embedding, dan kebijakan retensi sudah dihitung.
- Tim mampu mengelola Linux, Docker Compose, firewall, backup, monitoring, dan insiden.
- Secret tidak tersimpan di repository.

## Arsitektur yang Direkomendasikan

```text
Pengguna
   └── Cloudflare DNS/Proxy
          ├── app.domainmu.com → Next.js
          └── api.domainmu.com → FastAPI
                                  ├── PostgreSQL + pgvector
                                  ├── Ollama atau vLLM
                                  └── R2/B2/S3 untuk dokumen RAG

Coolify mengelola service Docker Compose, domain, TLS, log, dan deployment.
```

Gunakan network internal untuk komunikasi antarkontainer. Jangan membuka PostgreSQL (`5432`), Ollama (`11434`), atau endpoint administrasi model ke internet.

## 1. VPS CPU

CPU inference cocok untuk development, demo, embedding, batch ringan, dan model kecil dengan traffic rendah. Untuk model 7B quantized, mulai dari 8–16 GB RAM lalu ukur latency dan pemakaian nyata.

| Provider | Spesifikasi awal | Perkiraan biaya | Keterangan | Link resmi |
|---|---|---:|---|---|
| Hetzner Cloud CPX/CCX | 8–16 GB RAM | Mulai sekitar €13–25/bulan | Nilai resource bagus; periksa region dan ketersediaan | [hetzner.com/cloud](https://www.hetzner.com/cloud/) |
| DigitalOcean Droplet | 8 GB RAM | Mulai sekitar US$48/bulan | Dashboard dan dokumentasi mudah diikuti | [digitalocean.com/pricing/droplets](https://www.digitalocean.com/pricing/droplets) |
| Vultr Cloud Compute | 8 GB RAM | Mulai sekitar US$48/bulan | Banyak pilihan region | [vultr.com/pricing](https://www.vultr.com/pricing/) |

Angka tersebut belum tentu mencakup backup, snapshot, volume tambahan, bandwidth keluar, IPv4, dan pajak.

### Patokan RAM model

- Model 7B quantized: sekitar 4–8 GB untuk bobot model.
- Model 13B quantized: sekitar 8–16 GB; sediakan setidaknya sekitar 16 GB RAM host.
- Model 70B: umumnya memerlukan GPU dan VRAM besar atau beberapa GPU.

Bobot model bukan satu-satunya pemakai memori. Tambahkan ruang untuk KV cache, context, concurrency, runtime, FastAPI, Next.js, PostgreSQL, indeks pgvector, Coolify, dan filesystem cache. Jangan membeli VPS tepat sebesar file model.

## 2. VPS atau Cloud GPU

GPU inference cocok untuk model besar, latency interaktif, context panjang, atau banyak request bersamaan.

| Provider | Model layanan | Cocok untuk | Perhatian | Link resmi |
|---|---|---|---|---|
| RunPod | GPU sesuai permintaan dan Community Cloud | Eksperimen, worker sementara, inference elastis | Periksa reputasi host, storage persisten, egress, dan waktu aktif | [runpod.io](https://www.runpod.io/) |
| Vast.ai | Marketplace GPU | Mencari harga GPU murah | Kualitas host bervariasi; cek reliability, disk, jaringan, dan keamanan | [vast.ai](https://vast.ai/) |
| Lambda Cloud | Instance GPU termasuk A100/H100 sesuai ketersediaan | Workload produksi dan model besar | Stok region dapat terbatas; hitung biaya per jam penuh | [lambda.ai/service/gpu-cloud](https://lambda.ai/service/gpu-cloud) |
| DigitalOcean GPU Droplets | GPU cloud terkelola | Tim yang sudah memakai DigitalOcean | Periksa jenis GPU, minimum sewa, region, dan biaya volume | [digitalocean.com/products/gpu-droplets](https://www.digitalocean.com/products/gpu-droplets) |
| Hetzner Dedicated GPU | Server dedicated dengan GPU tertentu | Beban stabil jangka panjang | Komitmen dan operasi server lebih besar; cek stok serta lokasi | [hetzner.com/dedicated-rootserver](https://www.hetzner.com/dedicated-rootserver/) |

Untuk GPU sementara, pisahkan server aplikasi/database dari worker inference. Server utama tetap hidup, sedangkan GPU dapat dimatikan ketika tidak digunakan. Lindungi koneksi inference dengan jaringan privat, VPN, atau autentikasi kuat.

## 3. Tabel Sizing CPU vs GPU

| Skenario | Model awal | Compute awal | RAM/VRAM awal | Cocok untuk | Keputusan naik kelas |
|---|---|---|---|---|---|
| Embedding dan RAG kecil | Model embedding + 1B–3B quantized | 4 vCPU | 8 GB RAM | Lab, ingest kecil, demo | RAM sering di atas 80% atau latency gagal target |
| Chat 7B traffic rendah | 7B quantized | 8 vCPU | 16 GB RAM | Internal tool, pengguna sedikit | P95 terlalu lambat atau antrean bertambah |
| Chat 13B traffic rendah | 13B quantized | CPU kelas tinggi | 24–32 GB RAM | Batch dan penggunaan nonreal-time | Butuh respons interaktif atau concurrency naik |
| Chat 7B–13B interaktif | 7B–13B quantized | 1 GPU | Sekitar 12–24 GB VRAM | Aplikasi chat produksi kecil | OOM, context tidak muat, atau throughput kurang |
| Model 30B–70B | Quantized sesuai kualitas target | GPU besar atau multi-GPU | Sekitar 48–80+ GB VRAM total | Kualitas model lebih tinggi | Benchmark menunjukkan kebutuhan tensor parallel atau GPU tambahan |
| API model eksternal | Model provider | VPS aplikasi 4–8 GB RAM | Tidak ada VRAM lokal | Traffic belum pasti | Biaya token stabil lebih mahal daripada GPU terukur |

Tabel ini titik awal, bukan jaminan. Ukur token per detik, time-to-first-token, P50/P95 latency, concurrency, RAM/VRAM puncak, dan kualitas jawaban dengan workload produksi.

## 4. PostgreSQL + pgvector

### Pilihan utama: container self-hosted

Jalankan image PostgreSQL yang mendukung extension `vector`, pasang volume persisten, dan akses hanya dari network internal Coolify.

Kebutuhan minimum:

1. Pin versi PostgreSQL dan pgvector.
2. Aktifkan extension melalui migrasi:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

3. Gunakan indeks HNSW atau IVFFlat hanya setelah mengukur dataset dan pola query.
4. Pantau ukuran tabel, indeks, WAL, koneksi, dan query lambat.
5. Kirim backup ke lokasi di luar VPS dan uji restore.

### Alternatif terkelola

| Provider | Dukungan pgvector | Cocok untuk | Perhatian | Link resmi |
|---|---|---|---|---|
| Neon | Ya | Database serverless dan pemisahan compute dari VPS AI | Periksa compute, storage, batas koneksi, region, dan egress | [neon.tech/docs/extensions/pgvector](https://neon.tech/docs/extensions/pgvector) |
| Supabase | Ya | PostgreSQL dengan dashboard dan layanan tambahan | Periksa ukuran compute, backup, pooling, dan batas paket | [supabase.com/docs/guides/database/extensions/pgvector](https://supabase.com/docs/guides/database/extensions/pgvector) |

Database terkelola mengurangi pekerjaan operasi, tetapi latency jaringan ke API/inference harus diuji. Pilih region sedekat mungkin dengan VPS.

## 5. Object Storage Dokumen RAG

Jangan menyimpan dokumen upload hanya di filesystem container. Gunakan object storage dengan bucket privat.

| Provider | Kelebihan | Perhatian | Link resmi |
|---|---|---|---|
| Cloudflare R2 | API kompatibel S3 dan egress internet tanpa biaya pada skema umum R2 | Periksa biaya operasi, penyimpanan, dan batas terbaru | [cloudflare.com/developer-platform/products/r2](https://www.cloudflare.com/developer-platform/products/r2/) |
| Backblaze B2 | Biaya penyimpanan kompetitif dan dukungan API S3 | Hitung download, transaksi, dan integrasi CDN | [backblaze.com/cloud-storage](https://www.backblaze.com/cloud-storage) |
| Amazon S3 | Ekosistem matang, lifecycle, versioning, dan banyak region | Struktur biaya lebih rumit; periksa request dan egress | [aws.amazon.com/s3/pricing](https://aws.amazon.com/s3/pricing/) |

Aktifkan enkripsi, versioning bila perlu, lifecycle retention, validasi tipe/ukuran file, dan URL bertanda tangan. Simpan object key di database; jangan menjadikan bucket publik.

## 6. Coolify dan Domain

Gunakan Coolify self-hosted untuk mengelola resource Docker Compose, deploy dari repository, environment variable, domain, TLS, health check, dan log.

Gunakan Cloudflare untuk DNS:

| Hostname | Target |
|---|---|
| `app.domainmu.com` | Service Next.js melalui proxy Coolify |
| `api.domainmu.com` | Service FastAPI melalui proxy Coolify |
| `panel.domainmu.com` | Panel Coolify yang dibatasi aksesnya |

Gunakan mode SSL/TLS **Full (strict)** setelah sertifikat origin valid. Jangan membuka panel Coolify tanpa password kuat, 2FA bila tersedia, dan pembatasan akses.

## 7. Peringatan Biaya GPU

1. GPU ditagih per jam dan dapat tetap berjalan saat tidak menerima request. Matikan instance idle bila arsitektur mengizinkan.
2. Disk persisten, snapshot, bandwidth keluar, public IP, dan pajak dapat ditagih terpisah.
3. Download model berulang memperpanjang waktu sewa. Gunakan volume atau cache persisten yang aman.
4. Marketplace murah memiliki variasi reliability. Jangan menaruh database produksi pada host GPU sementara.
5. GPU besar belum tentu lebih hemat. Bandingkan biaya per 1.000 request atau per satu juta token dengan API eksternal.
6. Pasang budget alert dan batas pengeluaran. Jangan mengandalkan pemeriksaan manual.
7. Benchmark sebelum kontrak bulanan. Uji model, quantization, context, batch, dan concurrency yang sama dengan produksi.

## 8. Rekomendasi Berdasarkan Tahap

| Tahap | Rekomendasi | Alasan |
|---|---|---|
| Belajar/demo | VPS CPU 8–16 GB + model 1B–7B quantized | Biaya terkendali dan operasi sederhana |
| MVP traffic tidak pasti | VPS aplikasi/database + API model eksternal atau GPU sesuai permintaan | Tidak membayar GPU 24 jam sebelum ada beban |
| Produksi kecil stabil | VPS aplikasi/database terpisah + satu GPU terukur | Gangguan inference tidak langsung menjatuhkan database |
| Produksi besar | Database terkelola, object storage, beberapa worker vLLM, load balancer | Skalabilitas dan isolasi workload lebih baik |

## 9. Verifikasi Setelah Deploy

```bash
curl --fail --head https://app.domainmu.com
curl --fail https://api.domainmu.com/health
```

Lakukan satu alur lengkap: upload dokumen nonrahasia, ekstraksi, embedding, penyimpanan, retrieval, jawaban, dan penghapusan. Restart seluruh stack lalu ulangi untuk membuktikan persistence.

Pantau:

- RAM, swap, disk, CPU, dan load average;
- VRAM, utilisasi GPU, suhu, dan error driver;
- token per detik, time-to-first-token, P95 latency, serta antrean;
- koneksi DB, query lambat, ukuran tabel dan indeks;
- kapasitas bucket, kegagalan upload, serta biaya egress.

## Checklist

### Kapasitas

- [ ] Model, quantization, context, dan concurrency sudah dipin.
- [ ] RAM/VRAM diuji dengan request nyata, bukan hanya dihitung dari ukuran file model.
- [ ] Tersedia headroom untuk KV cache, runtime, database, dan Coolify.
- [ ] Keputusan CPU, GPU, atau API eksternal didasarkan pada benchmark.
- [ ] Budget alert aktif, termasuk GPU, disk, snapshot, dan egress.

### Keamanan dan data

- [ ] PostgreSQL, Ollama/vLLM, dan panel administrasi tidak terbuka tanpa perlindungan.
- [ ] Bucket dokumen privat dan memakai URL bertanda tangan.
- [ ] Secret hanya berada di secret manager Coolify.
- [ ] Upload divalidasi berdasarkan ukuran, tipe, dan hak akses.
- [ ] Backup PostgreSQL dikirim keluar VPS dan restore sudah diuji.
- [ ] Dokumen RAG memiliki kebijakan retensi dan penghapusan.

### Operasional

- [ ] Volume PostgreSQL, model, dan file yang perlu bertahan sudah persisten.
- [ ] Health check setiap service aktif.
- [ ] Monitoring RAM, VRAM, disk, latency, dan error aktif.
- [ ] Restart membuktikan model, data, dan indeks tetap tersedia.
- [ ] Rollback image, model, migrasi, dan embedding pernah disimulasikan.
- [ ] Harga serta batas paket sudah diperiksa pada situs resmi terbaru.
