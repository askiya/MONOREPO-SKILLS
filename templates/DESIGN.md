# DESIGN.md — Design Token & Bahasa Visual

## Warna

| Token | Hex | Peran |
|---|---|---|
| --color-primary | #... | tombol utama, link aktif |
| --color-primary-hover | #... | hover tombol utama |
| --color-bg | #FFFFFF | background body |
| --color-bg-card | #F9FAFB | background card |
| --color-text | #111827 | teks utama |
| --color-text-muted | #6B7280 | teks sekunder |
| --color-border | #E5E7EB | garis pemisah |
| --color-success | #10B981 | sukses |
| --color-error | #EF4444 | error |
| --color-warning | #F59E0B | peringatan |

## Tipografi

| Elemen | Font | Size | Weight |
|---|---|---|---|
| H1 | ... | 2.25rem | 700 |
| H2 | ... | 1.5rem | 600 |
| H3 | ... | 1.25rem | 600 |
| Body | ... | 1rem | 400 |
| Small | ... | 0.875rem | 400 |
| Button | ... | 0.875rem | 500 |

## Spacing

Base: 4px. Gunakan kelipatan: 8, 12, 16, 24, 32, 48, 64.
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

Desain mobile-first. Grid: 1 kolom mobile, 2 tablet, 3–4 desktop.

## Komponen Kunci

### Button
- Variant: primary, secondary, ghost, destructive
- State: default, hover, focus-visible, disabled, loading
- Loading: spinner + teks

### Input
- State: default, focus, error, disabled
- Error message di bawah input

### Card
- Shadow ringan, border, radius 12px
- Hover: shadow sedikit lebih besar (opsional)

### Badge / Tag
- Kecil, rounded full, background warna sesuai jenis

### Modal / Dialog
- Overlay gelap, card tengah, tutup Escape + tombol
- Fokus terjebak di dalam modal

## Aturan

- Jangan pakai warna di luar token ini tanpa izin.
- Jangan campur font berbeda tanpa alasan.
- State loading/empty/error wajib ada di setiap halaman data.
