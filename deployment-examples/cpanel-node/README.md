# Node.js di cPanel

Prasyarat: cPanel punya menu **Setup Node.js App**. Kalau tidak ada, jangan paksa.
Pakai Vercel/VPS.

1. Upload `server.js` ke folder di luar `public_html`, misal `/home/USER/app/`.
2. Setup Node.js App → Create Application.
3. Isi:
   - Node version: LTS
   - Application mode: Production
   - Application root: `app`
   - Application URL: domain/subdomain target
   - Startup file: `server.js`
4. Jangan isi `PORT` manual — cPanel/Passenger memberi port.
5. Save → Restart Application.
6. Buka `/health`.

Hasil:
```json
{"status":"ok","time":"..."}
```

Ini contoh server stdlib tanpa dependency. Untuk app nyata, upload `package.json`,
jalankan NPM Install dari panel, lalu restart.
