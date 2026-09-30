# 02 — Hermes Agent untuk Multi-Service

## Tujuan

Menyiapkan Hermes Agent agar perubahan tetap kecil, terarah, dan tervalidasi pada layanan yang benar.

## Prasyarat

Dokumen dari [`01-pedoman.md`](01-pedoman.md) selesai. Instal dan konfigurasi Hermes mengikuti dokumentasi resmi terbaru: <https://hermes-agent.nousresearch.com/docs>.

## 1. Buka dari Root Proyek

```bash
cd ~/proyek-multiservice
hermes
```

**Hasil yang diharapkan:** sesi Hermes aktif dengan root proyek yang berisi `apps/`, `docs/`, dan `compose.yaml`.

## 2. Beri Konteks Berlapis

Jangan tempel seluruh repo. Beri konteks urut:

1. `docs/PRD.md` — tujuan dan acceptance criteria.
2. `docs/ARCHITECTURE.md` — batas layanan dan kontrak.
3. `AGENTS.md` — aturan kerja dan perintah validasi.
4. File layanan target saja.
5. Kontrak bersama yang langsung dipakai.

Prompt awal:

```text
Baca AGENTS.md, docs/PRD.md, dan docs/ARCHITECTURE.md. Fokus hanya task API enqueue report. Jangan ubah web, infrastruktur, atau kontrak publik. Sebelum edit, sebutkan file target dan test yang akan membuktikan hasil. Setelah edit, jalankan lint, unit test API, dan test integrasi queue. Hentikan jika kontrak dokumen tidak cukup jelas.
```

## 3. Gunakan Peta Konteks

Tambahkan ke `AGENTS.md`:

```markdown
## Peta layanan
- apps/web: Next.js; hanya HTTP ke PUBLIC_API_URL
- apps/api: NestJS/Fastify; pemilik endpoint dan migrasi
- apps/worker: BullMQ consumer; tidak membuka port
- packages/contracts: DTO/event schema berversi
- infra: Compose, Traefik, observability

## Validasi
- web: npm --workspace apps/web run lint && npm --workspace apps/web run build
- api: npm --workspace apps/api run test && npm --workspace apps/api run build
- worker: npm --workspace apps/worker run test && npm --workspace apps/worker run build
- stack: docker compose config
```

**Hasil yang diharapkan:** agent tahu batas edit dan command yang wajib lulus.

## 4. Pecah Task per Boundary

Urutan aman:

1. Ubah kontrak bersama dan test kontrak.
2. Ubah producer API dan unit test.
3. Ubah consumer worker dan test idempotensi.
4. Ubah UI dan test state.
5. Ubah Compose bila benar-benar perlu.
6. Jalankan test lintas layanan.

Jangan meminta “bangun seluruh aplikasi”. Satu prompt harus punya satu outcome yang dapat dibuktikan.

## 5. Prompt Siap Pakai

### Endpoint API

```text
Implementasikan POST /v1/reports sesuai ARCHITECTURE. Validasi input, buat record job dalam transaksi, enqueue payload report.generate.v1, balas 202. Jangan mengubah UI. Tambahkan test untuk input invalid, enqueue sukses, dan Redis gagal. Jalankan validasi API.
```

### Worker

```text
Implementasikan consumer report.generate.v1. Gunakan jobId sebagai idempotency key. Retry hanya error sementara; error permanen jangan diulang. Jangan membuka HTTP port. Tambahkan test duplicate delivery dan retry. Jalankan validasi worker.
```

### Infrastruktur

```text
Audit compose.yaml terhadap ARCHITECTURE. Pastikan postgres dan redis tanpa ports publik, healthcheck tersedia, depends_on memakai service_healthy, secret berasal dari env, dan Traefik satu-satunya ingress. Ubah hanya temuan yang terbukti. Jalankan docker compose config.
```

## 6. Review Output Agent

Periksa:

```bash
npm run lint
npm test
npm run build
docker compose config --quiet
```

**Hasil yang diharapkan:** semua command exit code `0`; agent menjelaskan perubahan kontrak dan kegagalan test, bukan menyembunyikannya.

## Masalah Umum

| Gejala | Tindakan |
|---|---|
| Agent mengubah banyak layanan | batalkan task; ulang dengan batas folder dan outcome tunggal |
| DTO API dan worker berbeda | pindahkan schema ke `packages/contracts`, tambah contract test |
| Test memakai Redis produksi | gunakan instance Compose lokal dan DB index terpisah |
| Compose rusak setelah edit | jalankan `docker compose config`; bandingkan ARCHITECTURE |
| Agent menebak secret | hentikan; gunakan nama variabel dan nilai dummy nonrahasia |

## Checklist

- [ ] Hermes dibuka dari root proyek.
- [ ] PRD, ARCHITECTURE, dan AGENTS dibaca sebelum perubahan.
- [ ] Peta konteks menyebut semua layanan dan kontrak bersama.
- [ ] Setiap prompt membatasi folder, outcome, dan test.
- [ ] Perubahan lintas layanan dikerjakan bertahap.
- [ ] Tidak ada secret dimasukkan ke prompt atau repo.
- [ ] Lint, test, build, dan `docker compose config --quiet` lulus.
