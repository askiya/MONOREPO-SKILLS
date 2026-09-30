# 04 — Aturan Main Kelas

## Tujuan

Kesepakatan kerja supaya hasil kelas ini bukan sekadar demo, tapi produk yang
bisa dipakai orang.

## Sepuluh Aturan

1. **Dokumen dulu.** Tidak ada kode sebelum PRD + ARCHITECTURE ada.
2. **Satu task satu commit.** Commit message jelas, bukan "update".
3. **Baca diff sebelum menerima.** Minimal tahu file apa yang berubah dan kenapa.
4. **Secret tidak pernah masuk Git.** Cek `git status` sebelum commit.
5. **Localhost dulu.** Belum jalan lokal = belum boleh deploy.
6. **Gate testing wajib.** lint + build + test hijau sebelum deploy.
7. **Staging sebelum produksi.** Selalu ada URL uji coba.
8. **Rollback harus mungkin.** Tahu cara balik ke versi sebelumnya.
9. **Tulis ulang aturan baru yang kamu temukan** ke `AGENTS.md` project kamu.
10. **Kalau agent ngeyel salah 3 kali, berhenti.** Baca kode sendiri, perbaiki
    manual, lalu lanjut. Jangan loop tak berujung.

## Definisi "Selesai" untuk Satu Fitur

Sebuah fitur selesai kalau:

- [ ] Berfungsi di localhost
- [ ] Tidak ada error merah di console
- [ ] State kosong / loading / error sudah ditangani
- [ ] Tampil benar di layar HP (lebar 375px)
- [ ] `npm run build` sukses
- [ ] Sudah di-commit

Kurang satu = belum selesai. Titik.

## Cara Melapor Bug ke AI Agent

Format yang bikin agent langsung paham:

```
LANGKAH:
1. Buka /login
2. Isi email salah
3. Klik Masuk

DIHARAPKAN: muncul pesan "Email atau password salah"
KENYATAAN: halaman blank, console error:
<tempel error mentah di sini>

FILE TERKAIT: app/login/page.tsx, app/api/auth/route.ts
```

Jangan bilang "error nih, tolong benerin". Agent akan menebak, dan tebakannya
akan salah.

## Checklist

- [ ] Saya setuju 10 aturan di atas
- [ ] Saya hafal definisi "selesai"
- [ ] Saya tahu format lapor bug
