# 03 — Menulis DESIGN.md (Design Token & Bahasa Visual)

## Tujuan

Semua halaman tampak satu produk, bukan kumpulan halaman acak. Agent punya
sumber kebenaran warna, font, spacing, dan komponen.

## Isi Minimum DESIGN.md

### 1. Warna

```markdown
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
| --color-success | #10B981 | sukses |
| --color-error | #EF4444 | error |
| --color-warning | #F59E0B | peringatan |
```

### 2. Tipografi

```markdown
## Tipografi

| Elemen | Font | Size | Weight |
|---|---|---|---|
| H1 | Inter | 2.25rem (36px) | 700 |
| H2 | Inter | 1.5rem (24px) | 600 |
| Body | Inter | 1rem (16px) | 400 |
| Small | Inter | 0.875rem (14px) | 400 |
| Button | Inter | 0.875rem | 500 |
```

### 3. Spacing & Radius

```markdown
## Spacing

Base: 4px. Gunakan kelipatan: 8, 12, 16, 24, 32, 48, 64.
Padding card: 24px. Gap grid: 16px. Margin antar-section: 48px.

## Radius

Button: 8px. Card: 12px. Input: 8px. Avatar: 9999px (bulat).
```

### 4. Komponen Kunci

Minimal definisikan visual:

- Button (primary, secondary, ghost, disabled, loading)
- Input (default, focus, error, disabled)
- Card
- Badge / Tag
- Modal / Dialog

Beri deskripsi cukup supaya agent bisa mengimplementasi tanpa screenshot,
atau tautkan ke referensi yang agent bisa akses.

### 5. Breakpoints

```markdown
## Responsive

| Nama | Min-width |
|---|---|
| mobile | 0 |
| tablet | 768px |
| desktop | 1024px |
| wide | 1280px |

Desain mobile-first. Grid: 1 kolom mobile, 2 tablet, 3–4 desktop.
```

## Cara Pakai

1. Copy `templates/DESIGN.md` ke project.
2. Ganti warna, font, spacing sesuai brand.
3. Saat build, selalu suruh agent:

```text
Baca DESIGN.md. Gunakan token dari sana. Jangan pakai warna, font, atau spacing
di luar yang terdefinisi tanpa minta izin.
```

## Kesalahan Umum

- DESIGN.md berisi terlalu banyak detail yang seharusnya di Figma, jadinya document rot.
- Tidak ada token — agent pakai arbitrary Tailwind value tiap halaman beda.
- Tidak ada komponen kunci — setiap form tampil beda.
- Dark mode disebut tapi token-nya tidak diisi.

## Checklist

- [ ] Minimal warna, tipografi, spacing, breakpoint terdefinisi
- [ ] Agent bisa membangun UI tanpa menebak warna/font
- [ ] Tidak ada warna hardcode di luar token yang disepakati
