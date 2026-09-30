# 04 — Preview dan Testing Lokal

## Tujuan

Aplikasi berjalan di localhost, semua alur PRD teruji manual dan otomatis, sebelum menyentuh VPS.

## Prasyarat

- Bab `03-build.md` selesai.
- PostgreSQL lokal hidup.

## 1. Jalankan Development Server

```bash
npm run dev
```

Buka `http://localhost:3000`.

**Hasil yang diharapkan:** halaman termuat tanpa error di terminal dan tanpa error merah di console browser.

## 2. Uji Health Endpoint

```bash
curl -s http://localhost:3000/api/health
```

**Hasil yang diharapkan:**

```json
{"status":"ok"}
```

Bila muncul `{"status":"error"}` dengan HTTP 503, database tidak terjangkau. Periksa container PostgreSQL:

```bash
docker ps
```

## 3. Uji API Manual

```bash
curl -s -X POST http://localhost:3000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Ebook Vibe Coding","price":99000}'
```

**Hasil yang diharapkan:** HTTP 201 dan JSON produk berisi `id` dan `createdAt`.

Baca kembali:

```bash
curl -s http://localhost:3000/api/products
```

**Hasil yang diharapkan:** array berisi produk yang baru dibuat.

## 4. Uji Build Produksi Lokal

Development server tidak mewakili produksi. Uji build nyata:

```bash
npm run build
node .next/standalone/server.js
```

Buka `http://localhost:3000`.

**Hasil yang diharapkan:** aplikasi berjalan dari output standalone. Ini mode yang sama dengan yang dijalankan Coolify.

> Jika `server.js` tidak ada, `output: "standalone"` belum aktif di `next.config.js`.

Bila asset atau CSS hilang saat menjalankan standalone secara manual, itu karena folder `public` dan `.next/static` harus berada di samping `server.js` — di Docker/Coolify hal ini ditangani oleh proses build image.

## 5. Tulis Test Minimal

Pasang Vitest:

```bash
npm install -D vitest
```

Tambah script:

```json
{
  "scripts": {
    "test": "vitest run"
  }
}
```

Contoh test integrasi sederhana untuk validasi harga:

```ts
import { describe, expect, it } from "vitest";

function validatePrice(price: unknown) {
  if (typeof price !== "number" || price <= 0) throw new Error("invalid price");
  return price;
}

describe("validatePrice", () => {
  it("menerima harga positif", () => {
    expect(validatePrice(99000)).toBe(99000);
  });

  it("menolak harga nol dan negatif", () => {
    expect(() => validatePrice(0)).toThrow();
    expect(() => validatePrice(-1)).toThrow();
  });

  it("menolak tipe bukan angka", () => {
    expect(() => validatePrice("99000")).toThrow();
  });
});
```

Jalankan:

```bash
npm test
```

**Hasil yang diharapkan:** semua test lulus, exit code `0`.

## 6. Smoke Test Manual

Uji setiap alur utama dari PRD:

- [ ] Halaman utama termuat di desktop dan mobile (lebar 375 px).
- [ ] Alur inti selesai dari awal sampai akhir.
- [ ] State loading terlihat saat data dimuat.
- [ ] State kosong muncul saat belum ada data.
- [ ] Error server menampilkan pesan ramah, bukan stack trace.
- [ ] Validasi input menolak data salah.
- [ ] Navigasi keyboard (Tab) bisa mencapai semua kontrol.
- [ ] Refresh halaman tidak menghilangkan data tersimpan.

## 7. Quality Gate Sebelum Deploy

```bash
npm run lint
npm test
npm run build
```

Ketiga perintah harus exit code `0`. Jangan deploy bila salah satu gagal.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| `EADDRINUSE: 3000` | Port dipakai proses lain | Matikan proses lama atau pakai `PORT=3001 npm run dev` |
| Halaman blank, console error | Error runtime client | Baca console browser, perbaiki komponen |
| `500` pada API | Query Prisma gagal | Baca log terminal, cek nama field dan migrasi |
| Data hilang setelah restart | Container DB tanpa volume | Tambah volume atau terima sifat ephemeral untuk dev |
| Build lokal lulus, standalone gagal | Env dibutuhkan saat runtime | Pastikan semua env runtime tersedia |

## Checklist

- [ ] `npm run dev` jalan tanpa error.
- [ ] Health endpoint mengembalikan `ok`.
- [ ] API create dan read terverifikasi dengan `curl`.
- [ ] `node .next/standalone/server.js` berjalan.
- [ ] Test otomatis ada dan lulus.
- [ ] Smoke test manual semua alur PRD lulus.
- [ ] Lint, test, build semuanya exit `0`.
