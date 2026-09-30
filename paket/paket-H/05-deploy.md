# H5 — Deploy: Vercel (Web) + Railway/Render (API)

> 🟠 LANJUTAN · Target waktu: 2–3 jam

## Tujuan

Frontend live di Vercel, API live di Railway atau Render, masing-masing memakai env terpisah dan terhubung.

> Harga, UI platform, dan fitur dapat berubah. Cocokkan langkah aktual dengan dokumentasi resmi masing-masing.

## Prasyarat

- Repo monorepo di GitHub, `npx turbo run lint build` sukses.
- Akun Vercel + Railway/Render.
- Tidak ada secret dalam repo.

---

## Bagian A — Deploy API ke Railway (atau Render)

### Opsi 1: Railway

1. Railway → **New Project → Deploy from GitHub repo**.
2. Pilih repo monorepo.
3. **Settings → Source**:
   - Root Directory: `apps/api`
   - Branch: `main`
4. Build Command: `npm run build`
5. Start Command: `npm start`

#### Tambah PostgreSQL (jika diperlukan)

1. **Add Service → Database → PostgreSQL**.
2. Referensikan `DATABASE_URL` ke service API.

#### Environment Variables

| Variable | Nilai |
|---|---|
| `PORT` | Railway sediakan otomatis |
| `DATABASE_URL` | reference variable Postgres |
| `CORS_ORIGIN` | `https://app.domainmu.com` (domain web setelah tersedia) |
| `NODE_ENV` | production |

**Hasil yang diharapkan:** build lalu deploy sehat; `https://....up.railway.app/health` memberi 200.

> ⚠️ Root Directory Railway menentukan konteks build. Pastikan path benar untuk monorepo.
> Turborepo tasks dependsOn yang melibatkan shared perlu build berjalan dari root atau shared sudah built.
> Jika platform hanya bisa build subdirectory, pertimbangkan custom build script yang menjalankan `npx turbo run build --filter=@santriverse/api` dari root.

### Opsi 2: Render

1. Render → **New → Web Service → From GitHub**.
2. Root Directory: `apps/api`.
3. Build Command: `npm run build`
4. Start Command: `npm start`
5. Environment Variables: sama seperti Railway, tanpa reference; set manual.

**Hasil yang diharapkan:** Render URL memberi health 200.

---

## Bagian B — Deploy Web ke Vercel

1. Vercel → **Add New → Project → Import** repo monorepo.
2. Konfigurasi kritis:
   - **Framework Preset**: Next.js
   - **Root Directory**: `apps/web`
3. Build & Output Settings biasanya terdeteksi otomatis dari Next.js; periksa build command memakai `npm run build` dari workspace context.

#### Environment Variables Vercel

| Variable | Nilai |
|---|---|
| `NEXT_PUBLIC_API_URL` | URL API produksi (Railway/Render URL, atau domain API nanti) |

Jangan taruh `DATABASE_URL` di Vercel; web tidak boleh menyentuh database langsung.

**Hasil yang diharapkan:** Vercel build sukses; domain `*.vercel.app` menampilkan web.

### Vercel + Monorepo: Root Directory Setting

Vercel mengonfigurasi workspace lewat **Root Directory**. Jika build gagal karena shared package tidak ditemukan:

1. Pastikan `Install Command` di Vercel adalah `npm install` dari root (bukan workspace).
2. Atau override Build Command ke `cd ../.. && npx turbo run build --filter=@santriverse/web`.
3. Periksa bahwa `package-lock.json` ada di root repository.

---

## Bagian C — Sambungkan Web ↔ API

Setelah kedua service live:

1. Set `CORS_ORIGIN` API ke URL web produksi. Redeploy API.
2. Set `NEXT_PUBLIC_API_URL` web ke URL API produksi. Redeploy web.

Uji:

```bash
curl -I https://URL-API/health
curl -I https://URL-WEB
```

Buka web produksi:
- Buat pesanan lewat form.
- Buka DevTools Network; pastikan request ke API produksi, bukan localhost.
- Pastikan tidak ada error CORS.

## Bagian D — Atur Deploy Terpisah

Keuntungan monorepo: deploy independen. Pastikan platform hanya deploy saat workspace terkait berubah:

**Vercel:** secara default mendeteksi perubahan berdasarkan Root Directory. Untuk kontrol lebih: **Settings → Git → Ignored Build Step** — gunakan `npx turbo-ignore`.

**Railway:** Railway mendeploy setiap push. Untuk filtering:
- Gunakan branch strategy: merge ke branch tertentu per service.
- Atau gunakan Railway config yang mendukung watch paths jika tersedia.

## Error Umum

| Gejala | Penyebab | Solusi |
|---|---|---|
| `Cannot find module @santriverse/shared` di Vercel | Install berjalan dari workspace saja | Install dari root; periksa Root Directory dan Install Command |
| CORS error di browser | `CORS_ORIGIN` belum di-set/salah | Cocokkan origin web produksi; redeploy API |
| API build OK tapi start crash | `dist/` tidak berisi file compiled | Periksa `tsconfig` dan `build` script |
| Vercel deploy app dan API berubah setiap push | Ignored Build Step tidak diset | Tambah `npx turbo-ignore` |
| `NEXT_PUBLIC_API_URL` undefined di browser | Tidak di-set sebagai env Vercel | Tambah dan redeploy |
| Railway Root Directory salah | Path dari root, bukan nama folder | Gunakan `apps/api` |

## Checklist

- [ ] API live, health 200
- [ ] Web live, halaman utama muncul
- [ ] `CORS_ORIGIN` cocok URL web produksi
- [ ] `NEXT_PUBLIC_API_URL` cocok URL API produksi
- [ ] Form web → API produksi berhasil, tidak ada CORS error
- [ ] `DATABASE_URL` hanya ada di API, tidak ada di Vercel
- [ ] Secret tidak ada di source code
- [ ] Deploy terpisah diatur (turbo-ignore / watch paths)
- [ ] Deploy API dahulu jika API berubah; web setelah health hijau

➡️ Lanjut ke **[06-domain-ssl.md](06-domain-ssl.md)**.
