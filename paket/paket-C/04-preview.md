# 04 — Preview: Localhost & Test Build Output (Paket C)

## Tujuan

Website berjalan di localhost. Output build (`out/`) diverifikasi bisa diakses
tanpa server Node.js (seperti kondisi di cPanel nanti).

---

## Sebelum Mulai

- [ ] `npm run build` sudah berhasil ([03-build.md](03-build.md))
- [ ] Folder `out/` sudah ada

---

## Langkah

### 1. Preview Development Mode

```bash
npm run dev
```

Buka `http://localhost:3000`. Cek:
- Semua halaman muncul
- Navigasi antar halaman berfungsi
- Responsive di mobile (buka DevTools → toggle device toolbar)
- Tidak ada error di console browser

**Output yang benar:** website tampil tanpa error di console.

### 2. Preview Static Build (Simulasi cPanel)

Karena di cPanel nanti **tidak ada Node server**, kamu harus tes apakah file
statis `out/` bisa berjalan sendiri.

#### Cara 1: Pakai `npx serve`

```bash
npx serve out
```

**Output yang benar:**
```text
   ┌──────────────────────────────────────┐
   │                                      │
   │   Serving!                           │
   │                                      │
   │   - Local:    http://localhost:3000   │
   │                                      │
   └──────────────────────────────────────┘
```

Buka URL tersebut. Website harus tampil sama seperti di `npm run dev`.

#### Cara 2: Buka File Langsung

Double-click `out/index.html` di file explorer. Halaman harus tampil
(meski beberapa fitur routing mungkin tidak jalan di `file://` protocol).

### 3. Test PHP API (Kalau Ada)

Kalau project punya PHP endpoint, jalankan PHP dev server terpisah:

```bash
cd api
php -S localhost:8080
```

**Output yang benar:**
```text
PHP Development Server started at http://localhost:8080
```

Test endpoint:

```bash
curl -X POST http://localhost:8080/contact.php \
  -H "Content-Type: application/json" \
  -d '{"nama":"Test","email":"test@test.com","pesan":"Halo"}'
```

**Output yang benar:**
```json
{"success":true,"message":"Pesan terkirim"}
```

> Di localhost, frontend (port 3000) dan PHP (port 8080) beda port.
> Di cPanel nanti, keduanya di domain yang sama — CORS tidak masalah.

### 4. Verifikasi Tidak Ada Fitur SSR Tersisa

```bash
# Cari getServerSideProps yang tersisa
grep -r "getServerSideProps" app/ --include="*.tsx" --include="*.ts"
grep -r "getServerSideProps" pages/ --include="*.tsx" --include="*.ts" 2>/dev/null

# Cari middleware
ls middleware.ts 2>/dev/null
ls middleware.js 2>/dev/null
```

**Output yang benar:** tidak ada hasil (kosong). Kalau ada, hapus atau ganti
ke client-side.

### 5. Cek Ukuran Build

```bash
du -sh out/
```

Pastikan ukuran wajar untuk hosting:
- Landing page: < 5 MB
- Website kecil: < 20 MB
- Dengan gambar banyak: < 50 MB (pertimbangkan kompresi gambar)

> cPanel shared hosting biasanya punya limit disk 1-10 GB. File statis kecil
> bukan masalah, tapi gambar besar bisa cepat habis kuota.

### 6. Test di Browser Berbeda

Buka `http://localhost:3000` (atau `npx serve out`) di:
- Chrome
- Firefox
- Safari (kalau ada)
- Browser HP (via IP lokal: `http://IP-LAPTOP:3000`)

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Halaman blank di `npx serve out` | Path asset salah, basePath tidak cocok | Cek `next.config.js` basePath |
| 404 saat navigasi di static serve | SPA routing butuh rewrite | Pakai `.htaccess` rewrite (sudah ada di `deployment-examples/cpanel-static/`) |
| PHP error di localhost | PHP belum terinstal | Install PHP: Windows → XAMPP/WampServer, Mac → `brew install php` |
| CORS error frontend → PHP | Beda port | Tambahkan CORS header di PHP (sudah ada di contoh 03-build.md) |
| Gambar tidak muncul | Path gambar hardcode `/` tapi basePath ada | Pakai `basePath` di path gambar atau relative path |

---

## Checklist

- [ ] `npm run dev` — website tampil tanpa error console
- [ ] `npx serve out` — website tampil dari folder statis
- [ ] Navigasi antar halaman berfungsi
- [ ] Responsive di mobile (DevTools device toolbar)
- [ ] PHP API endpoint berfungsi di localhost (kalau ada)
- [ ] Tidak ada `getServerSideProps` atau middleware tersisa
- [ ] Ukuran `out/` wajar untuk hosting
