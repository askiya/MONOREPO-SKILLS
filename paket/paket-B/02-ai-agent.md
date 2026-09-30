# 02 — Setup AI Agent: Antigravity

## Tujuan

Antigravity membuka folder project dan memahami dua dokumen konteks: `PRD.md`
dan `DESIGN.md`. Konteks Paket B sengaja ringkas karena tidak ada backend.

## Sebelum Mulai

- `PRD.md` dan `DESIGN.md` selesai ([`01-pedoman.md`](01-pedoman.md))
- Node.js LTS dan Git terinstal

Panduan instalasi utama:
[`../../docs/01-setup-ai-agent/01-antigravity.md`](../../docs/01-setup-ai-agent/01-antigravity.md)

---

## Langkah 1 — Instal Antigravity

1. Unduh installer hanya dari sumber resmi Antigravity.
2. Instal sesuai OS.
3. Login atau hubungkan provider model yang didukung.
4. Jangan masukkan API key ke file source code.

> UI dan nama menu dapat berubah. Dokumentasi resmi Antigravity menjadi sumber
> terbaru; panduan repo ini menjelaskan alur, bukan menjamin posisi tombol.

---

## Langkah 2 — Buka folder project

1. Pilih **Open Folder**.
2. Pilih folder `website-statis` yang berisi `PRD.md` dan `DESIGN.md`.
3. Jangan membuka seluruh drive atau folder induk berisi banyak project.

**Expected output:** sidebar Antigravity menampilkan dua dokumen tersebut.

---

## Langkah 3 — Beri konteks ringkas

Copy prompt:

```text
Baca PRD.md dan DESIGN.md sebagai sumber kebenaran project ini.

Project ini website statis Paket B:
- Tidak ada backend
- Tidak ada API privat
- Tidak ada database
- Tidak ada login
- Tidak boleh ada secret/API key di kode browser
- Deploy ke Cloudflare Pages, output harus dist/

Setelah membaca, jawab:
1. Tujuan website
2. Audiens utama
3. Halaman yang harus dibuat
4. CTA utama
5. Stack yang dipilih
6. Lima keputusan visual dari DESIGN.md

Jangan mulai coding sebelum saya konfirmasi.
```

**Expected output:** jawaban sesuai dokumen, tanpa menambah login, database,
dashboard admin, atau fitur yang tidak diminta.

Kalau jawaban salah, jangan lanjut. Perbaiki dokumen atau tunjukkan bagian yang
terlewat lalu minta agent membaca ulang.

Panduan konteks lebih lengkap:
[`../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md`](../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md)

---

## Langkah 4 — Uji operasi file

Copy prompt:

```text
Buat README.md singkat berdasarkan PRD.md.
Isi: nama website, tujuan, stack, cara menjalankan lokal,
dan perintah build. Jangan membuat kode aplikasi dulu.
```

**Expected output:** `README.md` muncul di root, isinya sesuai PRD dan stack.

---

## Aturan Prompt Selama Build

Setiap prompt harus menyebut:

1. Task tunggal yang dikerjakan
2. Dokumen sumber (`PRD.md`, `DESIGN.md`)
3. Larangan backend/database/secret
4. Perintah verifikasi (`npm run build`)
5. Daftar file yang harus dilaporkan agent

Contoh:

```text
Kerjakan hanya navigasi dan hero berdasarkan PRD.md dan DESIGN.md.
Mobile-first dan aksesibel. Tidak ada backend atau dependency tambahan.
Setelah selesai jalankan npm run build. Laporkan file yang diubah.
```

---

## Kesalahan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Agent menambah database | Konteks Paket B tidak ditegaskan | Kirim ulang prompt konteks |
| Agent memilih Next.js | Stack tidak tercatat di PRD | Tulis Astro atau Vite secara eksplisit |
| Agent membuat semua halaman sekaligus | Prompt terlalu besar | Satu section/halaman per prompt |
| Agent install banyak package | Scope tidak dibatasi | Minta pakai HTML/CSS/native dulu |
| Agent menaruh token di `.env` client | Site statis tidak bisa menyimpan secret | Hapus token; jangan gunakan API privat |

---

## Checklist

- [ ] Antigravity terinstal dari sumber resmi
- [ ] Provider model terhubung
- [ ] Hanya folder project yang dibuka
- [ ] Agent membaca `PRD.md` dan `DESIGN.md`
- [ ] Agent menyebut tujuan, audiens, halaman, CTA, stack, dan desain dengan benar
- [ ] Agent memahami larangan backend, database, login, dan secret
- [ ] Tes membuat `README.md` berhasil
