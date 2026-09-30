# 04 — Preview, Browser Check, dan Lighthouse

## Tujuan

Website terbukti benar sebelum deploy: jalan di localhost, tampil rapi di tiga
ukuran layar, semua link bekerja, build produksi sukses, dan skor Lighthouse
minimal 90 di empat kategori.

## Sebelum Mulai

- Build fase 3 selesai ([`03-build.md`](03-build.md))

Referensi:
- [`../../docs/07-preview-localhost/01-localhost-dan-debug.md`](../../docs/07-preview-localhost/01-localhost-dan-debug.md)
- [`../../docs/08-testing/01-testing-sebelum-deploy.md`](../../docs/08-testing/01-testing-sebelum-deploy.md)
- [`../../docs/19-seo-performance-legal/01-seo-performance.md`](../../docs/19-seo-performance-legal/01-seo-performance.md)

---

## Langkah 1 — Preview development

```bash
npm run dev
```

Astro membuka `http://localhost:4321`, Vite `http://localhost:5173`.

Jika port terpakai:

```bash
npm run dev -- --port 4400
```

---

## Langkah 2 — Preview hasil build

Ini versi yang benar-benar akan di-deploy.

```bash
npm run build
npm run preview
```

**Expected output:** build sukses dan server preview menampilkan URL lokal.
Uji halaman dari URL preview ini, bukan hanya dari `npm run dev`.

---

## Langkah 3 — Browser check

Buka DevTools dengan `F12`.

| Tab | Yang diperiksa | Lolos kalau |
|---|---|---|
| Console | Error dan warning | Tidak ada error merah |
| Network | Status request | Tidak ada 404 pada CSS, JS, gambar, font |
| Elements | Struktur semantik | Ada `header`, `nav`, `main`, `footer`, satu `h1` |

### Uji navigasi

- Semua menu membuka halaman yang benar
- Semua link internal tidak 404
- Semua link eksternal membuka tujuan yang benar
- CTA utama bekerja (WhatsApp/email terbuka)
- Refresh di halaman non-beranda tetap tampil

### Uji keyboard

- Tekan `Tab` berulang: fokus berpindah dengan urutan logis
- Elemen yang difokus punya indikator terlihat
- Menu mobile bisa dibuka dan ditutup lewat keyboard

---

## Langkah 4 — Responsive check

DevTools → device toolbar (`Ctrl+Shift+M`).

| Lebar | Yang diperiksa |
|---|---|
| 375px | Tanpa horizontal scroll, teks terbaca, target sentuh cukup besar |
| 768px | Layout berpindah rapi, tidak ada elemen tumpang tindih |
| 1440px | Konten punya lebar maksimum, tidak melebar berlebihan |

**Bukti kelulusan:** screenshot 375px, 768px, dan 1440px.

Uji juga dari HP nyata setelah deploy ([`05-deploy.md`](05-deploy.md)).

---

## Langkah 5 — Lighthouse

Jalankan pada hasil build, bukan development server.

### Cara 1 — DevTools

1. Jalankan `npm run build` lalu `npm run preview`
2. Buka URL preview di Chrome
3. DevTools → tab **Lighthouse**
4. Mode: **Navigation**, Device: **Mobile**
5. Centang Performance, Accessibility, Best Practices, SEO
6. **Analyze page load**

### Cara 2 — CLI

```bash
npx lighthouse http://localhost:4321 --view --preset=desktop
```

Ganti port sesuai URL preview.

### Target minimal

| Kategori | Target |
|---|---|
| Performance | ≥ 90 |
| Accessibility | ≥ 90 |
| Best Practices | ≥ 90 |
| SEO | ≥ 90 |

Website statis sederhana wajar mendapat skor tinggi. Skor rendah berarti ada
aset berat, markup salah, atau metadata kurang.

---

## Langkah 6 — Perbaiki temuan Lighthouse

| Temuan | Penyebab umum | Perbaikan |
|---|---|---|
| Largest Contentful Paint lambat | Gambar hero terlalu besar | Kompres, ukuran sesuai tampilan |
| Cumulative Layout Shift tinggi | Gambar tanpa dimensi | Tambahkan `width` dan `height` |
| Contrast ratio kurang | Warna teks terlalu terang | Perbaiki nilai warna di `DESIGN.md` |
| Image elements missing alt | Gambar tanpa `alt` | Tambahkan `alt` deskriptif |
| Document has no meta description | Metadata belum diisi | Tambahkan `title` dan `meta description` per halaman |
| Links not crawlable | Navigasi bukan `<a href>` | Ganti ke anchor dengan `href` |

Prompt perbaikan:

```text
Skor Lighthouse saya:
Performance [x], Accessibility [x], Best Practices [x], SEO [x]

Temuan utama:
[TEMPEL DAFTAR TEMUAN LIGHTHOUSE]

Perbaiki penyebabnya di kode, bukan dengan menyembunyikan elemen.
Jangan menambah dependency baru.
Setelah selesai jalankan npm run build dan laporkan file yang diubah.
```

---

## Langkah 7 — Metadata dan SEO dasar

Prompt:

```text
Tambahkan metadata per halaman sesuai PRD.md:
1. title unik per halaman
2. meta description unik
3. Open Graph: og:title, og:description, og:image, og:url
4. lang="id" pada elemen html
5. favicon
6. robots.txt dan sitemap sederhana

Jangan menambah script analytics atau layanan yang butuh secret.
```

Verifikasi setelah build:

```bash
npm run build
ls dist
```

Pastikan `dist/` berisi `index.html`, `robots.txt`, dan `sitemap` jika dibuat.

---

## Kesalahan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Lighthouse jauh lebih rendah dari perkiraan | Diuji di `npm run dev` | Uji pada `npm run preview` |
| Gambar 404 setelah build | Path aset salah | Letakkan aset di folder publik dan rujuk path root |
| Halaman bagus di desktop, rusak di HP | Tidak diuji 375px | Uji ulang mobile-first |
| Skor Accessibility rendah | Kontras dan `alt` kurang | Perbaiki warna dan teks alternatif |
| Console error `undefined` | Script mengakses elemen yang belum ada | Jalankan script setelah DOM siap |

Decision tree error: [`../../docs/15-troubleshooting/02-decision-tree.md`](../../docs/15-troubleshooting/02-decision-tree.md)
Format lapor bug: [`../../BANTUAN.md`](../../BANTUAN.md)

---

## Checklist

- [ ] `npm run dev` jalan tanpa error
- [ ] `npm run build` sukses
- [ ] `npm run preview` jalan dan dipakai untuk pengujian
- [ ] Console browser tanpa error merah
- [ ] Network tanpa 404 aset
- [ ] Semua link internal dan eksternal benar
- [ ] CTA utama bekerja
- [ ] Refresh di halaman non-beranda tetap tampil
- [ ] Navigasi keyboard dan focus state terlihat
- [ ] Responsive 375px, 768px, 1440px
- [ ] Screenshot tiga ukuran disimpan
- [ ] Lighthouse Performance ≥ 90
- [ ] Lighthouse Accessibility ≥ 90
- [ ] Lighthouse Best Practices ≥ 90
- [ ] Lighthouse SEO ≥ 90
- [ ] Metadata, favicon, dan `lang="id"` terpasang
- [ ] Semua perubahan di-commit
