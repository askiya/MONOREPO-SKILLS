# Deploy Static ke cPanel

1. Build di lokal: `npm run build`.
2. Zip **isi** folder output (`dist/` atau `out/`), bukan folder luarnya.
3. cPanel → File Manager → `public_html` → Upload zip → Extract.
4. Copy `.htaccess` ini ke `public_html`.
5. Pastikan `index.html` langsung di `public_html/index.html`.
6. cPanel → SSL/TLS Status → Run AutoSSL.

Permission: folder `755`, file `644`.

Untuk HTML multi-page biasa, hapus bagian "SPA fallback" di `.htaccess`.

Verifikasi:
```bash
curl -I https://DOMAINMU.com
curl -I http://DOMAINMU.com
```
HTTPS harus 200, HTTP harus 301 ke HTTPS.
