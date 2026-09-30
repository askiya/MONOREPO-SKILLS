# Latihan Praktik: Sesi 5 — Deploy Staging

## Target Praktik
Aplikasi bisa dibuka orang lain lewat internet, bukan cuma localhost.

## Estimasi Waktu
60 menit.

## Yang Harus Disiapkan
- project sudah di GitHub,
- `npm run build` sukses di lokal,
- daftar environment variable (nama saja).

## Langkah

### 1. Pre-Deploy Gate

```bash
npm run lint
npm run test
npm run build
git status
```

**Wajib semua hijau sebelum lanjut.** Jangan deploy build yang gagal lokal.

Cek secret:
```bash
git ls-files | grep -E "^\.env$"
```
**Hasil yang benar:** tidak ada output. Kalau `.env` muncul → hapus dari Git,
rotasi semua secret, baru lanjut.

### 2. Siapkan .env.example

```bash
DATABASE_URL=
NEXTAUTH_SECRET=
NEXTAUTH_URL=
```
Nama saja, tanpa nilai. Commit file ini.

### 3. Deploy ke Vercel

1. Buka https://vercel.com → Sign in with GitHub.
2. **Add New** → **Project**.
3. Pilih repo → **Import**.
4. Framework Preset: terdeteksi otomatis (Next.js).
5. **Environment Variables** — tambahkan satu per satu:
   ```text
   Name : DATABASE_URL
   Value: [connection string database STAGING, bukan produksi]
   ```
   Ulangi untuk semua variable di `.env.example`.
6. Klik **Deploy**.

**Hasil yang benar:** build log selesai, muncul URL `https://nama-project.vercel.app`.

**Gagal kalau:**
- `Missing environment variable` → ada env yang belum diisi, tambahkan + Redeploy
- `Type error` → perbaiki di lokal, push, deploy ulang
- `Module not found` → cek kapitalisasi nama file (Linux case-sensitive)

### 4. Uji di URL Publik

Buka URL Vercel dari **HP**, bukan laptop:
- [ ] landing tampil
- [ ] register jalan
- [ ] login jalan
- [ ] alur utama jalan
- [ ] tidak ada stack trace yang bocor saat error

### 5. Minta 3 Orang Coba

Kirim URL ke 3 teman. Minta:
- coba daftar,
- coba alur utama,
- laporkan yang aneh/membingungkan.

Catat feedback. Tidak harus langsung diperbaiki — yang penting tercatat.

## Cara Verifikasi
- [ ] URL bisa dibuka dari jaringan luar
- [ ] Database staging TERPISAH dari produksi
- [ ] `.env` tidak ada di GitHub
- [ ] 3 tester sudah mencoba
- [ ] Feedback tercatat

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| build sukses lokal, gagal Vercel | kapitalisasi file | samakan nama file persis |
| Missing env variable | belum diisi di dashboard | tambah + Redeploy |
| Prisma error runtime | `prisma generate` tidak jalan | tambah `postinstall` script |
| 500 semua halaman | DATABASE_URL salah | cek Function Logs |
| pakai DB produksi | salah connection string | buat DB staging terpisah |

## Tugas Mandiri

Tambahkan halaman `/health` yang mengembalikan:
```json
{ "status": "ok", "time": "2026-01-01T00:00:00.000Z" }
```
Deploy ulang. Buka `https://url-kamu.vercel.app/health`.

## Bukti Kelulusan

Kirim ke mentor:
1. URL staging publik,
2. akun uji (email/password khusus tes),
3. screenshot app dibuka dari HP,
4. daftar nama environment variable (tanpa nilai),
5. catatan feedback 3 tester.
