# G4 — Preview Localhost dan Testing

> 🔵 MENENGAH · Target waktu: 2–3 jam · Prasyarat: build lokal selesai

## Tujuan

Memastikan fitur, API, database, tampilan, dan build produksi bekerja sebelum memakai Railway.

## Langkah 1 — Jalankan Development Server

```bash
npm install
npx prisma migrate dev
npm run dev
```

Buka <http://localhost:3000>.

**Output yang diharapkan:** terminal menampilkan `Ready`, halaman terbuka tanpa error overlay.

## Langkah 2 — Uji Jalur Utama Manual

Uji dengan jendela incognito agar sesi lama tidak menipu hasil.

| Skenario | Langkah | Hasil benar |
|---|---|---|
| Daftar produk | Buka `/produk` | Minimal 3 produk dari DB tampil |
| Detail | Klik satu produk | Nama, harga, deskripsi cocok |
| Pesanan valid | Isi nama/email valid, submit | Pesan sukses; 1 order baru di DB |
| Pesanan invalid | Email `abc`, submit | Error jelas; DB tidak bertambah |
| URL tidak ada | Buka `/produk/tidak-ada` | Halaman 404, bukan crash |
| Admin tanpa login | Buka `/admin` incognito | Dialihkan ke login / ditolak |

Cek data:

```bash
npx prisma studio
```

**Output yang diharapkan:** tabel `Order` memuat tepat satu data dari skenario valid.

## Langkah 3 — Uji API

```bash
curl -i http://localhost:3000/api/health
curl -i http://localhost:3000/api/products
curl -i -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -d '{"productId":"ID_PRODUK_VALID","namaPembeli":"Santri Uji","email":"uji@example.com"}'
curl -i -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -d '{"productId":"","namaPembeli":"A","email":"salah"}'
```

Ganti `ID_PRODUK_VALID` dengan ID dari Prisma Studio.

**Output yang diharapkan:** health `200`, create valid `201`, payload invalid `400`.

## Langkah 4 — Uji Tampilan Responsif

DevTools → Toggle Device Toolbar. Cek:

- 360×800 (ponsel kecil)
- 768×1024 (tablet)
- 1440×900 (desktop)

Tidak boleh ada scroll horizontal, tombol terpotong, teks bertumpuk, atau form sulit disentuh.

## Langkah 5 — Uji Gangguan Database

1. Hentikan PostgreSQL lokal.
2. Panggil health check.

```bash
curl -i http://localhost:3000/api/health
```

**Output yang diharapkan:** HTTP `503` dan JSON `database: "unreachable"`, bukan HTML stack trace.

3. Nyalakan PostgreSQL lagi.
4. Panggil ulang sampai mendapat `200`.

## Langkah 6 — Simulasi Produksi

Hentikan dev server, lalu:

```bash
npm run lint
npm run build
npm start
```

**Output yang diharapkan:** build tanpa error dan aplikasi tersedia di <http://localhost:3000>.

Ulangi tiga jalur utama: list produk, create order, health check.

## Bukti Kelulusan

Simpan:

- Screenshot halaman produk pada 360×800 dan 1440×900.
- Output terminal `npm run build` sukses.
- Output `curl -i /api/health` berstatus 200.
- Screenshot satu order uji di Prisma Studio.

Jangan sertakan `.env`, URL database, cookie, atau secret dalam screenshot.

## Error Umum

| Gejala | Cek | Perbaikan |
|---|---|---|
| Dev jalan, build gagal | Error TypeScript/build log | Perbaiki sebelum deploy; jangan abaikan |
| API 500 | Terminal server | Validasi env, migrasi, dan query |
| Order dobel | Tombol bisa ditekan ulang | Disable saat loading + idempotensi bila transaksi penting |
| Data tidak berubah | App memakai DB lain | Cocokkan host/database pada `DATABASE_URL` tanpa menyalin secret |
| Mobile melebar | Elemen fixed width | Gunakan `max-width`, grid responsif, `overflow-wrap` |

## Checklist

- [ ] Semua skenario manual lulus
- [ ] Status API sesuai: 200, 201, 400, 503
- [ ] Data valid masuk DB; data invalid tidak masuk
- [ ] 3 viewport bebas overflow
- [ ] Gangguan DB menghasilkan respons aman
- [ ] `npm run lint` sukses
- [ ] `npm run build` sukses
- [ ] Bukti kelulusan tersimpan tanpa secret

➡️ Lanjut ke **[05-deploy.md](05-deploy.md)**.
