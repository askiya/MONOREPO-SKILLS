# 02 — Struktur Monorepo

## Tujuan

Menentukan bentuk repo: satu aplikasi, atau beberapa paket dalam satu repo.

## Tiga Bentuk Umum

### A. Single App (paling disarankan untuk mulai)

```
project/
├── app/
├── components/
├── lib/
├── prisma/
└── package.json
```

Next.js menangani frontend dan API sekaligus. Cocok untuk 90% project kelas ini.

### B. Monorepo Sederhana (FE + BE terpisah)

```
project/
├── apps/
│   ├── web/          # Next.js frontend
│   └── api/          # Express/NestJS/FastAPI backend
├── packages/
│   └── shared/       # types & util dipakai bersama
├── package.json      # workspaces
└── README.md
```

`package.json` root:

```json
{
  "name": "project",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev:web": "npm run dev -w apps/web",
    "dev:api": "npm run dev -w apps/api",
    "build": "npm run build -w apps/web && npm run build -w apps/api"
  }
}
```

Pakai ini kalau backend bukan JavaScript, atau backend harus deploy terpisah.

### C. Monorepo dengan Tooling (Turborepo/Nx)

Baru perlu kalau: banyak app, banyak paket bersama, build lambat, dan tim > 1.
Jangan mulai dari sini.

## Cara Memilih

| Kondisi | Pilih |
|---|---|
| Satu web app, backend ringan | A |
| Backend Python/Go, frontend Next.js | B |
| Web + mobile + admin + banyak shared lib | C |

## Aturan Folder Apa Pun Bentuknya

- Nama folder huruf kecil, pakai tanda hubung.
- Satu folder = satu tanggung jawab.
- `lib/` untuk logika yang tidak berhubungan UI.
- `components/ui/` untuk primitif; `components/sections/` untuk bagian halaman.
- Jangan ada folder `misc/`, `temp/`, `new/`, `final2/`.

## Contoh Struktur Next.js Lengkap

```
app/
├── (public)/
│   ├── page.tsx              # landing
│   ├── produk/page.tsx
│   └── produk/[slug]/page.tsx
├── (auth)/
│   ├── login/page.tsx
│   └── register/page.tsx
├── (member)/
│   ├── layout.tsx            # cek session di sini
│   └── dashboard/page.tsx
├── (admin)/
│   ├── layout.tsx            # cek role admin
│   └── admin/products/page.tsx
├── api/
│   ├── auth/[...nextauth]/route.ts
│   ├── products/route.ts
│   └── webhook/payment/route.ts
├── layout.tsx
└── globals.css
```

## Checklist

- [ ] Bentuk repo dipilih (A/B/C) dan ditulis di ARCHITECTURE.md
- [ ] Struktur folder didokumentasikan
- [ ] Tidak ada folder tanpa tanggung jawab jelas
