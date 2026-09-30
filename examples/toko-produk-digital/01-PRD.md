# PRD — Toko Produk Digital SantriLearn

## 1. Ringkasan Produk

SantriLearn adalah toko produk digital (e-book, video course, template) untuk
santri yang belajar IT. Pengguna bisa mendaftar, melihat katalog, membeli
produk, dan mengakses kontennya. Admin mengelola produk dan pesanan.

## 2. Masalah yang Diselesaikan

1. Pembuat konten IT islami sulit menjual produk digital tanpa marketplace besar.
2. Pembeli harus menunggu email manual untuk akses setelah bayar.
3. Tidak ada satu tempat yang mengumpulkan konten kelas.

## 3. Persona Pengguna

### Persona 1: Aisyah (Pembeli)
- Umur: 19
- Pekerjaan: mahasiswi IT
- Masalah: ingin beli e-book coding tapi harus transfer manual + nunggu WA
- Solusi sekarang: transfer lalu kirim bukti ke WhatsApp admin
- Harapan: bayar → langsung bisa download

### Persona 2: Ustadz Farhan (Admin / Creator)
- Umur: 28
- Pekerjaan: pengajar IT di pesantren
- Masalah: kelola pesanan manual, sering lupa kasih akses
- Harapan: pesanan tercatat otomatis, akses produk otomatis

## 4. Daftar Fitur

### MVP (Fase 1)
- [ ] F-001: Landing page — hero, fitur, pricing, CTA daftar
- [ ] F-002: Daftar akun (email + password + nama)
- [ ] F-003: Login + session
- [ ] F-004: Dashboard user — profil, daftar pesanan, produk yang dimiliki
- [ ] F-005: Katalog produk — grid + detail per produk
- [ ] F-006: Checkout → Xendit/Lynk invoice → akses otomatis setelah bayar

### Fase 2
- [ ] F-010: Dashboard admin — CRUD produk
- [ ] F-011: Admin — daftar user + pesanan
- [ ] F-012: Upload file produk (PDF/video link)

### Nanti
- [ ] F-020: Referral sederhana
- [ ] F-021: Review / rating produk
- [ ] F-022: Kupon diskon

## 5. Batasan (Out of Scope)

- Tidak ada keranjang / multi-item checkout (beli satu per satu)
- Tidak ada subscription/langganan
- Tidak ada mobile app
- Tidak ada chat/live support
- Tidak ada multi-bahasa
- Tidak ada content streaming DRM

## 6. Metrik Sukses

| Metrik | Target MVP |
|---|---|
| User bisa daftar + login dalam 2 menit | ya |
| Checkout sampai akses tanpa campur tangan admin | ya |
| Waktu load halaman utama | < 3 detik |
| Build hijau tanpa error | ya |

## 7. Dependensi Layanan Luar

| Layanan | Fungsi |
|---|---|
| Xendit / Lynk.id | Payment gateway |
| Neon / Supabase | PostgreSQL hosted |
| Vercel | Deploy staging |
| Cloudflare R2 (opsional) | Storage file produk |

## 8. Risiko

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Payment webhook gagal | user bayar tapi tidak dapat akses | retry queue + pengecekan manual admin |
| Secret bocor ke GitHub | penyalahgunaan API key | `.env` strict, scan sebelum push |
| Hosting gratis limit | site lambat saat ramai | pindah VPS saat trafik konsisten |
