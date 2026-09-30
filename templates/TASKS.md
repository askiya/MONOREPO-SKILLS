# TASKS.md

## Legenda
- `[ ]` belum | `[~]` sedang | `[x]` selesai
- Prioritas: P0 (blocker), P1 (penting), P2 (nanti)

---

## Fase 1 — Fondasi

- [ ] T-001 P0 Inisialisasi Next.js + TypeScript + Tailwind
      Selesai kalau: `npm run dev` jalan, halaman kosong tampil, `npm run build` sukses

- [ ] T-002 P0 Setup Prisma + koneksi PostgreSQL
      Selesai kalau: `npx prisma migrate dev` sukses, tabel muncul di DB

- [ ] T-003 P0 Komponen UI dasar: Button, Input, Card
      File: components/ui/
      Selesai kalau: 3 komponen sesuai DESIGN.md, semua state ada

- [ ] T-004 P1 Layout global: header, footer, container
      Selesai kalau: tampil di semua halaman, responsive 375/768/1280

---

## Fase 2 — Auth

- [ ] T-010 P0 POST /api/auth/register
      Validasi: email format, password min 8, email unik
      Selesai kalau: user tersimpan, password ter-hash, tidak dikembalikan di response

- [ ] T-011 P0 POST /api/auth/login + session
      Selesai kalau: login benar → session aktif; salah → 401 dengan pesan generik

- [ ] T-012 P0 Halaman /register dan /login
      Selesai kalau: form validasi client + server, loading state, error tampil

- [ ] T-013 P0 Proteksi route member
      Selesai kalau: akses tanpa login → redirect /login; test 401/403 lolos

---

## Fase 3 — Fitur Inti

- [ ] T-020 P0 GET /api/products (list + pagination)
- [ ] T-021 P0 Halaman /produk (grid + loading/empty/error)
- [ ] T-022 P0 Halaman /produk/[slug] (detail)
- [ ] T-023 P1 Dashboard member

---

## Fase 4 — Admin

- [ ] T-030 P1 Layout admin + proteksi role
- [ ] T-031 P1 CRUD produk admin
- [ ] T-032 P2 Kelola user

---

## Fase 5 — Payment

- [ ] T-040 P0 POST /api/checkout → buat invoice provider
- [ ] T-041 P0 Webhook payment (verifikasi signature, idempotent, atomik)
- [ ] T-042 P1 Halaman status pesanan

---

## Fase 6 — Rilis

- [ ] T-050 P0 Gate testing: lint + build + test hijau
- [ ] T-051 P0 Deploy staging + uji 3 orang
- [ ] T-052 P0 Deploy produksi + domain + HTTPS
- [ ] T-053 P0 Backup database terjadwal + restore drill
