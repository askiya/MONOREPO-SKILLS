# 03 — Cara Memberi Konteks ke AI Agent

## Tujuan

Membuat agent paham project tanpa prompt raksasa dan tanpa mengarang keputusan.

## Urutan Konteks

Agent membaca dari umum ke khusus:

1. `AGENTS.md` — aturan tetap.
2. `PRD.md` — produk dan pengguna.
3. `DESIGN.md` — UI visual.
4. `ARCHITECTURE.md` — stack, folder, kontrak.
5. `TASKS.md` — task aktif.
6. File kode terkait task — detail implementasi.

Jangan selalu tempel semua isi ke chat. Minta agent membaca file langsung.

## Prompt Onboarding Sesi Baru

```text
Baca AGENTS.md, PRD.md, DESIGN.md, ARCHITECTURE.md, dan TASKS.md.
Lalu jalankan git status. Jangan mengubah file.
Laporkan singkat:
1. tujuan produk,
2. stack,
3. aturan keras,
4. task aktif berikutnya,
5. konflik atau informasi yang belum ada.
```

## Prompt Menjalankan Task

```text
Kerjakan hanya task T-012 di TASKS.md.
Baca file terkait sebelum edit. Jangan ubah dependency atau struktur folder.
Setelah implementasi, jalankan test yang relevan, lint, dan build.
Kalau ada informasi yang belum jelas, berhenti dan tanya — jangan menebak.
Laporkan file berubah, hasil verifikasi nyata, dan sisa masalah.
```

## Batas Task yang Sehat

Task bagus selesai dalam 15–90 menit dan punya hasil terlihat.

Buruk:
> Bangun dashboard admin lengkap.

Bagus:
> T-012: Tambah halaman `/admin/products` berisi tabel title, status, harga;
> data dari `GET /api/admin/products`; state loading, empty, error; responsive.

Pecah task kalau:

- menyentuh lebih dari 8–10 file,
- frontend + backend + migrasi + deploy sekaligus,
- tidak bisa dijelaskan dalam 5 kalimat,
- definisi selesai tidak bisa dites.

## Saat Agent Salah

Jangan tambah kata-kata emosional. Beri bukti:

```text
Implementasi belum sesuai.
Expected: tombol disabled ketika loading.
Actual: tombol tetap aktif dan request terkirim dua kali.
Bukti: dua request POST terlihat di Network tab.
Perbaiki root cause. Tambah test regresi. Jangan ubah desain lain.
```

## Context Rot

Chat panjang membuat agent lupa keputusan awal. Obat:

1. Commit pekerjaan selesai.
2. Update `TASKS.md`.
3. Tulis keputusan baru ke `ARCHITECTURE.md` atau `AGENTS.md`.
4. Mulai chat baru pakai prompt onboarding.

## Checklist

- [ ] Aturan tetap ada di `AGENTS.md`
- [ ] Agent diminta membaca, bukan menebak
- [ ] Satu prompt hanya satu task
- [ ] Hasil diverifikasi dengan output nyata
- [ ] Keputusan chat penting dipindah ke file
