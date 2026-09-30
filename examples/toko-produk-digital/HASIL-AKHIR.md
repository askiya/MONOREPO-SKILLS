# Hasil Akhir yang Diharapkan — SantriLearn

Dokumen ini bukan klaim bahwa aplikasi contoh sudah dibangun. Ini **rubrik hasil**
yang harus terlihat ketika member menyelesaikan project berdasarkan dokumen.

## Alur Perubahan

```text
Ide mentah: "mau bikin toko e-book"
  ↓
PRD: siapa pengguna, 6 fitur MVP, batasan jelas
  ↓
DESIGN: warna, font, spacing, komponen, responsive
  ↓
ARCHITECTURE: Next.js + Prisma + PostgreSQL + kontrak API
  ↓
TASKS: 25+ task kecil dengan definisi selesai
  ↓
Prompt satu-task → agent implementasi
  ↓
Error → laporan dengan langkah+output+expected/actual
  ↓
Localhost → QA 375/768/1280
  ↓
Test + lint + build hijau
  ↓
Vercel staging → 3 tester
  ↓
Domain + HTTPS + payment webhook → produksi
```

## Halaman yang Harus Ada

| URL | Hasil |
|---|---|
| `/` | landing: hero, manfaat, produk unggulan, CTA |
| `/produk` | katalog grid dengan filter + pagination |
| `/produk/[slug]` | detail produk + tombol beli |
| `/register` | daftar dengan validasi |
| `/login` | login + feedback error |
| `/dashboard` | ringkasan akun + pesanan |
| `/library` | produk yang sudah dibayar |
| `/admin/products` | CRUD produk khusus admin |
| `/admin/orders` | daftar pesanan khusus admin |

## Bukti Localhost

Member harus mengumpulkan:

- screenshot landing desktop (1280px),
- screenshot katalog mobile (375px),
- screenshot login error,
- screenshot dashboard setelah login,
- output `npm run test`,
- output `npm run lint`,
- output `npm run build`,
- URL commit GitHub.

## Berhasil Kalau

```text
npm run test  → semua test hijau, exit code 0
npm run lint  → 0 error, exit code 0
npm run build → "Compiled successfully" / build selesai, exit code 0
```

Build dianggap gagal kalau ada:
- `Type error`,
- `Module not found`,
- `Missing environment variable`,
- exit code selain 0.

## Bukti Staging

- URL Vercel bisa dibuka tanpa login provider,
- register + login jalan,
- database staging terpisah dari produksi,
- 3 tester mengisi QA checklist,
- tidak ada secret di GitHub.

## Bukti Produksi

- domain sendiri,
- HTTPS valid,
- payment sandbox → live diuji transaksi kecil,
- webhook duplikat tidak memberi akses dua kali,
- backup DB aktif dan restore pernah diuji,
- monitoring uptime aktif.

## Yang Sengaja Tidak Dibuat

- keranjang belanja,
- subscription,
- mobile app,
- referral,
- review/rating.

Alasan: bukan bagian MVP. Produk sederhana yang selesai lebih bernilai daripada
produk besar yang tidak bisa di-deploy.
