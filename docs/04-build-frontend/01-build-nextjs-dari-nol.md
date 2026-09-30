# 01 — Build Frontend Next.js dari Nol

## Tujuan

Agent membuat fondasi frontend sesuai dokumen, bukan template acak.

## Prasyarat

- `PRD.md`, `DESIGN.md`, `ARCHITECTURE.md`, `TASKS.md` final
- Node.js LTS dan Git aktif
- Folder project sudah jadi repo Git

## Prompt Inisialisasi

```text
Baca semua file Markdown di root. Kerjakan hanya T-001: inisialisasi Next.js
App Router + TypeScript + Tailwind di folder saat ini. Jangan hapus dokumen.
Gunakan npm. Setelah selesai jalankan npm run lint dan npm run build.
Laporkan output nyata dan file yang berubah. Jangan commit.
```

Kalau folder sudah berisi dokumen, agent dapat menjalankan `create-next-app` di
folder sementara lalu memindahkan file aplikasi tanpa menimpa dokumen.

## Urutan Build

1. **Fondasi** — layout, font, CSS variables dari `DESIGN.md`.
2. **Primitif UI** — Button, Input, Card, Badge.
3. **Shell** — header, sidebar, footer, container.
4. **Halaman publik** — landing, katalog, detail.
5. **Auth UI** — login/register, belum harus terhubung backend.
6. **Member/Admin shell** — route dan layout terproteksi nanti.
7. **State** — loading, empty, error, success.
8. **Responsive** — uji 375px, 768px, 1280px.

Jangan build semua halaman dalam satu prompt.

## Prompt per Halaman

```text
Kerjakan T-021 saja: halaman katalog `/produk`.
Baca DESIGN.md dan komponen yang sudah ada. Pakai data mock lokal 6 item dulu.
Wajib: loading skeleton, empty state, error state, grid 1/2/3 kolom pada
375/768/1024px. Jangan tambah dependency. Jalankan lint dan build.
```

## Review Visual

Cek di browser:

- hierarchy judul jelas,
- kontras teks light mode terbaca,
- tombol aktif terlihat aktif,
- tidak ada overflow horizontal,
- kontrol punya hover/focus/disabled,
- elemen klik berfungsi atau dihapus,
- ukuran tap mobile minimal sekitar 44px.

## Checklist

- [ ] Fondasi mengikuti DESIGN.md
- [ ] Komponen reused, tidak diduplikasi per halaman
- [ ] State loading/empty/error ada
- [ ] Mobile 375px tidak terpotong
- [ ] `npm run lint` dan `npm run build` sukses
