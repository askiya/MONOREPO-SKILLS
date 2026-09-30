# SDLC — Toko Produk Digital SantriLearn

## Fase 1 — Perencanaan
- **Durasi**: 2 hari
- **Input**: ide, riset toko digital kompetitor
- **Output**: PRD.md final
- **Selesai kalau**: setiap fitur MVP bisa dijawab "kenapa" dan "untuk siapa"

## Fase 2 — Desain & Arsitektur
- **Durasi**: 1 hari
- **Input**: PRD
- **Output**: DESIGN.md, ARCHITECTURE.md, TASKS.md, AGENTS.md
- **Selesai kalau**: agent bisa merangkum 4 dokumen tanpa kontradiksi

## Fase 3 — Build MVP
- **Durasi**: 7–10 hari
- **Input**: semua dokumen + TASKS.md
- **Output**: app jalan di localhost, F-001–F-006 berfungsi
- **Selesai kalau**:
  - landing page tampil,
  - register + login jalan,
  - katalog bisa dilihat,
  - checkout menghasilkan invoice,
  - `npm run build` sukses

## Fase 4 — Testing
- **Durasi**: 2 hari
- **Input**: app localhost
- **Output**: lint + build + test hijau, bug kritis 0
- **Selesai kalau**: checklist testing tercentang semua

## Fase 5 — Deploy Staging
- **Durasi**: 1 hari
- **Input**: build hijau
- **Output**: URL publik staging di Vercel
- **Selesai kalau**: 3 orang sudah mencoba + feedback dicatat

## Fase 6 — Produksi
- **Durasi**: 2 hari
- **Input**: staging teruji
- **Output**: domain sendiri, HTTPS, payment live
- **Selesai kalau**: transaksi percobaan kecil berhasil end-to-end
