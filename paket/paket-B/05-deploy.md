# 05 — Deploy ke Cloudflare Pages

## Tujuan

Website live di URL `.pages.dev` dengan HTTPS aktif, deploy otomatis setiap
push ke `main`, dan custom domain terpasang jika tersedia.

## Sebelum Mulai

- `npm run build` sukses dan `dist/` berisi `index.html` ([`04-preview.md`](04-preview.md))
- Akun GitHub dan Cloudflare siap

Referensi:
- [`../../deployment-examples/cloudflare-pages/README.md`](../../deployment-examples/cloudflare-pages/README.md)
- [`../../docs/09-deploy-gratis/01-deploy-staging.md`](../../docs/09-deploy-gratis/01-deploy-staging.md)
- [`../../docs/12-domain-dns-cloudflare/01-domain-dns.md`](../../docs/12-domain-dns-cloudflare/01-domain-dns.md)

---

## Langkah 1 — Pastikan repo bersih

Cek file yang akan ter-push:

```bash
git status
```

Pastikan `node_modules` dan `dist` tidak ikut ter-commit:

```bash
grep -q "^node_modules$" .gitignore || echo "node_modules" >> .gitignore
grep -q "^dist$" .gitignore || echo "dist" >> .gitignore
grep -q "^.env$" .gitignore || echo ".env" >> .gitignore
```

Verifikasi:

```bash
git check-ignore -v node_modules dist
```

**Expected output:** kedua path disebut bersama baris `.gitignore`. Jika kosong,
aturan ignore belum berlaku — perbaiki sebelum push.

---

## Langkah 2 — Push ke GitHub

```bash
git add .
git commit -m "chore: siap deploy cloudflare pages"
git branch -M main
git remote add origin https://github.com/<USERNAME>/<REPO>.git
git push -u origin main
```

Jika remote sudah ada, cukup `git push`.

Panduan Git dasar:
[`../../docs/01-setup-ai-agent/04-git-github-dasar.md`](../../docs/01-setup-ai-agent/04-git-github-dasar.md)

---

## Langkah 3 — Hubungkan repo ke Cloudflare Pages

1. Buka [dash.cloudflare.com](https://dash.cloudflare.com)
2. **Workers & Pages** → **Create** → tab **Pages** → **Connect to Git**
3. Authorize GitHub dan pilih repository project
4. Pilih production branch: `main`

> Nama menu dashboard dapat berubah. Yang dicari: pembuatan project Pages dari
> repository Git.

---

## Langkah 4 — Build settings

| Field | Astro | Vite |
|---|---|---|
| Framework preset | Astro | Vite |
| Build command | `npm run build` | `npm run build` |
| Build output directory | `dist` | `dist` |
| Root directory | kosong (root repo) | kosong (root repo) |
| Node version | set `NODE_VERSION` = `20` bila build gagal karena versi | sama |

Environment variables: kosongkan untuk Paket B.

> Website statis mengirim seluruh isi build ke browser. Nilai apa pun yang
> ditaruh di sini bisa terbaca pengunjung, jadi jangan masukkan secret.

Klik **Save and Deploy**.

---

## Langkah 5 — Verifikasi deploy

Tunggu build selesai. Status harus **Success** dan Cloudflare memberi URL
`https://<nama-project>.pages.dev`.

Cek dari terminal:

```bash
curl -I https://<nama-project>.pages.dev
```

**Expected output:** baris pertama berstatus `200`.

Uji manual:

| Uji | Lolos kalau |
|---|---|
| Beranda | Tampil lengkap, gaya dan gambar muncul |
| Semua halaman | Terbuka, tidak 404 |
| Refresh di sub-route | Tetap tampil |
| CTA utama | WhatsApp/email terbuka |
| Buka dari HP | Layout rapi, tanpa horizontal scroll |
| Console browser | Tanpa error merah |

---

## Langkah 6 — Deploy otomatis

Setelah project terhubung:

- Push ke `main` → deploy produksi
- Push branch lain atau buka PR → preview deployment dengan URL sendiri

Uji sekali:

```bash
git commit --allow-empty -m "chore: uji auto deploy"
git push
```

**Expected output:** deployment baru muncul di dashboard Pages tanpa aksi manual.

---

## Langkah 7 — Custom domain

Lewati langkah ini jika belum punya domain; URL `.pages.dev` sudah HTTPS.

Urutan langkah ini penting. Ikuti berurutan agar website tidak tidak dapat diakses saat perpindahan.

1. Pastikan domain sudah ada di akun Cloudflare yang sama dengan project Pages.
   Jika belum, tambahkan domain lewat **Add a Site**, lalu ubah nameserver di
   registrar ke nameserver yang diberikan Cloudflare.
2. Tunggu domain berstatus aktif di Cloudflare. Verifikasi:

   ```bash
   dig NS namadomain.com +short
   ```

   Nameserver Cloudflare harus muncul sebelum lanjut.
3. Buka project Pages → **Custom domains** → **Set up a custom domain**.
4. Masukkan domain, misalnya `namadomain.com`, lalu konfirmasi.
   Karena domain berada di akun yang sama, Cloudflare membuat DNS record yang
   dibutuhkan secara otomatis.
5. Ulangi untuk `www.namadomain.com` jika ingin kedua alamat bekerja.
6. Tunggu status domain menjadi **Active** dan sertifikat selesai diterbitkan.
   Proses sertifikat bisa memakan waktu beberapa menit.
7. Verifikasi sebelum menyebarkan alamat baru:

   ```bash
   dig namadomain.com +short
   curl -I https://namadomain.com
   ```

   Domain harus resolve dan status HTTP harus `200`.
8. Buka domain di browser, klik ikon gembok, dan pastikan sertifikat valid serta
   belum kedaluwarsa.

### SSL/TLS

Cloudflare dashboard → **SSL/TLS** → **Overview** → set **Full (Strict)**.

Jangan gunakan mode Flexible: mode tersebut bisa menyebabkan redirect
berulang dan koneksi yang tidak terenkripsi sepenuhnya.

---

## Langkah 8 — Rollback bila deploy rusak

1. Dashboard project Pages → **Deployments**
2. Pilih deployment lama yang statusnya Success
3. Gunakan opsi rollback ke deployment tersebut
4. Verifikasi ulang dengan `curl -I` dan buka di browser

Perbaiki penyebabnya di repo, baru deploy lagi.

---

## Kesalahan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Build gagal: `command not found` | Build command salah | Gunakan `npm run build` sesuai `package.json` |
| Deploy sukses tapi halaman kosong | Output directory salah | Set output ke `dist` |
| CSS dan gambar 404 | Path aset salah saat produksi | Rujuk aset dari folder publik dengan path root |
| Build gagal karena versi Node | Versi Node berbeda dari lokal | Tambahkan variable `NODE_VERSION` |
| Sub-route 404 saat refresh | Halaman tidak ter-generate | Pastikan halaman ada di struktur halaman dan ikut build |
| Custom domain tidak aktif | Domain bukan di akun Cloudflare yang sama atau nameserver belum pindah | Selesaikan Langkah 7 nomor 1–2 lebih dulu |
| Redirect berulang | SSL mode Flexible | Ubah ke Full (Strict) |

---

## Checklist

- [ ] `node_modules`, `dist`, dan `.env` masuk `.gitignore` dan terverifikasi
- [ ] Repo ter-push ke GitHub branch `main`
- [ ] Project Pages terhubung ke repository
- [ ] Build command `npm run build` dan output `dist` diset
- [ ] Tidak ada secret di environment variables
- [ ] Build status **Success**
- [ ] `curl -I https://<nama-project>.pages.dev` → status 200
- [ ] Semua halaman terbuka, refresh sub-route aman
- [ ] Dibuka dari HP nyata, layout rapi
- [ ] Auto-deploy terbukti jalan setelah push
- [ ] Custom domain aktif (jika dipakai) dan `curl -I` → 200
- [ ] Sertifikat HTTPS valid, SSL mode Full (Strict)
- [ ] Cara rollback sudah dicoba atau dipahami
