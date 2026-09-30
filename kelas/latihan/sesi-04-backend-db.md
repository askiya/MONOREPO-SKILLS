# Latihan Praktik: Sesi 4 — Backend API + Database

## Target Praktik
Setup database, buat model, buat API endpoint pertama, akses dari localhost.

## Estimasi Waktu
90 menit.

## Yang Harus Disiapkan
- project Next.js sudah jalan (sesi 3 selesai),
- akun database (Neon / Supabase / Aiven — pilih satu, pakai free tier),
- AI agent siap.

## Langkah

### 1. Buat Database

1. Buka dashboard provider (contoh: https://neon.tech).
2. Buat project baru → dapat connection string.
3. **JANGAN** tempel connection string ke chat / file yang masuk Git.
4. Buat file `.env`:
   ```bash
   DATABASE_URL="postgresql://user:password@host/dbname?sslmode=require"
   ```
5. Pastikan `.env` ada di `.gitignore`.

### 2. Setup Prisma

Prompt ke agent:
```text
Install Prisma. Buat schema sesuai ARCHITECTURE.md (minimal model User dan
Product). Email dan slug unique. Uang integer rupiah. Jalankan migrasi development.
Jangan jalankan migrate reset.
```

**Hasil yang benar:**
```text
npx prisma migrate dev --name init
→ migration created, applied
```

Verifikasi:
```bash
npx prisma studio
```
→ buka browser, tabel terlihat.

**Gagal kalau:**
- `P1001 Can't reach database` → cek connection string, IP allowlist
- `password authentication failed` → encode karakter khusus (`@ → %40`)

### 3. Buat API Endpoint

Prompt:
```text
Buat GET /api/products yang mengembalikan semua produk published.
Pagination: ?page=1&limit=10. Buat juga seed 5 produk contoh (npx prisma db seed).
Jalankan seed lalu test endpoint di browser.
```

**Hasil yang benar:**
- Buka `http://localhost:3000/api/products` → JSON array produk
- `?page=2` → halaman 2 atau array kosong

### 4. Test Manual

```bash
# Pastikan masih jalan
npm run lint
npm run build
```

### 5. Commit

```bash
git add -A
git status   # WAJIB: .env TIDAK boleh ada di daftar
git commit -m "feat: database + product API"
git push
```

## Cara Verifikasi
- [ ] `npx prisma studio` → tabel ada, kolom benar
- [ ] `/api/products` → JSON produk
- [ ] Seed idempotent (jalankan dua kali, tidak dobel)
- [ ] `.env` TIDAK ada di `git status`
- [ ] `npm run build` → sukses

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| P1001 Can't reach DB | connection string / IP | cek URL, allowlist |
| password failed | karakter khusus | URL-encode: `@ → %40` |
| table already exists | migrasi ulang | hapus migrasi lama di folder |
| PrismaClient browser | import di client component | pakai server-only import |
| seed dobel | tidak idempotent | pakai upsert, bukan create |

## Tugas Mandiri

Tambahkan model `Category` (id, name, slug). Relasikan ke Product.
Buat seed 3 kategori. Buat endpoint `/api/categories`.
Push ke GitHub.

## Bukti Kelulusan

Kirim ke mentor:
1. screenshot Prisma Studio (tabel terlihat),
2. screenshot `/api/products` di browser (JSON),
3. output `npm run build` (sukses),
4. link commit GitHub.
