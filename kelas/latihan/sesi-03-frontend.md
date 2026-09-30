# Latihan Praktik: Sesi 3 — Build Frontend dari Nol

## Target Praktik
Membuat project Next.js dari nol, menampilkan halaman pertama di localhost.

## Estimasi Waktu
60 menit.

## Yang Harus Disiapkan
- Node.js + Git sudah jalan (sesi 1 selesai),
- dokumen perencanaan sudah ada (sesi 2 selesai),
- AI agent siap.

## Langkah

### 1. Inisialisasi Project

Kirim prompt ke agent:
```text
Inisialisasi Next.js 14 App Router + TypeScript + Tailwind CSS di folder ini.
Gunakan npm. Jangan hapus file Markdown yang ada.
Setelah selesai, jalankan npm run lint dan npm run build.
```

**Hasil yang benar:**
```text
npm run lint   → 0 error, exit code 0
npm run build  → "Compiled successfully" atau selesai tanpa error
```

**Gagal kalau:**
- `Type error` → minta agent perbaiki type
- `Module not found` → dependency kurang

### 2. Buat Halaman Landing

Kirim prompt:
```text
Buat halaman landing di app/page.tsx sesuai DESIGN.md. Isi:
- hero section: judul produk, tagline, tombol CTA
- section manfaat: 3 card
- footer: copyright
Gunakan token warna dari DESIGN.md. Responsive 375px.
Jalankan npm run build.
```

**Hasil yang benar:**
- `npm run dev` → buka `http://localhost:3000`
- landing page tampil sesuai warna DESIGN.md
- di 375px: layout stack vertikal, tidak terpotong

### 3. Cek di Browser

1. Buka `http://localhost:3000`
2. Buka DevTools (F12) → Console → **tidak ada error merah**
3. Responsive mode → pilih 375px → cek layout
4. Pilih 1280px → cek layout

### 4. Commit

```bash
git add -A
git status    # pastikan .env TIDAK ada
git commit -m "feat: landing page"
git push
```

## Cara Verifikasi
- [ ] `npm run dev` → halaman muncul
- [ ] `npm run build` → sukses
- [ ] Console: 0 error merah
- [ ] 375px: tidak terpotong
- [ ] 1280px: layout desktop wajar
- [ ] git push berhasil

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| `npm run dev` blank | import error | cek Console, perbaiki import |
| warna tidak sesuai | agent abaikan DESIGN.md | ulang prompt, sebut hex |
| mobile terpotong | fixed width | pakai `w-full`, bukan `w-[500px]` |
| `.env` ter-commit | `.gitignore` kurang | tambah `.env` ke `.gitignore` |
| build gagal setelah commit | file belum disimpan | save all, build ulang |

## Tugas Mandiri

Tambahkan 1 halaman `/tentang` yang berisi:
- foto/avatar placeholder (bisa emoji),
- deskripsi project,
- link balik ke landing.

Push ke GitHub.

## Bukti Kelulusan

Kirim ke mentor:
1. screenshot landing di 375px dan 1280px,
2. output `npm run build` (sukses),
3. link commit GitHub.
