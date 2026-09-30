# 05 — Node.js App di cPanel

## Tujuan

Menjalankan aplikasi Node.js (Express, Next.js) di shared hosting cPanel.

## Cek Dulu: Apakah Hosting Mendukung?

cPanel → cari menu **Setup Node.js App**.

- **Ada** → lanjut.
- **Tidak ada** → hosting tidak mendukung Node. Pilihan:
  - upgrade paket,
  - pakai static export (tanpa server),
  - pindah ke VPS,
  - taruh frontend di cPanel, backend di Vercel/Railway.

## Batasan Shared Hosting

Sadari sebelum mulai:

| Batasan | Dampak |
|---|---|
| RAM kecil (256MB–1GB) | `npm run build` sering gagal |
| Proses dibatasi | app bisa di-kill saat trafik naik |
| Tidak ada root | tidak bisa install system package |
| Versi Node terbatas | pilih dari daftar yang tersedia |
| Tidak ada background worker | cron job terbatas |
| Timeout request | request lama diputus |

**Strategi:** build di lokal, upload hasil build. Jangan build di server.

## Langkah Setup

### 1. Siapkan Project Lokal

```bash
npm run build
```

Untuk Next.js, aktifkan standalone di `next.config.js`:

```js
module.exports = {
  output: 'standalone',
}
```

### 2. Upload Project

Upload ke folder **di luar** `public_html`, misalnya:

```
/home/username/nodeapp/
```

Yang perlu di-upload:
- `package.json`, `package-lock.json`
- hasil build (`.next/standalone`, `.next/static`, `public`)
- file entry (`server.js` atau `app.js`)

**Jangan** upload `node_modules` — akan di-install di server.

### 3. Buat Node.js App

cPanel → **Setup Node.js App** → **Create Application**:

| Field | Isi |
|---|---|
| Node.js version | pilih versi terbaru yang tersedia (20/22) |
| Application mode | `Production` |
| Application root | `nodeapp` (relatif dari home) |
| Application URL | `domainmu.com` atau `api.domainmu.com` |
| Application startup file | `server.js` |

Klik **Create**.

### 4. Environment Variables

Di halaman Node.js App yang sama, scroll ke **Environment Variables**:

| Name | Value |
|---|---|
| `NODE_ENV` | `production` |
| `PORT` | (biasanya diisi otomatis cPanel) |
| `DATABASE_URL` | `mysql://user:pass@localhost/dbname` |
| `NEXTAUTH_SECRET` | hasil generate random |
| `NEXTAUTH_URL` | `https://domainmu.com` |

**Klik Add setiap variabel, lalu Save.**

Ini cara yang benar. **Jangan** upload file `.env` ke `public_html`.

### 5. Install Dependencies

Di halaman Node.js App, klik **Run NPM Install**.

Kalau gagal karena memori:
- hapus devDependencies dari `package.json` sebelum upload,
- atau upload `node_modules` hasil `npm ci --omit=dev` dari lokal.

### 6. Start Aplikasi

Klik **Start App** / **Restart**.

Cek log kalau error: ada tombol log di panel, atau file `stderr.log` di folder app.

### 7. Hubungkan ke Domain

Kalau app tidak langsung muncul di domain, tambah `.htaccess` di `public_html`:

```apache
RewriteEngine On
RewriteRule ^$ http://127.0.0.1:PORT/ [P,L]
RewriteCond %{REQUEST_URI} !^/?(index\.php|public|robots\.txt)
RewriteRule ^/?(.*)$ http://127.0.0.1:PORT/$1 [P,L]
```

Ganti `PORT` dengan port dari panel Node.js App.

Biasanya cPanel sudah membuat `.htaccess` otomatis. Jangan timpa kalau sudah ada.

## Update Aplikasi

Setiap kali kode berubah:

1. Build di lokal.
2. Upload file yang berubah.
3. cPanel → Setup Node.js App → **Restart**.

## Troubleshooting

| Gejala | Cek |
|---|---|
| 503 Service Unavailable | app tidak jalan — cek log, restart |
| 500 Internal Server Error | error runtime — baca `stderr.log` |
| App start lalu mati | memori habis atau crash — cek log |
| `Cannot find module` | `npm install` belum jalan / gagal |
| Env tidak terbaca | belum di-Save di panel, atau belum restart |
| Port conflict | biarkan cPanel yang assign port |

## Checklist

- [ ] Hosting punya Setup Node.js App
- [ ] Project di luar `public_html`
- [ ] Env variables diisi di panel, bukan file `.env` publik
- [ ] `npm install` sukses
- [ ] App status **Running**
- [ ] Domain menampilkan aplikasi
- [ ] Prosedur restart setelah update dipahami
