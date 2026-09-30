# G6 — Custom Domain dan SSL: Railway + Cloudflare

> 🔵 MENENGAH · Target waktu: 30–90 menit (propagasi DNS dapat lebih lama)

## Tujuan

Aplikasi tersedia di `https://app.domainmu.com` melalui Railway, DNS Cloudflare, dan sertifikat SSL valid.

## Prasyarat

- Domain aktif dan nameserver sudah memakai Cloudflare.
- URL Railway `*.up.railway.app` sehat.
- Akses ke Railway dan Cloudflare.

## Pilihan Domain

| Pilihan | Contoh | Rekomendasi |
|---|---|---|
| Subdomain | `app.domainmu.com` | Paling aman; mudah dipindah |
| Apex/root | `domainmu.com` | Cocok bila aplikasi adalah situs utama |
| `www` | `www.domainmu.com` | Gunakan redirect konsisten ke satu host utama |

Panduan memakai `app.domainmu.com`.

## Langkah 1 — Daftarkan Domain di Railway

1. Railway → service aplikasi → **Settings → Networking**.
2. Pilih **Custom Domain**.
3. Masukkan `app.domainmu.com`.
4. Salin target CNAME yang ditampilkan Railway, misalnya `xxxx.up.railway.app`.

**Hasil yang diharapkan:** Railway menampilkan status menunggu konfigurasi DNS dan target CNAME.

Jangan menebak target. Salin nilai yang Railway tampilkan untuk servicemu.

## Langkah 2 — Buat DNS Record di Cloudflare

Cloudflare → domain → **DNS → Records → Add record**:

| Field | Nilai |
|---|---|
| Type | `CNAME` |
| Name | `app` |
| Target | target dari Railway |
| Proxy status | **DNS only** (awan abu-abu) saat verifikasi awal |
| TTL | Auto |

Simpan.

**Hasil yang diharapkan:** record `app` terlihat dan menunjuk target Railway.

## Langkah 3 — Verifikasi DNS Sebelum Mengubah Proxy

```bash
nslookup app.domainmu.com
```

Atau:

```bash
dig CNAME app.domainmu.com +short
```

**Output yang diharapkan:** target Railway muncul. Propagasi dapat memakan waktu; jangan berulang kali menghapus/membuat record.

## Langkah 4 — Tunggu SSL Railway Aktif

Kembali ke Railway. Tunggu custom domain berstatus aktif, lalu:

```bash
curl -I https://app.domainmu.com
```

**Output yang diharapkan:** koneksi TLS valid dan respons HTTP 200/3xx.

Buka browser dan cek ikon koneksi aman. Sertifikat harus mencantumkan hostname yang benar.

## Langkah 5 — Aktifkan Proxy Cloudflare (Opsional)

Setelah HTTPS langsung ke Railway terbukti valid, proxy dapat diubah ke **Proxied** (awan oranye) untuk proteksi dan cache Cloudflare.

Cloudflare → **SSL/TLS → Overview**: pilih **Full (strict)**.

Jangan gunakan **Flexible**. Mode itu mengenkripsi browser ke Cloudflare tetapi tidak memastikan koneksi aman ke origin.

Uji ulang:

```bash
curl -I https://app.domainmu.com
curl -i https://app.domainmu.com/api/health
```

**Output yang diharapkan:** HTTPS valid; health 200.

## Langkah 6 — Perbarui URL Aplikasi

Jika aplikasi punya URL publik/callback, tambah di Railway Variables:

```dotenv
NEXT_PUBLIC_APP_URL=https://app.domainmu.com
```

Perbarui callback auth, CORS, webhook, dan link email bila digunakan. Redeploy, lalu tes ulang login dan form.

## Urutan Aman Cutover Domain Aktif

1. Turunkan TTL record lama bila masih ada.
2. Pastikan URL Railway sehat.
3. Tambah custom domain di Railway.
4. Buat CNAME Cloudflare sebagai **DNS only**.
5. Verifikasi DNS dan HTTPS.
6. Baru aktifkan proxy Cloudflare bila diperlukan.
7. Pertahankan record lama sampai pengujian selesai; jangan menghapus rollback path lebih awal.

## Error Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Railway terus “Waiting for DNS” | CNAME salah / masih diproksi | Cocokkan target; DNS only saat verifikasi |
| `ERR_TOO_MANY_REDIRECTS` | Mode SSL Flexible | Ubah ke Full (strict) |
| Sertifikat tidak valid | DNS belum propagasi / hostname salah | Tunggu; cek CNAME dan domain Railway |
| 404 dari Railway | Domain ditempel ke service salah | Pindahkan domain ke service aplikasi |
| Domain benar, auth gagal | Callback masih URL lama | Perbarui app URL/callback lalu redeploy |

## Checklist

- [ ] Custom domain terdaftar pada service aplikasi
- [ ] CNAME Cloudflare sama persis dengan target Railway
- [ ] DNS terverifikasi sebelum proxy aktif
- [ ] `https://app.domainmu.com` dapat dibuka
- [ ] Sertifikat valid untuk hostname
- [ ] Cloudflare SSL mode Full (strict)
- [ ] `/api/health` memberi 200 lewat domain
- [ ] Callback/auth/CORS memakai domain baru
- [ ] Rollback path tersedia sampai pengujian selesai

➡️ Lanjut ke **[07-maintenance.md](07-maintenance.md)**.
