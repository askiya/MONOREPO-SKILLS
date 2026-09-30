# 01 — Memilih Tech Stack

## Tujuan

Memutuskan stack sekali, tulis di `ARCHITECTURE.md`, lalu berhenti berdebat.

## Aturan Pemilihan

1. Pilih yang **kamu bisa debug**, bukan yang paling keren.
2. Pilih yang **dokumentasinya banyak** — AI agent lebih jago di stack populer.
3. Pilih yang **deploy-nya murah/gratis** untuk tahap awal.
4. Jangan tambah teknologi kalau yang ada sudah cukup.

## Rekomendasi Default Kelas Ini

| Layer | Pilihan | Kenapa |
|---|---|---|
| Framework | **Next.js (App Router) + TypeScript** | frontend+backend satu repo, SEO, deploy gratis Vercel |
| Styling | **Tailwind CSS** | cepat, konsisten dengan DESIGN.md |
| Komponen | **shadcn/ui** | copy ke repo, bisa diubah bebas |
| Database | **PostgreSQL** (Neon/Supabase) | relasional, tier gratis, standar industri |
| ORM | **Prisma** | migrasi jelas, type-safe |
| Auth | **NextAuth.js** atau JWT sendiri | sesuai kebutuhan OAuth |
| Deploy awal | **Vercel** | gratis, satu klik dari GitHub |
| Deploy produksi | **VPS + Coolify / Docker** | kontrol penuh, biaya tetap |
| Payment | **Xendit / Lynk.id / Midtrans** | lokal Indonesia |
| Email | **Resend / SMTP** | notifikasi transaksi |
| Storage | **Cloudflare R2 / S3** | file & gambar |

Kalau bingung: pakai daftar ini apa adanya. Baru ubah kalau punya alasan tertulis.

## Kapan Pakai yang Lain

| Kebutuhan | Pakai |
|---|---|
| Web statis / blog / landing saja | Astro, Next.js static export, HTML+Tailwind |
| Dashboard internal tanpa SEO | Vite + React SPA + backend terpisah |
| Backend berat / worker / AI pipeline | FastAPI (Python) atau NestJS |
| Realtime chat / kolaborasi | Next.js + WebSocket server, atau Supabase Realtime |
| Mobile app | React Native / Expo, backend tetap sama |
| Toko online cepat jadi | WooCommerce/Shopify — jangan buat dari nol kalau tak perlu |

## Yang Sebaiknya Dihindari Pemula

- Microservice untuk project satu orang.
- Kubernetes sebelum punya pengguna.
- GraphQL kalau REST cukup.
- Monorepo tooling (Nx/Turborepo) untuk satu aplikasi.
- Database eksotis tanpa alasan kuat.
- Framework yang dokumentasinya sedikit — agent akan mengarang API-nya.

## Cara Menulis Keputusan

Di `ARCHITECTURE.md`:

```markdown
## Keputusan Stack

- Next.js 14 App Router — alasan: satu repo untuk FE+BE, deploy gratis.
- PostgreSQL via Neon — alasan: relasional, tier gratis cukup untuk 1000 user.
- TIDAK pakai Redis dulu — alasan: belum ada masalah performa.
- TIDAK pakai microservice — alasan: tim 1 orang.
```

Tulis juga yang **tidak** dipakai. Itu mencegah agent menambah sendiri.

## Checklist

- [ ] Stack dipilih per layer
- [ ] Alasan ditulis
- [ ] Yang tidak dipakai juga ditulis
- [ ] Tercatat di `ARCHITECTURE.md`, bukan cuma di kepala
