# 04 — Preview dan Testing dengan Docker Compose

## Tujuan

Seluruh stack (web, API, PostgreSQL) berjalan lokal lewat Docker Compose, semua service sehat, dan alur PRD teruji sebelum menyentuh VPS.

## Prasyarat

- Bab `03-build.md` selesai.
- Docker dan Docker Compose berjalan (`docker --version`).
- File `.env` lokal terisi.

## 1. Jalankan Stack

```bash
docker compose up --build
```

Biarkan log berjalan di terminal ini. Buka terminal kedua untuk verifikasi.

**Hasil yang diharapkan:** tiga service naik; log `db` menunjukkan `database system is ready to accept connections`; log `api` menunjukkan `API listening on 4000`; log `web` menunjukkan server Next.js siap.

## 2. Cek Status Service

```bash
docker compose ps
```

**Hasil yang diharapkan:** kolom status semua service `Up`, dan `db` bertanda `healthy`.

Jika ada service `Restarting`, baca lognya:

```bash
docker compose logs api
docker compose logs web
```

## 3. Uji Health

```bash
curl -s http://localhost:4000/health
curl -s http://localhost:3000
```

**Hasil yang diharapkan:** API mengembalikan `{"status":"ok"}`; web mengembalikan HTML.

## 4. Uji Koneksi Database

```bash
docker compose exec db psql -U dev -d myapp -c '\dt'
```

**Hasil yang diharapkan:** daftar tabel hasil migrasi Prisma. Bila kosong, migrasi belum dijalankan.

Jalankan migrasi di dalam container API:

```bash
docker compose exec api npx prisma migrate deploy
```

## 5. Verifikasi Isolasi Jaringan

Database tidak boleh terjangkau dari luar container:

```bash
curl -s --max-time 3 http://localhost:5432
```

**Hasil yang diharapkan:** gagal terhubung. Jika berhasil terhubung, berarti port `5432` dipublish — hapus dari `docker-compose.yml`.

Sementara API harus terjangkau dari dalam network:

```bash
docker compose exec web wget -qO- http://api:4000/health
```

**Hasil yang diharapkan:** `{"status":"ok"}`.

## 6. Uji Alur End-to-End

```bash
curl -s -X POST http://localhost:4000/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Ebook Docker","price":120000}'

curl -s http://localhost:4000/products
```

**Hasil yang diharapkan:** POST mengembalikan 201 dengan `id`; GET mengembalikan array berisi data tersebut.

Lalu buka `http://localhost:3000` dan pastikan web menampilkan data dari API, bukan data hardcode.

## 7. Uji Persistensi Volume

```bash
docker compose restart db
docker compose exec db psql -U dev -d myapp -c 'SELECT COUNT(*) FROM "Product";'
```

**Hasil yang diharapkan:** jumlah baris tetap sama. Data hilang berarti volume `pgdata` tidak terpasang.

## 8. Jalankan Test Otomatis

```bash
npm run lint --workspaces
npm test --workspaces
```

**Hasil yang diharapkan:** exit code `0` untuk semua workspace.

## 9. Smoke Test Manual

- [ ] Web termuat di desktop dan lebar 375 px.
- [ ] Alur inti PRD selesai lewat UI.
- [ ] State loading, kosong, error, sukses tampil benar.
- [ ] Validasi input menolak data salah dengan pesan jelas.
- [ ] Error API tidak menampilkan stack trace ke pengguna.
- [ ] Navigasi keyboard mencapai semua kontrol.

## 10. Bersihkan (Hati-hati)

Hentikan stack tanpa menghapus data:

```bash
docker compose down
```

**Peringatan:** jangan gunakan `docker compose down -v` kecuali kamu memang ingin menghapus seluruh isi database. Flag `-v` menghapus volume `pgdata` dan data tidak dapat dikembalikan tanpa backup.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| `api` exit code 1 | Env kurang | Cek `.env`, `docker compose logs api` |
| API tidak bisa connect DB | Pakai `localhost` bukan `db` | Ubah `DATABASE_URL` host menjadi `db` |
| `db` unhealthy | Credential env tidak konsisten | Samakan `POSTGRES_*` di `.env` dan compose |
| Web fetch gagal dari browser | `NEXT_PUBLIC_API_URL` menunjuk `api:4000` | Browser butuh `http://localhost:4000` |
| Perubahan kode tidak terlihat | Image belum dibangun ulang | `docker compose up --build` |
| Disk penuh | Image lama menumpuk | `docker image prune` |

## Checklist

- [ ] Tiga service `Up`, `db` `healthy`.
- [ ] Health web dan API terverifikasi.
- [ ] Migrasi terpasang dan tabel ada.
- [ ] Port `5432` tidak terjangkau dari host publik.
- [ ] Alur end-to-end POST dan GET lulus.
- [ ] Data bertahan setelah restart container DB.
- [ ] Lint dan test semua workspace lulus.
- [ ] Smoke test manual lulus.
