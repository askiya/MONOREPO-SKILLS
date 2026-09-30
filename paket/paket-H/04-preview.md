# H4 — Preview dan Testing Monorepo

> 🟠 LANJUTAN · Target waktu: 3–4 jam

## Tujuan

Menjalankan seluruh graph dengan `turbo dev`, menguji workspace terisolasi, kontrak shared, integrasi web–API, dan build root.

## Langkah 1 — Jalankan Semua App

Dari root:

```bash
npm install
npm run dev
```

**Output yang diharapkan:** Turbo menjalankan task persistent; web di <http://localhost:3000>, API di <http://localhost:4000>.

Uji API:

```bash
curl -i http://localhost:4000/health
```

**Output yang diharapkan:** HTTP 200 dan `{"status":"ok"}`.

## Langkah 2 — Jalankan App Terfilter

```bash
npx turbo dev --filter=@santriverse/web
npx turbo dev --filter=@santriverse/api
```

Jalankan satu per satu karena keduanya persistent.

**Hasil yang diharapkan:** hanya workspace target dan dependency yang diperlukan berjalan.

## Langkah 3 — Uji API Validasi

```bash
curl -i -X POST http://localhost:4000/v1/orders \
  -H "Content-Type: application/json" \
  -d '{"productId":"produk-1","namaPembeli":"Santri Uji","email":"uji@example.com"}'

curl -i -X POST http://localhost:4000/v1/orders \
  -H "Content-Type: application/json" \
  -d '{"productId":"","namaPembeli":"A","email":"salah"}'
```

**Output yang diharapkan:** input valid 201 dengan `data.id`; input invalid 400 dengan `error.code=INVALID_INPUT`.

## Langkah 4 — Testing per Workspace

```bash
npm --workspace @santriverse/shared test
npm --workspace @santriverse/api test
npm --workspace @santriverse/web test
```

Jika template web belum punya script test, tambahkan test minimal sesuai tool yang sudah dipilih project; jangan menambahkan framework hanya agar checklist hijau. Build/typecheck tetap wajib.

```bash
npm --workspace @santriverse/shared run build
npm --workspace @santriverse/api run build
npm --workspace @santriverse/web run build
```

**Output yang diharapkan:** tiap workspace selesai dengan exit code 0.

## Langkah 5 — Uji Kontrak Shared

Ubah sementara batas `namaPembeli` pada shared atau tambah field optional, lalu:

```bash
npx turbo run build --filter=@santriverse/shared...
```

**Hasil yang diharapkan:** shared dan semua dependent (web/API) dibangun. Kembalikan perubahan uji bila bukan bagian task.

Untuk perubahan breaking, build harus menunjukkan consumer yang perlu diperbaiki—jangan melemahkan type untuk membuat build hijau.

## Langkah 6 — Uji CORS

```bash
curl -i -X OPTIONS http://localhost:4000/v1/orders \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: POST"
```

**Output yang diharapkan:** header `Access-Control-Allow-Origin: http://localhost:3000`.

Uji origin yang tidak diizinkan setelah konfigurasi strict; server tidak boleh memberi izin origin produksi palsu.

## Langkah 7 — Uji dari Web

1. Buka web.
2. Isi form valid; pastikan network request menuju port 4000 dan status 201.
3. Isi email salah; validasi client dan server konsisten.
4. Matikan API; web harus menampilkan pesan gagal yang bisa dipahami, bukan spinner selamanya.
5. Nyalakan API; coba ulang berhasil.

## Langkah 8 — Gate Root

```bash
npx turbo run lint test build
```

Jalankan ulang untuk menguji cache:

```bash
npx turbo run lint test build
```

**Output yang diharapkan:** run pertama semua sukses; run kedua menampilkan cache hit/replay untuk task cacheable.

## Error Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Hanya satu app hidup | Script `dev`/turbo task salah | Pastikan dua workspace punya `dev` |
| CORS browser gagal, curl biasa sukses | Origin tidak cocok | Set `CORS_ORIGIN=http://localhost:3000` |
| Test root melewatkan app | App tidak punya script `test` | Tambah script atau dokumentasikan gate pengganti |
| Cache tidak pernah hit | Output/env input tidak benar | Periksa `outputs` dan konfigurasi env Turbo |
| Perubahan shared tidak mengetes consumer | Filter salah | Gunakan `--filter=@santriverse/shared...` |

## Checklist

- [ ] `npm run dev` menjalankan web + API
- [ ] Filter tiap app bekerja
- [ ] API valid 201 dan invalid 400
- [ ] Test/build tiap workspace sukses
- [ ] Perubahan shared membangun consumer
- [ ] CORS hanya mengizinkan origin yang benar
- [ ] Web menangani API mati
- [ ] Root lint/test/build exit 0
- [ ] Run kedua menunjukkan cache hit

➡️ Lanjut ke **[05-deploy.md](05-deploy.md)**.
