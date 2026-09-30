# 01 — SEO dan Performance Dasar

## SEO Minimum

Setiap halaman publik perlu:
- `<title>` unik (50–60 karakter),
- meta description unik (±150 karakter),
- satu `<h1>` yang menjelaskan halaman,
- URL deskriptif (`/produk/kelas-nextjs`, bukan `/p?id=123`),
- gambar punya `alt`,
- canonical URL,
- Open Graph image (1200×630) untuk share sosial.

Next.js App Router:
```ts
export const metadata = {
  title: "Judul Halaman | Nama Brand",
  description: "Deskripsi singkat halaman.",
};
```

## sitemap.xml dan robots.txt

Next.js mendukung `app/sitemap.ts` dan `app/robots.ts`. Pastikan:
- URL produksi benar,
- halaman admin/dashboard tidak masuk sitemap,
- staging di-`noindex` supaya tidak muncul Google.

Staging header:
```text
X-Robots-Tag: noindex, nofollow
```

## Performance

Target awal Core Web Vitals:
- LCP < 2.5 detik,
- INP < 200 ms,
- CLS < 0.1.

Tindakan paling berdampak:
1. Gunakan `next/image`, isi `width`+`height` agar layout tidak loncat.
2. Kompres gambar, WebP/AVIF.
3. Jangan kirim JS besar untuk konten statis; pakai Server Component.
4. Lazy-load gambar/komponen di bawah fold.
5. Font lokal/`next/font`, batasi weight.
6. Jangan install library besar untuk satu fungsi kecil.

## Lighthouse

Chrome DevTools → Lighthouse → pilih Mobile → Analyze.

Simpan skor sebelum/sesudah. Jangan mengejar skor 100 dengan merusak UX.
Prioritas:
1. accessibility/security,
2. error nyata,
3. Core Web Vitals,
4. baru skor kosmetik.

## Checklist

- [ ] Title + description unik per halaman
- [ ] Satu H1 per halaman
- [ ] Alt text gambar bermakna
- [ ] sitemap.xml + robots.txt
- [ ] Staging noindex
- [ ] Open Graph image
- [ ] Lighthouse Mobile dijalankan
- [ ] Tidak ada gambar besar tanpa optimasi
