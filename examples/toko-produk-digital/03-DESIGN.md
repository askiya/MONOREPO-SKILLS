# DESIGN.md — SantriLearn

## Warna

| Token | Hex | Peran |
|---|---|---|
| --color-primary | #2563EB | tombol utama, link aktif |
| --color-primary-hover | #1D4ED8 | hover tombol utama |
| --color-bg | #FFFFFF | background body |
| --color-bg-card | #F9FAFB | background card |
| --color-text | #111827 | teks utama |
| --color-text-muted | #6B7280 | teks sekunder |
| --color-border | #E5E7EB | garis pemisah |
| --color-success | #10B981 | sukses / badge aktif |
| --color-error | #EF4444 | error / badge gagal |
| --color-warning | #F59E0B | peringatan |

## Tipografi

| Elemen | Font | Size | Weight |
|---|---|---|---|
| H1 | Inter | 2.25rem (36px) | 700 |
| H2 | Inter | 1.5rem (24px) | 600 |
| H3 | Inter | 1.25rem (20px) | 600 |
| Body | Inter | 1rem (16px) | 400 |
| Small | Inter | 0.875rem (14px) | 400 |
| Button | Inter | 0.875rem | 500 |

## Spacing

Base: 4px. Kelipatan: 8, 12, 16, 24, 32, 48, 64.
- Padding card: 24px
- Gap grid: 16px
- Margin antar-section: 48px

## Radius

| Elemen | Radius |
|---|---|
| Button | 8px |
| Card | 12px |
| Input | 8px |
| Avatar | 9999px |
| Modal | 16px |

## Responsive Breakpoints

| Nama | Min-width |
|---|---|
| mobile | 0 |
| tablet | 768px |
| desktop | 1024px |
| wide | 1280px |

Grid produk: 1 kolom mobile, 2 tablet, 3 desktop.

## Komponen

### Button
- primary (biru), secondary (outline), ghost, destructive (merah)
- State: default, hover, focus-visible, disabled, loading (spinner+text)

### Input
- State: default, focus (ring biru), error (ring merah + pesan bawah), disabled

### Card Produk
- Gambar atas, judul, harga, badge tipe, tombol "Lihat Detail"
- Hover: shadow sedikit naik

### Badge
- Rounded full, ukuran kecil
- Varian: e-book (biru muda), video (ungu muda), template (hijau muda)

### Modal
- Overlay gelap (#00000080), card putih tengah, tombol tutup + Escape
- Fokus terjebak di dalam modal
