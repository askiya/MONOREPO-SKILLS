# Prompt Siap Tempel

Copy, ganti bagian `<...>`, tempel ke AI agent.

---

## 1. Onboarding Sesi Baru

```text
Baca AGENTS.md, PRD.md, SDLC.md, DESIGN.md, ARCHITECTURE.md, dan TASKS.md.
Lalu jalankan git status dan git log -5 --oneline. Jangan mengubah file apa pun.

Laporkan singkat:
1. tujuan produk,
2. stack,
3. aturan keras,
4. task aktif berikutnya,
5. konflik antar-dokumen atau informasi yang belum ada.
```

---

## 2. Inisialisasi Project

```text
Kerjakan hanya T-001: inisialisasi Next.js App Router + TypeScript + Tailwind
di folder saat ini. Jangan hapus file Markdown yang sudah ada. Gunakan npm.
Setelah selesai jalankan npm run lint dan npm run build.
Laporkan output nyata dan daftar file yang berubah. Jangan commit.
```

---

## 3. Kerjakan Satu Task

```text
Kerjakan hanya task <T-0XX> di TASKS.md.
Baca file terkait sebelum mengedit. Jangan ubah dependency atau struktur folder.
Setelah implementasi, jalankan test yang relevan, lint, dan build.
Kalau ada yang belum jelas, berhenti dan tanya — jangan menebak.
Laporkan: file yang berubah, hasil verifikasi nyata, sisa masalah.
```

---

## 4. Bangun Halaman UI

```text
Kerjakan <T-0XX> saja: halaman <path>.
Baca DESIGN.md dan komponen yang sudah ada di components/. Jangan buat komponen
baru kalau yang lama bisa dipakai. Pakai data mock lokal dulu.
Wajib: loading skeleton, empty state, error state, responsive 375/768/1280.
Jangan tambah dependency. Jalankan lint dan build.
```

---

## 5. Bangun Endpoint API

```text
Kerjakan <T-0XX>: <METHOD> <path> sesuai kontrak di ARCHITECTURE.md.
Baca schema Prisma dan pola route tetangga sebelum menulis.
Validasi semua input di server. Terapkan pengecekan role sesuai matriks akses.
Jangan kembalikan field internal/sensitif.
Tambah test: sukses, input invalid, unauthorized, forbidden, duplicate.
Jalankan test, lint, build. Jangan commit.
```

---

## 6. Lapor Bug

```text
Implementasi belum sesuai.

LANGKAH:
1. <...>
2. <...>

EXPECTED: <...>
ACTUAL: <...>

OUTPUT/ERROR MENTAH:
<tempel penuh, sensor secret>

FILE TERKAIT: <...>

Perbaiki root cause, bukan gejalanya. Tambah test regresi.
Jangan ubah bagian lain yang sudah jalan.
```

---

## 7. Review Sebelum Commit

```text
Jalankan git diff. Review perubahan seperti senior engineer:
1. ada secret/key yang ikut ter-stage?
2. ada input yang tidak divalidasi di server?
3. ada error handling yang hilang?
4. ada kode mati / debug log tertinggal?
5. ada perubahan di luar scope task?

Laporkan temuan. Jangan commit. Tunggu izin saya.
```

---

## 8. Audit Responsive & Aksesibilitas

```text
Audit halaman <path> terhadap DESIGN.md, viewport 375/768/1280, dan
aksesibilitas keyboard (fokus terlihat, label input, aria-label tombol icon,
modal bisa ditutup Escape).
Perbaiki hanya masalah yang terbukti. Jangan ubah layout besar.
Jalankan lint dan build setelahnya.
```

---

## 9. Persiapan Deploy

```text
Siapkan project untuk deploy staging:
1. jalankan lint, build, test — laporkan output nyata,
2. pastikan .env tidak masuk Git dan .env.example lengkap,
3. daftar semua environment variable yang dibutuhkan di platform deploy,
4. cek hardcode localhost atau URL dev yang tertinggal.

Jangan deploy sendiri. Laporkan dulu.
```

---

## 10. Troubleshoot Deploy Gagal

```text
Deploy gagal. Berikut log mentah:
<tempel log penuh, sensor secret>

Analisis root cause dari log, bukan tebakan. Tentukan layer masalahnya:
install dependency / build / runtime / env / DB / DNS.
Usulkan diff minimum untuk memperbaiki. Jangan ganti secret tanpa bukti.
```

---

## 11. Pindah Agent / Lanjut Project Lama

```text
Baca AGENTS.md dan semua dokumen Markdown di root. Jangan mengubah apa pun.
Jalankan git status dan git log -10 --oneline.
Jelaskan kondisi project sekarang, aturan yang berlaku, task yang belum selesai,
dan risiko yang kamu lihat sebelum kita lanjut.
```

---

## 12. Tutup Sesi (lawan context rot)

```text
Sebelum saya tutup sesi:
1. update TASKS.md — centang yang selesai, tandai yang sedang jalan,
2. tulis keputusan teknis baru ke ARCHITECTURE.md,
3. tulis aturan baru yang saya koreksi hari ini ke AGENTS.md,
4. ringkas dalam 5 baris apa yang berubah hari ini.

Jangan commit.
```
