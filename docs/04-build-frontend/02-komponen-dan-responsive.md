# 02 — Komponen, State, dan Responsive

## Tujuan

UI konsisten, dapat dipakai ulang, dan berfungsi di HP sampai desktop.

## Kapan Membuat Komponen

Buat komponen kalau:
- dipakai minimal dua tempat,
- punya state/perilaku sendiri,
- panjang halaman jadi sulit dibaca.

Jangan abstraksi satu elemen yang hanya dipakai sekali.

## Kontrak Komponen

Contoh Button:

```ts
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "ghost";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
}
```

Harus punya state: default, hover, focus-visible, disabled, loading.

## Empat State Wajib Data

1. **Loading** — skeleton atau spinner dengan label aksesibel.
2. **Empty** — pesan jelas + aksi berikutnya.
3. **Error** — pesan manusiawi + tombol coba lagi.
4. **Success** — data tampil; feedback setelah mutasi.

## Mobile-first

Mulai CSS dari layar kecil, tambah breakpoint:

```tsx
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
```

Jangan membuat desktop lalu "mengecilkan". Uji:

| Lebar | Target |
|---|---|
| 375px | HP kecil |
| 768px | tablet |
| 1024px | laptop |
| 1440px | desktop lebar |

## Aksesibilitas Minimum

- Satu `h1` per halaman; heading berurutan.
- Input punya `<label>`.
- Tombol icon punya `aria-label`.
- Fokus keyboard terlihat.
- Modal bisa ditutup Escape dan fokus tidak hilang.
- Gambar informatif punya `alt`; dekoratif pakai `alt=""`.
- Jangan bedakan status hanya dari warna.

## Prompt Review Agent

```text
Audit halaman yang baru dibuat terhadap DESIGN.md, responsive 375/768/1280,
dan aksesibilitas keyboard. Perbaiki hanya masalah yang terbukti. Jangan ubah
layout besar. Jalankan lint dan build setelahnya.
```

## Checklist

- [ ] Tidak ada komponen duplikat
- [ ] Empat state data ada
- [ ] Uji empat viewport
- [ ] Keyboard dan label berfungsi
- [ ] Tidak ada kontrol mati/placeholder palsu
