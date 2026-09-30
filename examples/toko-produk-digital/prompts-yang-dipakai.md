# Prompt yang Dipakai — SantriLearn

Ini rekaman prompt nyata per tahap. Copy boleh, tapi ganti task ID dan konteks.

## 1. Onboarding Agent

```text
Baca 01-PRD.md sampai 06-AGENTS.md. Jangan mengubah file.
Jelaskan tujuan produk, stack, aturan keras, task aktif, dan konflik dokumen.
```

**Hasil yang benar:** agent menyebut toko produk digital, Next.js+PostgreSQL,
tidak boleh commit/push, task aktif T-014. Kalau beda, jangan lanjut.

## 2. Inisialisasi

```text
Kerjakan T-001 saja. Inisialisasi Next.js 14 App Router + TypeScript + Tailwind
di folder saat ini tanpa menghapus 6 dokumen Markdown. Gunakan npm.
Jalankan npm run lint dan npm run build. Jangan commit.
```

## 3. Setup Database

```text
Kerjakan T-002 dan T-010 secara berurutan. Baca ARCHITECTURE.md dulu.
Setup Prisma PostgreSQL dengan model User, Product, Order, OrderItem/entitlement
sesuai kebutuhan. Uang integer rupiah. Email dan slug unique. Jangan jalankan
migrate reset. Buat migrasi development dan laporkan output.
```

## 4. Implementasi Register

```text
Kerjakan T-011 saja: POST /api/auth/register.
Validasi email, nama, password min 8 di server. Email unik → 409. Password
bcrypt, tidak pernah dikembalikan. Tambah test sukses, email invalid, password
pendek, duplicate. Jalankan test, lint, build. Jangan commit.
```

## 5. Halaman Katalog

```text
Kerjakan T-021 saja: halaman /produk.
Baca DESIGN.md dan komponen yang sudah ada. Gunakan GET /api/products.
Wajib loading skeleton, empty state, error+retry, grid 1/2/3 kolom pada
375/768/1024. Jangan tambah dependency. Jalankan lint dan build.
```

## 6. Webhook Payment

```text
Kerjakan T-041 saja. Ikuti dokumentasi resmi provider yang tercatat di
ARCHITECTURE.md. Verifikasi signature dari raw body, cocokkan amount dan
externalId dengan order DB, proses idempotent, dan ubah order PAID + entitlement
dalam satu transaksi DB. Tambah test invalid signature, amount salah, order
tidak ada, sukses, dan webhook duplikat. Jangan log secret.
```

## 7. Contoh Error dan Cara Melapor

### Error yang muncul

```text
Error: PrismaClient is unable to run in this browser environment,
or has been bundled for the browser
```

### Laporan buruk

```text
error bang, benerin
```

### Laporan benar

```text
TUJUAN: Menampilkan daftar produk di /produk
LANGKAH:
1. npm run dev
2. buka http://localhost:3000/produk

EXPECTED: grid produk tampil
ACTUAL: halaman blank

ERROR CONSOLE:
Error: PrismaClient is unable to run in this browser environment,
or has been bundled for the browser

FILE TERKAIT:
app/(public)/produk/page.tsx
lib/db.ts

PERUBAHAN TERAKHIR:
T-021 mengimpor prisma langsung ke client component.

Cari root cause. Perbaiki tanpa mengubah desain. Tambah test/regression check.
```

### Root cause yang diharapkan

`page.tsx` bertanda `"use client"` mengimpor Prisma server-only. Perbaikan:
ambil data lewat API atau ubah menjadi server component. Jangan memasukkan
Prisma ke bundle browser.

## 8. Persiapan Deploy

```text
Siapkan project untuk staging, tapi jangan deploy:
1. jalankan test, lint, build dan laporkan output nyata,
2. pastikan .env tidak tracked, .env.example lengkap,
3. daftar nama environment variable untuk Vercel (tanpa nilai),
4. cari hardcode localhost,
5. cek migrasi produksi.
Tunggu izin setelah laporan.
```
