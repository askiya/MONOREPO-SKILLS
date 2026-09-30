# Latihan Praktik: Sesi 2 — Menulis Dokumen Perencanaan

## Target Praktik
Membuat PRD, DESIGN.md, ARCHITECTURE.md, TASKS.md, dan AGENTS.md
untuk project sendiri. Bukan copy-paste — diisi berdasarkan ide sendiri.

## Estimasi Waktu
90 menit.

## Yang Harus Disiapkan
- ide produk (sederhana, boleh toko/blog/portfolio),
- folder template sudah di-clone dari repo MONOREPO-SKILLS,
- AI agent siap.

## Langkah

### 1. Tentukan Produk

Jawab 3 pertanyaan:
1. Produk apa?
2. Untuk siapa?
3. Fitur apa yang **paling penting** (maksimal 5)?

Tulis jawaban di PRD.md bagian "Ringkasan" dan "Target Pengguna".

### 2. Isi PRD.md

Copy `templates/PRD.md` ke folder project. Isi setiap bagian.

**Hasil yang benar:**
- Setiap fitur MVP punya alasan "kenapa".
- "Batasan (Out of Scope)" terisi — bukan kosong.
- Tidak ada fitur yang deskripsinya hanya "nanti".

**Gagal kalau:** ada bagian kosong atau hanya "TODO".

### 3. Isi DESIGN.md

Copy `templates/DESIGN.md`. Isi:
- 5 warna (primary, secondary, bg, teks, error),
- font choice,
- border-radius,
- spacing scale.

Boleh pakai AI agent: "Sarankan palet warna untuk toko e-book edukatif
yang clean dan profesional, berikan hex code."

**Hasil yang benar:** setiap token punya hex/nilai, bukan "warna biru".

### 4. Isi ARCHITECTURE.md dan TASKS.md

- ARCHITECTURE: stack + alasan, folder structure, kontrak API.
- TASKS: minimal 10 task, setiap task punya definisi selesai.

**Gagal kalau:** task deskripsinya "bikin frontend" tanpa detail.

### 5. Isi AGENTS.md

Aturan keras minimal 5 baris. Perintah tabel wajib ada.

### 6. Uji dengan Agent

Kirim ke AI agent:
```text
Baca semua file Markdown di folder ini. Jelaskan:
1. Tujuan produk
2. Stack yang dipilih
3. Aturan keras
4. Task aktif
5. Konflik antar-dokumen (kalau ada)
```

**Hasil yang benar:** agent merangkum dengan benar. Tidak ada konflik.

**Gagal kalau:** agent bingung / dokumen kontradiksi.

## Cara Verifikasi
- [ ] Semua 5 file terisi penuh (tidak ada "TODO" / kosong)
- [ ] PRD punya "Out of Scope"
- [ ] DESIGN punya hex warna + font
- [ ] TASKS punya ≥10 task dengan definisi selesai
- [ ] Agent merangkum tanpa salah

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| Agent bikin stack beda | ARCHITECTURE belum dibaca | pastikan agent baca semua |
| Task terlalu besar | kurang pecah | 1 task = 1 endpoint / 1 halaman |
| Warna "biru" tanpa hex | DESIGN belum spesifik | beri hex code |

## Tugas Mandiri

Tukar dokumen dengan 1 teman kelas. Beri feedback:
- ada bagian kosong?
- ada konflik?
- task sudah cukup kecil?

## Bukti Kelulusan

Kirim ke mentor:
1. 5 file Markdown terisi,
2. screenshot agent merangkum dokumen tanpa salah,
3. feedback dari teman (kalau ada).
