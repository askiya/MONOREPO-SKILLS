# 03 — Build: Next.js Static Export + PHP API + MySQL (Paket C)

## Tujuan

Project Next.js bisa di-build menjadi folder `out/` berisi file HTML statis.
PHP API endpoint berfungsi. MySQL database siap dengan tabel yang dibutuhkan.

---

## Sebelum Mulai

- [ ] Antigravity sudah diberi konteks arsitektur ([02-ai-agent.md](02-ai-agent.md))
- [ ] Dokumen perencanaan sudah lengkap ([01-pedoman.md](01-pedoman.md))

---

## Bagian 1: Next.js Static Export

### 1. Buat Project Next.js

```bash
npx create-next-app@latest my-cpanel-site --typescript --tailwind --app --no-src-dir
cd my-cpanel-site
```

**Output yang benar:**
```text
✔ Would you like to use ESLint? … Yes
...
Success! Created my-cpanel-site
```

### 2. Konfigurasi Static Export

Edit `next.config.js` (atau `next.config.ts`):

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  // Kalau deploy ke subfolder cPanel, set basePath:
  // basePath: '/subfolder',
  images: {
    unoptimized: true, // wajib untuk static export
  },
}

module.exports = nextConfig
```

> ⚠️ **WAJIB:** `output: 'export'` dan `images.unoptimized: true`.
> Tanpa ini, build gagal karena cPanel tidak punya Node server untuk optimasi gambar.

### 3. Hapus Fitur yang Tidak Kompatibel

Static export **TIDAK BISA** menggunakan:

| Fitur | Kenapa | Alternatif |
|---|---|---|
| `getServerSideProps` | Butuh Node server | Fetch client-side dengan `useEffect` |
| `middleware.ts` | Butuh edge runtime | Redirect pakai `.htaccess` |
| API Routes Next.js (`app/api/`) | Butuh Node server | PHP endpoint terpisah |
| `next/image` dengan loader default | Butuh server optimasi | Set `unoptimized: true` |
| ISR / revalidate | Butuh Node server | Static-only, rebuild kalau konten berubah |
| `cookies()`, `headers()` server | Server-only | Akses via `document.cookie` di client |

### 4. Build Frontend dengan Antigravity

Kirim prompt ke Antigravity:

```text
Baca PRD.md dan DESIGN.md. Bangun halaman sesuai spesifikasi.

Ingat:
- Ini static export, semua data fetching harus client-side
- Gunakan Tailwind CSS
- Mobile-first responsive
- Buat komponen: Header, Footer, Hero, Features, CTA
- Jangan gunakan getServerSideProps atau API routes Next.js

Mulai dari halaman utama (app/page.tsx).
```

### 5. Test Build

```bash
npm run build
```

**Output yang benar:**
```text
   Creating an optimized production build ...
   Finalizing page optimization ...
   
 ✓ Generating static pages
 ✓ Collecting build traces

Route (app)                    Size
┌ ○ /                          5.2 kB
...
○  (Static)  prerendered as static content

Export successful. Files written to: out/
```

**Output yang SALAH (gagal):**
```text
Error: Page "/" is using getServerSideProps...
Error: Image Optimization using the default loader is not compatible with `next export`
```

Kalau gagal: cek kembali langkah 2 dan 3.

### 6. Verifikasi Folder `out/`

```bash
ls out/
```

Harus ada:
```text
index.html
404.html
_next/
```

`index.html` harus bisa dibuka langsung di browser tanpa server.

---

## Bagian 2: PHP API Endpoint (Opsional)

Kalau project butuh backend sederhana (form kontak, query data, dll):

### 1. Buat Folder API

```bash
mkdir api
```

### 2. Contoh: Form Kontak

Buat `api/contact.php`:

```php
<?php
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: POST, OPTIONS');
header('Access-Control-Allow-Headers: Content-Type');

if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(200);
    exit;
}

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    echo json_encode(['error' => 'Method not allowed']);
    exit;
}

$input = json_decode(file_get_contents('php://input'), true);

$nama = trim($input['nama'] ?? '');
$email = filter_var($input['email'] ?? '', FILTER_VALIDATE_EMAIL);
$pesan = trim($input['pesan'] ?? '');

if (!$nama || !$email || !$pesan) {
    http_response_code(400);
    echo json_encode(['error' => 'Semua field wajib diisi']);
    exit;
}

// Simpan ke database atau kirim email
// Contoh: simpan ke MySQL (lihat Bagian 3)

echo json_encode(['success' => true, 'message' => 'Pesan terkirim']);
```

### 3. Panggil dari Next.js (Client-Side)

```tsx
// Di komponen Next.js
const handleSubmit = async (data: FormData) => {
  const res = await fetch('/api/contact.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      nama: data.get('nama'),
      email: data.get('email'),
      pesan: data.get('pesan'),
    }),
  })
  const result = await res.json()
  // handle result
}
```

> URL `/api/contact.php` berfungsi di cPanel karena Apache langsung serve PHP.
> Di localhost, kamu perlu PHP dev server terpisah (lihat [04-preview.md](04-preview.md)).

---

## Bagian 3: MySQL Database

### 1. Buat Database di phpMyAdmin (di cPanel)

Ini dilakukan nanti saat deploy. Untuk sekarang, siapkan skema:

### 2. Buat File Skema SQL

Buat `database/schema.sql`:

```sql
CREATE DATABASE IF NOT EXISTS `<NAMA_DB>` 
  DEFAULT CHARACTER SET utf8mb4 
  COLLATE utf8mb4_unicode_ci;

USE `<NAMA_DB>`;

-- Contoh tabel kontak
CREATE TABLE IF NOT EXISTS `contacts` (
  `id` INT AUTO_INCREMENT PRIMARY KEY,
  `nama` VARCHAR(255) NOT NULL,
  `email` VARCHAR(255) NOT NULL,
  `pesan` TEXT NOT NULL,
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Tambahkan tabel lain sesuai PRD
```

### 3. Update PHP untuk Koneksi MySQL

Buat `api/config.php`:

```php
<?php
// JANGAN commit file ini dengan credential asli
// Copy dari config.example.php dan isi sendiri di server
$db_host = '<ISI_SENDIRI>';
$db_name = '<ISI_SENDIRI>';
$db_user = '<ISI_SENDIRI>';
$db_pass = '<ISI_SENDIRI>';

try {
    $pdo = new PDO(
        "mysql:host=$db_host;dbname=$db_name;charset=utf8mb4",
        $db_user,
        $db_pass,
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
} catch (PDOException $e) {
    http_response_code(500);
    echo json_encode(['error' => 'Database connection failed']);
    exit;
}
```

Buat `api/config.example.php` dengan placeholder, **tanpa credential asli**.

### 4. Update .gitignore

```bash
echo "api/config.php" >> .gitignore
```

---

## Struktur Project Akhir

```text
my-cpanel-site/
├── app/
│   ├── page.tsx          ← halaman utama
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── ...
├── api/                  ← PHP backend (opsional)
│   ├── contact.php
│   ├── config.php        ← TIDAK di-commit (ada di .gitignore)
│   └── config.example.php
├── database/
│   └── schema.sql
├── out/                  ← hasil build (static HTML)
├── next.config.js        ← output: 'export'
├── PRD.md
├── ARCHITECTURE.md
├── ...
└── .gitignore
```

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| `npm run build` gagal "getServerSideProps" | Ada code SSR | Ganti ke client-side fetch |
| `npm run build` gagal "Image Optimization" | `images.unoptimized` belum di-set | Tambahkan ke `next.config.js` |
| Folder `out/` tidak muncul | `output: 'export'` belum di-set | Cek `next.config.js` |
| PHP error "Class PDO not found" | PHP di hosting tanpa PDO | Aktifkan extension PDO di cPanel → PHP Selector |
| CORS error saat fetch API | Header CORS belum di-set di PHP | Tambahkan header `Access-Control-Allow-Origin` |

---

## Checklist

- [ ] `next.config.js` punya `output: 'export'` dan `images.unoptimized: true`
- [ ] `npm run build` berhasil tanpa error
- [ ] Folder `out/` berisi `index.html` dan `_next/`
- [ ] `index.html` bisa dibuka langsung di browser (double-click)
- [ ] PHP API endpoint ada (kalau project butuh backend)
- [ ] File `schema.sql` siap untuk diimpor ke phpMyAdmin
- [ ] `api/config.php` ada di `.gitignore`
- [ ] Tidak ada `getServerSideProps` atau API routes Next.js di project
