# TASKS.md — SantriLearn

## Legenda
- `[ ]` belum | `[~]` sedang | `[x]` selesai
- P0 blocker, P1 penting, P2 nanti

---

## Fase 1 — Fondasi

- [x] T-001 P0 Inisialisasi Next.js 14 + TypeScript + Tailwind
      Selesai kalau: `npm run dev` jalan, `npm run build` sukses

- [x] T-002 P0 Setup Prisma + koneksi Neon PostgreSQL
      Selesai kalau: `npx prisma migrate dev` sukses, tabel muncul di Neon

- [x] T-003 P0 Komponen UI: Button, Input, Card, Badge
      File: components/ui/
      Selesai kalau: 4 komponen sesuai DESIGN.md, semua state ada

- [x] T-004 P1 Layout global: Header, Footer, Container
      Selesai kalau: tampil di semua halaman, responsive 375/768/1280

---

## Fase 2 — Auth

- [x] T-010 P0 Schema User + Product + Order di Prisma
      Selesai kalau: migrate sukses, relasi User→Order→Product benar

- [x] T-011 P0 POST /api/auth/register
      Validasi: email format, password min 8, email unik (409 kalau dobel)
      Selesai kalau: user tersimpan, password bcrypt, password tidak di response

- [x] T-012 P0 NextAuth credentials login
      Selesai kalau: login benar → session; salah → 401 pesan generik

- [x] T-013 P0 Halaman /register dan /login
      Selesai kalau: validasi client+server, loading state, error tampil di field

- [~] T-014 P0 Proteksi route (member) dan (admin)
      Selesai kalau: tanpa login → /login; member buka /admin → 403

---

## Fase 3 — Katalog

- [ ] T-020 P0 GET /api/products (pagination + filter type)
      Selesai kalau: ?page & ?limit jalan, hanya produk published tampil

- [ ] T-021 P0 Halaman /produk — grid + loading/empty/error
      Selesai kalau: grid 1/2/3 kolom, 3 state ada, skeleton saat loading

- [ ] T-022 P0 Halaman /produk/[slug] — detail
      Selesai kalau: detail tampil, slug tidak ada → 404 page, tombol Beli ada

- [ ] T-023 P1 Seed 6 produk contoh
      Selesai kalau: `npx prisma db seed` idempotent, 6 produk muncul

---

## Fase 4 — Member

- [ ] T-030 P1 Dashboard member — profil + ringkasan pesanan
- [ ] T-031 P1 Halaman /library — produk yang sudah dibeli
      Selesai kalau: hanya produk dengan order PAID milik user yang tampil

---

## Fase 5 — Payment

- [ ] T-040 P0 POST /api/checkout — buat order PENDING + invoice provider
      Selesai kalau: response berisi invoiceUrl, order tersimpan PENDING

- [ ] T-041 P0 POST /api/webhook/payment
      Wajib: verifikasi signature dari raw body, cocokkan amount + externalId,
      idempotent, order PAID + entitlement dalam satu transaksi DB
      Selesai kalau: test invalid signature, amount salah, duplikat, sukses lolos

- [ ] T-042 P1 Halaman status pesanan

---

## Fase 6 — Admin

- [ ] T-050 P1 Layout admin + proteksi role
- [ ] T-051 P1 CRUD produk admin
- [ ] T-052 P2 Daftar pesanan admin

---

## Fase 7 — Rilis

- [ ] T-060 P0 Gate testing: lint + build + test hijau
- [ ] T-061 P0 Deploy staging Vercel + uji 3 orang
- [ ] T-062 P0 Domain + HTTPS
- [ ] T-063 P0 Backup DB terjadwal + restore drill
