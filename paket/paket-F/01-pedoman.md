# 01 — Pedoman Perencanaan Paket F

## Tujuan

Menetapkan kontrak web, API, database, container, deploy, dan operasi sebelum agent menulis kode atau konfigurasi server.

## Prasyarat

- Scope produk sudah disepakati.
- Repo aplikasi tersedia.
- Tim mampu membaca Docker dan Nginx config.

## 1. Siapkan Enam Dokumen

```bash
cp templates/PRD.md PRD.md
cp templates/SDLC.md SDLC.md
cp templates/DESIGN.md DESIGN.md
cp templates/ARCHITECTURE.md ARCHITECTURE.md
cp templates/TASKS.md TASKS.md
cp templates/AGENTS.md AGENTS.md
wc -l PRD.md SDLC.md DESIGN.md ARCHITECTURE.md TASKS.md AGENTS.md
```

**Hasil yang diharapkan:** enam file terisi, bukan hanya judul template.

## 2. PRD: Scope Produk

Tentukan persona, masalah, alur inti, fitur versi pertama, non-scope, data sensitif, dan kriteria terima. Bedakan kebutuhan bisnis dari pilihan teknis.

## 3. SDLC: Jalur Perubahan

| Tahap | Gerbang |
|---|---|
| Lokal | lint, unit, integrasi lulus |
| Image | build reproducible dan scan tanpa temuan kritis |
| Staging | migrasi dan smoke test lulus |
| Produksi | backup tersedia, deploy disetujui |
| Pascadeploy | health, log, dan metrik normal |
| Rollback | image/commit lama dapat dijalankan |

Tentukan branch, review, version tag, migrasi, rollback aplikasi, rollback skema, dan penanggung jawab insiden.

## 4. DESIGN: Semua State

Dokumentasikan navigasi, komponen, breakpoint, token, aksesibilitas, serta state loading/kosong/error/offline/sukses. Pisahkan pesan validasi dari error server.

## 5. ARCHITECTURE: Batas Layanan

```text
Internet → Nginx
            ├── / atau domain.com → web:3000
            └── api.domain.com    → api:4000
                                      │
Docker network internal ──────────────┴── db:5432
Volume: pgdata
```

Catat kontrak:

| Komponen | Milik | Health | Data persisten |
|---|---|---|---|
| web | UI dan rendering | `/api/health` atau `/healthz` | tidak |
| api | aturan bisnis dan API | `/health` | tidak |
| db | PostgreSQL | `pg_isready` | volume `pgdata` |
| nginx | TLS dan routing | HTTP status | config host |

Tentukan kebijakan CORS, timeout, upload, rate limit, log, secret, network, port, dan dependensi. Jangan publish port database.

## 6. TASKS: Irisan Vertikal

Urutan minimum:

1. workspace, scripts, lockfile;
2. model dan migrasi database;
3. kontrak API tervalidasi;
4. satu alur web-ke-API-ke-DB;
5. test unit/integrasi;
6. Dockerfile web dan API;
7. Compose dengan healthcheck;
8. Nginx dan TLS;
9. CI/CD, backup, rollback.

Contoh:

```markdown
- [ ] F-04 — Simpan Product melalui API
  - Selesai jika: POST tervalidasi, data tersimpan, test integrasi lulus
  - Batas: apps/api dan packages/db
  - Risiko: duplikasi dan input tidak tepercaya
```

## 7. AGENTS: Batas Agent

Wajib nyatakan:

- perintah lint/test/build setiap service;
- direktori dan kepemilikan service;
- jangan baca, cetak, atau commit `.env`;
- jangan jalankan `docker compose down -v`;
- jangan membuka `5432` ke publik;
- jangan mengubah firewall/SSH/produksi tanpa persetujuan;
- migrasi produksi hanya setelah backup;
- perubahan Docker/Nginx wajib menyertakan verifikasi.

## 8. Validasi Dokumen

```bash
npm run lint
npm test
npm run build
```

**Hasil yang diharapkan:** exit code `0` untuk semua service. Jalankan script workspace resmi bila nama script berbeda.

## Kegagalan Umum

| Gejala | Perbaikan |
|---|---|
| Web mengakses DB langsung | Pindahkan aturan/data lewat API kecuali arsitektur sengaja menetapkan BFF |
| Semua port dipublish | Publish hanya port yang perlu; DB tetap internal |
| Rollback hanya “git checkout” | Catat image/tag, migrasi kompatibel, dan perintah rollback |
| Secret masuk Compose | Gunakan `.env` server yang tidak dilacak |

## Checklist

- [ ] PRD punya scope dan kriteria terukur.
- [ ] SDLC mencakup image, staging, deploy, rollback.
- [ ] DESIGN mencakup aksesibilitas dan semua state.
- [ ] ARCHITECTURE menjelaskan web, API, DB, Nginx, network, volume.
- [ ] TASKS kecil dan dapat diverifikasi.
- [ ] AGENTS melarang operasi destruktif dan kebocoran secret.
- [ ] Perintah quality gate lulus.
