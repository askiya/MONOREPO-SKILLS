# 01 — Pedoman Ringkas untuk Website Statis

## Tujuan

Menyiapkan dua dokumen yang cukup untuk project statis: `PRD.md` ringan dan
`DESIGN.md`. Paket B tidak memerlukan desain API, database, atau auth.

## Kenapa Hanya Dua Dokumen?

Website statis mengirim HTML, CSS, JavaScript, gambar, dan font ke browser.
Tidak ada server aplikasi dan database. Dokumentasi harus sesuai skala project,
bukan dibuat ramai tanpa manfaat.

| Dokumen | Status | Alasan |
|---|---|---|
| `PRD.md` | Wajib, versi ringan | Menetapkan audiens, halaman, CTA, dan batasan |
| `DESIGN.md` | Wajib | Menetapkan visual, layout, komponen, responsive |
| `SDLC.md` | Tidak wajib | Alur Git sederhana cukup dicatat di PRD |
| `ARCHITECTURE.md` | Tidak wajib | Tidak ada API/database; stack sudah ditentukan |
| `TASKS.md` | Opsional | Checklist di PRD cukup untuk site kecil |
| `AGENTS.md` | Opsional | Tambahkan hanya jika aturan agent mulai banyak |

Referensi bentuk lengkap:
- [`../../docs/02-dokumen-perencanaan/01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md)
- [`../../docs/02-dokumen-perencanaan/03-design-md.md`](../../docs/02-dokumen-perencanaan/03-design-md.md)

---

## Langkah 1 — Buat folder project

```bash
mkdir website-statis
cd website-statis
git init
```

**Expected output:** terminal berpindah ke folder `website-statis` dan Git
menampilkan pesan repository kosong berhasil dibuat.

---

## Langkah 2 — Copy template

Copy [`../../templates/PRD.md`](../../templates/PRD.md) dan
[`../../templates/DESIGN.md`](../../templates/DESIGN.md) ke root project.

Kalau repository pedoman berada satu folder di atas project:

```bash
cp ../pedoman-monorepo/templates/PRD.md .
cp ../pedoman-monorepo/templates/DESIGN.md .
```

Jika path lokal berbeda, copy lewat file manager. Jangan edit template asli.

---

## Langkah 3 — Sederhanakan PRD

Hapus bagian yang tidak relevan. Isi minimal:

```markdown
# PRD — Nama Website

## Tujuan
[Satu kalimat: hasil bisnis/komunikasi yang ingin dicapai]

## Pengguna
[Siapa pengunjung utama]

## Halaman
- Beranda
- Tentang
- Proyek/Layanan
- Kontak

## CTA Utama
[Contoh: Hubungi lewat WhatsApp]

## Konten
- [ ] Logo
- [ ] Foto
- [ ] Teks profil
- [ ] Daftar proyek/layanan
- [ ] Link media sosial

## Batasan
- Tidak ada login
- Tidak ada database
- Tidak ada backend/API privat
- Tidak menyimpan secret di frontend

## Kriteria Selesai
- Responsive pada 375px, 768px, 1440px
- Semua link bekerja
- Lighthouse semua kategori minimal 90
```

Lihat contoh keputusan yang sudah terisi di
[`../../examples/toko-produk-digital/01-PRD.md`](../../examples/toko-produk-digital/01-PRD.md),
tetapi jangan copy fitur login/database dari contoh tersebut.

---

## Langkah 4 — Isi DESIGN.md

Gunakan template [`../../templates/DESIGN.md`](../../templates/DESIGN.md).
Isi minimal:

- Warna primer, sekunder, latar, teks, dan status focus
- Font heading dan body
- Lebar maksimum konten
- Spacing section
- Bentuk tombol dan kartu
- Navigasi mobile
- Urutan section setiap halaman
- State hover, focus, dan active
- Aturan gambar (`alt`, rasio, ukuran)

Prompt untuk membantu memilih desain:

```text
Baca PRD.md. Usulkan isi DESIGN.md untuk website statis ini.
Gunakan maksimal 2 font dan 5 warna. Pastikan kontras WCAG AA,
mobile-first, focus state terlihat, dan semua keputusan spesifik.
Jangan menambah halaman atau fitur di luar PRD.
```

**Expected output:** agent mengusulkan nilai konkret seperti kode warna,
ukuran, layout, dan komponen; bukan kata umum seperti "modern" saja.

---

## Langkah 5 — Pilih stack dan catat di PRD

Tambahkan bagian:

```markdown
## Stack
- Framework: Astro
- Styling: Tailwind CSS
- Deploy: Cloudflare Pages
- Build output: dist/
```

Ganti `Astro` menjadi `Vite + React` hanya jika landing page butuh banyak
interaksi client-side.

---

## Kesalahan Umum

| Kesalahan | Akibat | Perbaikan |
|---|---|---|
| Menambah login ke site statis | Membutuhkan backend/auth | Pindah ke Paket A |
| Menaruh API key di JavaScript | Secret terbaca semua pengunjung | Hapus; gunakan layanan tanpa secret atau pindah paket |
| PRD tidak punya CTA | Website bagus tapi tidak punya tujuan | Tetapkan satu CTA utama |
| DESIGN.md hanya bilang "minimalis" | Agent menebak semua detail | Isi token warna, font, spacing, layout |
| Copy contoh fullstack mentah | Scope membesar tanpa kebutuhan | Ambil format, bukan fiturnya |

---

## Checklist

- [ ] Folder project dibuat dan `git init` selesai
- [ ] `PRD.md` versi ringan terisi
- [ ] Audiens dan satu CTA utama jelas
- [ ] Daftar halaman dan konten jelas
- [ ] Batasan: tanpa login, database, backend, dan secret tertulis
- [ ] `DESIGN.md` berisi nilai warna, font, spacing, layout, responsive
- [ ] Stack Astro atau Vite dipilih satu
- [ ] Build output `dist/` tercatat
