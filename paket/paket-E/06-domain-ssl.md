# 06 — Domain, DNS Cloudflare, dan SSL

## Tujuan

Domain mengarah ke VPS melalui Cloudflare, koneksi HTTPS Full (strict), dan staging terpisah dari produksi.

## Prasyarat

- Bab `05-deploy.md` selesai dan aplikasi berjalan di Coolify.
- Domain sudah dibeli.

> Rujukan: [`../../docs/12-domain-dns-cloudflare/01-domain-dns.md`](../../docs/12-domain-dns-cloudflare/01-domain-dns.md)

## 1. Pindahkan Domain ke Cloudflare

1. Login ke Cloudflare → **Add a Site** → masukkan domain.
2. Cloudflare memberi dua nameserver (misalnya `anita.ns.cloudflare.com` dan `chad.ns.cloudflare.com`).
3. Buka registrar tempat kamu membeli domain.
4. Ganti nameserver registrar menjadi nameserver Cloudflare.
5. Kembali ke Cloudflare dan tunggu status menjadi **Active**.

Propagasi bisa selesai dalam hitungan menit, tetapi kadang sampai 24–48 jam. Jangan mengubah apa pun selama menunggu.

## 2. Buat DNS Record

Di Cloudflare → DNS → Records, tambahkan:

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | `@` | `IP_VPS` | Proxied (awan oranye) |
| A | `www` | `IP_VPS` | Proxied |
| A | `staging` | `IP_VPS` | Proxied |
| A | `panel` | `IP_VPS` | DNS only (abu-abu) |

Catatan:

- `panel` diarahkan DNS only karena Coolify mengelola TLS-nya sendiri; proxy Cloudflare dapat mengganggu.
- Untuk troubleshooting awal, DNS only pada semua record dapat mempermudah diagnosis SSL. Aktifkan proxy kembali setelah yakin HTTPS origin bekerja.

## 3. Atur Domain di Coolify

1. Buka resource aplikasi → tab pengaturan domain.
2. Masukkan `domain-kamu.com` sebagai domain utama.
3. Aktifkan SSL otomatis (Let's Encrypt).
4. Deploy ulang agar Coolify mengaktifkan sertifikat.

Coolify membuat sertifikat via Let's Encrypt. Agar validasi berhasil, record DNS harus sudah mengarah ke IP VPS dan port 80 serta 443 terbuka di firewall.

## 4. Atur SSL/TLS di Cloudflare

Cloudflare → SSL/TLS → Overview → pilih mode **Full (strict)**.

| Mode | Arti | Kapan pakai |
|---|---|---|
| Flexible | Cloudflare ke origin memakai HTTP | Jangan pakai untuk produksi |
| Full | Cloudflare menerima sertifikat apa pun | Sementara saat troubleshoot |
| **Full (strict)** | Cloudflare mewajibkan sertifikat valid di origin | **Pakai ini** |

Aktifkan juga:

- **Always Use HTTPS** — redirect HTTP ke HTTPS.
- **Automatic HTTPS Rewrites** — perbaiki mixed content.

Jangan aktifkan HSTS sebelum seluruh subdomain (termasuk staging, panel) sudah HTTPS. HSTS sulit dibalik setelah browser menyimpannya.

## 5. Verifikasi

Dari komputer lokal:

```bash
curl -I https://domain-kamu.com
curl -I https://staging.domain-kamu.com
```

**Hasil yang diharapkan:** HTTP 200 atau redirect ke halaman yang benar; header berisi `cf-ray` (artinya melewati Cloudflare); sertifikat valid.

Buka di browser dan periksa gembok:

- Klik gembok → sertifikat harus menunjukkan Cloudflare (edge) dan origin (Let's Encrypt).
- Buka dari jaringan seluler untuk menghilangkan cache DNS lokal.

## 6. Pisahkan Staging dan Produksi

| Lingkungan | Domain | Branch Coolify | Database |
|---|---|---|---|
| Produksi | `domain-kamu.com` | `main` | PostgreSQL produksi |
| Staging | `staging.domain-kamu.com` | `develop` | PostgreSQL staging terpisah |

Jangan sambungkan staging ke database produksi. Buat resource PostgreSQL kedua di Coolify khusus staging.

## 7. Email DNS (Bila Kirim Email)

Jika aplikasi mengirim email transaksional dari domain ini (misalnya via Resend, Postmark, atau SMTP sendiri), pasang record dari provider email:

- MX
- SPF (TXT)
- DKIM (TXT/CNAME)
- DMARC (TXT)

Tanpa record ini, email transaksi akan masuk spam atau ditolak.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| `ERR_SSL_PROTOCOL_ERROR` | SSL mode Flexible tapi origin HTTPS | Ganti ke Full (strict) |
| `ERR_TOO_MANY_REDIRECTS` | Redirect loop antara Cloudflare dan Coolify | Pastikan mode Full (strict) dan Coolify tidak memaksa redirect sendiri |
| Sertifikat expired | Let's Encrypt gagal renew | Cek log Coolify, pastikan port 80 terbuka, DNS benar |
| `DNS_PROBE_FINISHED_NXDOMAIN` | Nameserver belum berpindah | Tunggu propagasi atau cek nameserver di registrar |
| Email masuk spam | SPF/DKIM/DMARC kurang | Pasang record dari provider email |

## Checklist

- [ ] Nameserver Cloudflare statusnya Active.
- [ ] Record A/CNAME mengarah ke IP VPS.
- [ ] Domain utama dan staging memiliki record terpisah.
- [ ] Coolify menerbitkan sertifikat Let's Encrypt untuk domain.
- [ ] Cloudflare SSL/TLS mode **Full (strict)**.
- [ ] Always Use HTTPS aktif.
- [ ] `curl -I https://domain` mengembalikan 200 dan `cf-ray`.
- [ ] Staging memakai database sendiri, bukan database produksi.
- [ ] SPF/DKIM/DMARC terpasang bila kirim email.
