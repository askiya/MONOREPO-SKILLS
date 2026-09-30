# 01 — Testing Sebelum Deploy

## Tujuan

Memastikan kode tidak rusak sebelum orang lain melihatnya.

## Tiga Jenis Test

### 1. Lint (kualitas kode)

```bash
npm run lint
```

Memperbaiki: gaya, import tidak dipakai, kesalahan umum. Kalau ESLint belum
dikonfigurasi, tambahkan. Agent bisa mengotomatisasi setup awal.

### 2. Build (kompilasi)

```bash
npm run build
```

Menemukan: error TypeScript, import rusak, halaman yang crash saat render.
Build sukses bukan berarti benar secara fungsi, tapi gagal artinya pasti salah.

### 3. Test Otomatis

Unit + integration:

```bash
npm run test
```

Minimum yang harus punya test:

- Validasi input endpoint utama (happy + beberapa unhappy path).
- Fungsi bisnis kritis (hitung harga, kalkulasi diskon).
- Middleware auth: role benar lolos, role salah ditolak, tanpa token ditolak.
- Webhook parsing: kalau ada payment webhook, verifikasi signature.

### 4. Manual QA

Buat checklist manual untuk UI yang sulit diotomasi:

```markdown
## Manual QA Checklist
- [ ] Mobile: hamburger menu buka dan tutup
- [ ] Loading state terlihat saat API lambat (throttle network)
- [ ] Empty state tampil saat data kosong
- [ ] Error boundary tidak menampilkan stack trace
- [ ] Form tidak bisa submit ganda saat loading
- [ ] Redirect setelah login ke halaman tujuan awal
```

## Gate: Kapan Boleh Deploy

```
✅ npm run lint      → 0 error
✅ npm run build     → sukses
✅ npm run test      → semua hijau
✅ Manual QA         → semua centang
✅ git status bersih → tidak ada file tercecer
```

Kurang satu = belum boleh deploy.

## Prompt Agent untuk Test

```text
Baca file T-031 dan tulis test untuk POST /api/products: kasus sukses,
invalid input (title kosong, harga negatif), unauthorized (tanpa token),
forbidden (member bukan admin), duplicate slug (409). Jangan mock database
kalau bisa pakai test database terpisah. Jalankan test dan laporkan hasilnya.
```

## Checklist

- [ ] `npm run lint` 0 error
- [ ] `npm run build` sukses
- [ ] `npm run test` hijau
- [ ] Manual QA dicentang
- [ ] Tidak ada file liar di `git status`
