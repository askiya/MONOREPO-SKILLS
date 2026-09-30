# 02 — Decision Tree Troubleshooting

Ikuti cabang dari gejala. Berhenti di cabang yang cocok, jangan acak.

---

## A. `npm install` gagal

```text
npm install gagal
├── "EACCES permission denied"
│     → jangan sudo. perbaiki permission folder / pakai nvm
├── "ERESOLVE unable to resolve dependency tree"
│     → konflik peer dependency
│     → baca package mana yang bentrok, sesuaikan versi
│     → jangan langsung --force / --legacy-peer-deps tanpa paham
├── "ENOENT no such file package.json"
│     → salah folder. cd ke root project
├── "network timeout / ECONNRESET"
│     → koneksi/registry. coba lagi, cek proxy
├── "engine node version incompatible"
│     → versi Node salah. cek "engines" di package.json
└── "out of memory"
      → tambah swap / build di mesin lain
```

---

## B. Localhost tidak muncul

```text
buka localhost:3000 tidak bisa
├── terminal menampilkan error saat npm run dev
│     → baca error pertama, bukan terakhir
├── terminal bilang "ready" tapi browser timeout
│     ├── port beda dari 3000 → cek output terminal
│     ├── firewall lokal memblokir → izinkan Node
│     └── app bind ke 0.0.0.0 vs localhost → cek config
├── "EADDRINUSE port already in use"
│     → proses lain pakai port
│     → matikan proses atau PORT=3001 npm run dev
├── halaman blank putih
│     → buka DevTools Console, baca error JS
└── 404 di semua route
      → struktur folder app/ salah, cek penempatan page.tsx
```

---

## C. Git push gagal

```text
git push gagal
├── "Authentication failed"
│     → token/credential salah atau expired
│     → pakai Personal Access Token, bukan password
├── "Permission denied (publickey)"
│     → SSH key belum terdaftar di GitHub
├── "rejected - non-fast-forward"
│     → remote punya commit yang kamu belum punya
│     → git pull dulu, selesaikan konflik, baru push
├── "remote: Repository not found"
│     → URL salah / tidak punya akses / repo private
├── "file too large / exceeds 100MB"
│     → ada file besar ter-commit
│     → hapus dari history, tambahkan ke .gitignore
└── "would clobber existing tag / diverged"
      → jangan force push tanpa paham. tanya mentor
```

---

## D. Vercel build gagal

```text
build gagal di Vercel (padahal lokal sukses)
├── "Type error"
│     → lokal mungkin skip typecheck. perbaiki type, jangan any
├── "Module not found"
│     ├── kapitalisasi file beda → Linux case-sensitive, Windows tidak
│     └── dependency di devDependencies padahal dibutuhkan build
├── "Missing environment variable"
│     → tambahkan di Vercel Settings → Environment Variables
│     → jangan lupa Redeploy setelah menambah
├── "Prisma Client not generated"
│     → tambahkan "postinstall": "prisma generate" di package.json
├── "Command exited with 1" tanpa detail
│     → scroll log ke atas, error asli ada di atas
└── build sukses tapi runtime 500
      → cek Function Logs, biasanya env/DB
```

---

## E. Database tidak terkoneksi

```text
tidak bisa connect database
├── "P1001 Can't reach database server"
│     ├── host/port salah → cek connection string
│     ├── IP tidak di-allowlist → tambahkan di dashboard DB
│     └── DB sedang mati/suspend → cek dashboard provider
├── "password authentication failed"
│     ├── password salah
│     └── karakter khusus belum di-URL-encode (@ → %40)
├── "database does not exist"
│     → nama DB salah. cPanel: ingat prefix username_
├── "too many connections"
│     → connection pool. pakai singleton Prisma client
├── "SSL connection required"
│     → tambah ?sslmode=require di connection string
└── konek lokal OK, gagal di deploy
      → env di platform deploy belum diisi / beda
```

---

## F. cPanel: domain tidak terbuka

```text
domain tidak bisa dibuka
├── nslookup → IP salah / NXDOMAIN
│     → DNS belum benar. cek A record / nameserver
├── nslookup benar, tapi timeout
│     → hosting down / firewall. hubungi support
├── HTTPS error / "not secure"
│     → SSL belum aktif. jalankan AutoSSL
│     → atau mode Cloudflare salah (Flexible → Full Strict)
├── 403 Forbidden
│     ├── permission folder salah (harus 755, file 644)
│     ├── tidak ada index.html/index.php
│     └── .htaccess memblokir
├── 404 Not Found
│     ├── Document Root salah
│     ├── file belum di-upload atau salah folder
│     └── SPA tanpa .htaccess rewrite
├── 500 Internal Server Error
│     → baca error_log di cPanel File Manager
│     → biasanya syntax PHP / .htaccess salah / permission
└── 503 Service Unavailable
      → Node.js app mati. Setup Node.js App → Restart
      → cek stderr.log
```

---

## G. Webhook tidak masuk

```text
payment sukses tapi akses tidak aktif
├── provider tidak mengirim webhook
│     → cek dashboard provider: ada log delivery?
├── webhook terkirim tapi gagal (4xx/5xx)
│     ├── 401/403 → signature verification gagal
│     │     → cek raw body dipakai, bukan parsed JSON
│     │     → cek secret benar (sandbox vs production)
│     ├── 404 → URL webhook salah di dashboard
│     ├── 500 → error di kode handler, baca log server
│     └── timeout → handler terlalu lambat, balas 2xx dulu
├── webhook masuk 200 tapi DB tidak berubah
│     ├── amount/externalId tidak cocok → order tidak ketemu
│     ├── transaksi DB gagal diam-diam → cek error handling
│     └── idempotency terlalu ketat → dianggap duplikat
└── akses aktif dua kali / order dobel
      → idempotency belum ada. tambahkan unique constraint
```

---

## H. DNS / SSL gagal

```text
masalah DNS atau SSL
├── domain baru, belum bisa diakses sama sekali
│     → propagasi. tunggu, cek nslookup domain 8.8.8.8
├── www jalan, root tidak (atau sebaliknya)
│     → kurang satu record (A untuk @, CNAME untuk www)
├── AutoSSL gagal issue
│     ├── DNS belum mengarah ke hosting
│     ├── Cloudflare proxy ON saat validasi → set DNS only sementara
│     └── .htaccess memblokir /.well-known/
├── "Too many redirects"
│     → Cloudflare SSL mode Flexible + force HTTPS di server
│     → ganti ke Full (Strict)
├── sertifikat expired
│     → auto-renew gagal. jalankan manual, cek cron certbot
└── mixed content warning
      → ada asset http:// di halaman https://
      → ganti semua URL asset ke https:// atau relative
```

---

## Cara Pakai Decision Tree

1. Identifikasi **gejala**, bukan tebakan penyebab.
2. Masuk ke pohon yang cocok.
3. Turun sampai ketemu cabang yang sesuai.
4. Kalau tidak ada cabang cocok → kumpulkan bukti, buka `BANTUAN.md`.
5. Jangan lompat cabang. Satu penyebab bisa menyerupai penyebab lain.
