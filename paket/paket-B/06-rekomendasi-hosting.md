# 06 — Rekomendasi Hosting Paket B

> **Paket B** = Cloudflare Static 🟢 PEMULA
> **Stack**: Astro/Vite statis, tanpa backend

> ⚠️ Harga dan fitur berubah sewaktu-waktu. Selalu cek situs resmi sebelum membeli.

## Static Hosting

Karena hasil build Astro/Vite berupa file statis, paket ini tidak memerlukan server aplikasi, PHP, atau database. Hubungkan repositori Git, isi perintah build, lalu arahkan folder hasil build sebagai direktori publik.

### Rekomendasi utama: Cloudflare Pages

Cloudflare Pages cocok untuk situs statis karena paket gratis menyediakan:

- 500 build per bulan.
- Maksimal 20.000 file per situs.
- Bandwidth tidak terbatas.
- CDN global dan HTTPS otomatis.
- Custom domain.
- Preview deployment untuk branch dan pull request.

Pengaturan umum:

- **Build command Astro**: `npm run build`
- **Build command Vite**: `npm run build`
- **Output directory Astro**: `dist`
- **Output directory Vite**: `dist`

### Alternatif

#### GitHub Pages

Gratis untuk situs statis dengan batas umum 1 GB per situs, bandwidth lunak 100 GB per bulan, dan 10 build per jam. Cocok untuk dokumentasi, portofolio, landing page, atau situs proyek publik. Tidak menyediakan backend dinamis.

#### Netlify

Paket gratis memakai 300 credit per bulan. Cocok jika membutuhkan form handling, preview deployment, redirect, dan workflow yang mudah. Pemakaian build, bandwidth, atau fitur lain dapat menghabiskan credit.

#### Vercel

Paket Hobby gratis menyediakan hingga 100 deploy per hari. Cocok untuk Astro/Vite dan proses deploy dari Git. Paket Hobby ditujukan untuk penggunaan pribadi dan non-komersial; periksa ketentuan terbaru sebelum memakai untuk bisnis.

#### Surge.sh

Gratis untuk proyek kecil dan mudah dipakai dari command line. Cocok untuk demo atau prototipe cepat. Fitur dashboard, kolaborasi, dan preview tidak selengkap provider utama.

## Domain & DNS

### Cloudflare DNS

Cloudflare DNS tersedia gratis dan dapat dipakai meskipun domain dibeli dari registrar lain. Fitur utama:

- DNS global cepat.
- Proxy dan CDN Cloudflare.
- SSL/TLS.
- Perlindungan DDoS dasar.
- Pengaturan cache dan redirect.

### Domain murah Indonesia

Domain `.my.id` biasanya tersedia sekitar **Rp 10.000–15.000 per tahun** dari registrar Indonesia. Harga promo dan harga perpanjangan dapat berbeda. Pilihan lain seperti `.web.id` atau `.biz.id` umumnya tetap murah, tetapi persyaratan dan harga harus diperiksa di registrar.

Langkah umum memasang domain:

1. Beli domain dari registrar.
2. Tambahkan domain ke Cloudflare.
3. Ganti nameserver domain sesuai instruksi Cloudflare.
4. Tambahkan custom domain di provider hosting.
5. Periksa DNS dan tunggu propagasi.
6. Pastikan HTTPS aktif.

## Tabel Perbandingan

| Provider | Biaya | Batas Utama | Cocok untuk | Link resmi |
|---|---:|---|---|---|
| Cloudflare Pages | Gratis | 500 build/bulan, 20.000 file, bandwidth tidak terbatas | Pilihan utama situs statis | [pages.cloudflare.com](https://pages.cloudflare.com/) |
| GitHub Pages | Gratis | 1 GB/situs, 100 GB bandwidth/bulan, 10 build/jam | Dokumentasi, portofolio, proyek GitHub | [pages.github.com](https://pages.github.com/) |
| Netlify | Gratis, 300 credit/bulan | Pemakaian dihitung dengan credit | Situs statis dengan form handling | [netlify.com](https://www.netlify.com/) |
| Vercel Hobby | Gratis | 100 deploy/hari, ketentuan non-komersial | Astro/Vite pribadi dan preview | [vercel.com](https://vercel.com/) |
| Surge.sh | Gratis untuk proyek kecil | Fitur lebih sederhana | Demo dan prototipe cepat | [surge.sh](https://surge.sh/) |

## Peringatan

- Harga, kuota, dan ketentuan paket gratis dapat berubah tanpa mengikuti dokumen ini.
- Periksa batas build, file, bandwidth, dan kebijakan penggunaan wajar di situs resmi.
- Periksa harga perpanjangan domain, bukan hanya harga promo tahun pertama.
- Jangan memilih GitHub Pages bila aplikasi memerlukan SSR, database, atau API rahasia.
- Jangan menyimpan secret atau token di kode frontend. Semua file hasil build dapat dilihat pengguna.
- Simpan salinan konfigurasi DNS sebelum mengubah nameserver.

## Link Resmi

- [Cloudflare Pages](https://pages.cloudflare.com/)
- [Dokumentasi batas Cloudflare Pages](https://developers.cloudflare.com/pages/platform/limits/)
- [GitHub Pages](https://pages.github.com/)
- [Dokumentasi batas GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)
- [Netlify](https://www.netlify.com/)
- [Harga Netlify](https://www.netlify.com/pricing/)
- [Vercel](https://vercel.com/)
- [Harga Vercel](https://vercel.com/pricing)
- [Surge.sh](https://surge.sh/)
- [Cloudflare DNS](https://www.cloudflare.com/dns/)

## Checklist

### Kebutuhan situs

- [ ] Situs benar-benar statis dan tidak memerlukan backend.
- [ ] Hasil build berada di folder `dist`.
- [ ] Tidak ada secret, password, atau token privat di frontend.
- [ ] Batas build dan file provider cukup untuk proyek.

### Deploy

- [ ] Repositori Git sudah terhubung.
- [ ] Build command sudah diisi `npm run build`.
- [ ] Output directory sudah diisi `dist`.
- [ ] Deploy produksi berhasil tanpa error.
- [ ] Preview branch berhasil bila dibutuhkan.

### Domain

- [ ] Harga pembelian dan perpanjangan domain sudah diperiksa.
- [ ] Nameserver atau record DNS sudah benar.
- [ ] Custom domain sudah terpasang.
- [ ] HTTPS aktif dan tidak ada mixed content.
- [ ] Domain utama dan versi `www` diarahkan secara konsisten.

### Operasional

- [ ] Halaman 404 tersedia.
- [ ] Redirect lama sudah diuji.
- [ ] Situs diuji dari perangkat seluler.
- [ ] Monitoring uptime dipasang bila situs penting.
