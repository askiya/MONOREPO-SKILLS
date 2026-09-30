# 03 — Build: Website Statis (Astro atau Vite)

## Tujuan

Website statis jalan di localhost, responsive, sesuai `DESIGN.md`, dan
menghasilkan folder `dist/` saat build. Tanpa backend, database, dan secret.

## Sebelum Mulai

- Agent sudah punya konteks ([`02-ai-agent.md`](02-ai-agent.md))
- Node.js LTS terinstal (`node -v`)

Referensi komponen dan responsive:
[`../../docs/04-build-frontend/02-komponen-dan-responsive.md`](../../docs/04-build-frontend/02-komponen-dan-responsive.md)

---

## Langkah 1 — Scaffold project

### Opsi A — Astro (disarankan)

```bash
npm create astro@latest .
```

Pilihan saat wizard:

| Pertanyaan | Jawab |
|---|---|
| Template | Empty |
| TypeScript | Yes → Strict |
| Install dependencies | Yes |
| Initialize git | No (sudah `git init`) |

Tambahkan Tailwind:

```bash
npx astro add tailwind
```

Jawab `y` saat konfirmasi.

### Opsi B — Vite + React

```bash
npm create vite@latest . -- --template react-ts
npm install
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Isi `content` di `tailwind.config.js`:

```js
content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
```

Tambahkan di CSS utama:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

> Pilih satu opsi saja. Jangan pasang Astro dan Vite dalam satu project.

---

## Langkah 2 — Verifikasi scaffold

```bash
npm run dev
```

**Expected output (Astro):**
```
astro  v4.x.x ready in xxx ms
┃ Local    http://localhost:4321/
```

**Expected output (Vite):**
```
VITE v5.x.x  ready in xxx ms
➜  Local:   http://localhost:5173/
```

Buka URL tersebut. Halaman default harus tampil tanpa error di console browser.
Hentikan server dengan `Ctrl+C`.

---

## Langkah 3 — Bangun layout dasar

Prompt:

```text
Buat layout dasar sesuai DESIGN.md:

1. Layout utama: header, main, footer
2. Header: logo/nama, menu navigasi, menu mobile yang bisa dibuka-tutup
3. Footer: copyright dan link media sosial dari PRD.md
4. Terapkan warna, font, spacing, dan lebar maksimum dari DESIGN.md
5. Mobile-first: breakpoint sm, md, lg

Aturan:
- Tanpa backend, API privat, database, dan secret
- Hanya dependency yang sudah terpasang
- Gunakan elemen semantik: header, nav, main, footer
- Link aktif harus punya focus state yang terlihat

Setelah selesai jalankan npm run build dan laporkan file yang diubah.
```

**Expected output:** file layout + komponen header/footer, `npm run build` sukses.

---

## Langkah 4 — Buat halaman satu per satu

Kerjakan per halaman. Jangan minta semua halaman dalam satu prompt.

```text
Buat halaman [NAMA HALAMAN] sesuai PRD.md dan DESIGN.md.

Wajib:
- Pakai layout utama yang sudah ada
- Urutan section sesuai DESIGN.md
- Heading hierarki benar: satu h1 per halaman
- Semua gambar punya alt deskriptif
- CTA utama terlihat tanpa scroll panjang di mobile
- Responsive 375px, 768px, 1440px
- Tanpa backend, database, dan secret

Setelah selesai jalankan npm run build. Laporkan file yang diubah.
```

Setelah setiap halaman selesai:

```bash
npm run build
git add .
git commit -m "feat: halaman [nama]"
```

Jangan lanjut ke halaman berikutnya jika build gagal.

---

## Langkah 5 — Form kontak tanpa backend

Website statis tidak bisa memproses form sendiri. Pilihan yang benar:

| Cara | Keterangan |
|---|---|
| Link WhatsApp | `https://wa.me/62XXXXXXXXXX` — paling sederhana |
| `mailto:` | Membuka aplikasi email pengunjung |
| Form provider pihak ketiga | Mengirim ke endpoint publik milik provider |

Prompt:

```text
Buat bagian kontak tanpa backend. Gunakan link WhatsApp dan mailto dari PRD.md.
Jangan membuat API route, jangan menyimpan data, jangan menaruh secret di kode.
```

> Jika form harus menyimpan data ke database, project ini bukan Paket B.
> Pindah ke Paket A.

---

## Langkah 6 — Aset dan performa

Prompt:

```text
Optimalkan aset statis:
1. Gambar dipakai dengan ukuran sesuai tampilan, bukan file mentah besar
2. Tambahkan width dan height untuk mencegah layout shift
3. Gambar di bawah viewport pakai loading="lazy"
4. Maksimal dua font family, subset latin
5. Hapus CSS dan dependency yang tidak dipakai

Jangan menambah dependency baru. Setelah selesai jalankan npm run build.
```

---

## Langkah 7 — Verifikasi output build

```bash
npm run build
ls dist
```

**Expected output:** folder `dist/` berisi `index.html`, folder aset, dan
halaman lain. Nama folder output harus dicatat untuk fase deploy.

| Stack | Output |
|---|---|
| Astro | `dist` |
| Vite | `dist` |

---

## Kesalahan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Class Tailwind tidak berlaku | `content` di config salah | Perbaiki glob path ke file project |
| Horizontal scroll di mobile | Elemen lebar tetap | Ganti ke `max-width: 100%` / class responsive |
| Font berubah setelah build | Font hanya diimpor lokal | Impor font lewat cara yang ikut ter-build |
| Link antar halaman 404 | Path salah atau file bukan di folder halaman | Gunakan path root absolut yang benar |
| Build sukses tapi halaman kosong | Komponen tidak dirender di halaman | Pastikan halaman memakai layout dan komponen |
| Agent menambah dependency berat | Scope tidak dibatasi | Minta hapus dan pakai HTML/CSS native |

---

## Checklist

- [ ] Satu stack dipilih: Astro atau Vite
- [ ] Tailwind aktif dan class-nya berlaku
- [ ] `npm run dev` jalan tanpa error console
- [ ] Layout header/main/footer selesai
- [ ] Menu mobile bisa dibuka dan ditutup
- [ ] Semua halaman dari PRD selesai
- [ ] Satu `h1` per halaman dan hierarki heading benar
- [ ] Semua gambar punya `alt`, `width`, dan `height`
- [ ] Kontak memakai WhatsApp/`mailto`/provider publik, bukan backend sendiri
- [ ] Tidak ada API key atau secret di kode
- [ ] Responsive pada 375px, 768px, 1440px
- [ ] `npm run build` sukses dan folder `dist/` berisi `index.html`
- [ ] Semua perubahan di-commit
