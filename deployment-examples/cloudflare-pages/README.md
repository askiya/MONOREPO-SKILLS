# Deploy Static Site ke Cloudflare Pages

Cocok: HTML/CSS/JS, Astro, Vite, Next.js static export. Bukan untuk Node server.

1. Push repo ke GitHub.
2. Cloudflare Dashboard → Workers & Pages → Create → Pages → Connect to Git.
3. Pilih repo.
4. Build command:
   - Vite: `npm run build`, output `dist`
   - Next static: `npm run build`, output `out`
   - HTML biasa: kosong, output `/`
5. Tambahkan environment variables jika ada (jangan taruh secret di frontend).
6. Save and Deploy.

Custom domain: Pages project → Custom domains → Set up a custom domain.
Cloudflare membuat DNS otomatis kalau domain ada di akun yang sama.

Verifikasi: buka URL `.pages.dev` dari HP. Cek semua route, refresh di sub-route.
