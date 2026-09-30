# 01 — Domain, DNS, dan Cloudflare

## Tujuan

Domain mengarah ke aplikasi, HTTPS aktif, dan staging terpisah dari produksi.

## Rancangan Domain

| Lingkungan | Domain | Branch |
|---|---|---|
| Produksi | `domain.com` | `main` |
| Staging | `staging.domain.com` | `develop` |
| API terpisah | `api.domain.com` | backend production |
| Admin infra | `panel.domain.com` | Coolify/panel |

Jangan pakai database produksi untuk staging.

## Tambah Domain ke Cloudflare

1. Buat akun Cloudflare → Add a Site.
2. Masukkan domain.
3. Cloudflare memberi dua nameserver.
4. Buka registrar tempat membeli domain, ganti nameserver.
5. Tunggu status Active. Propagasi bisa menit sampai 48 jam.

## Record Umum

### VPS

| Type | Name | Content |
|---|---|---|
| A | `@` | `IP_VPS` |
| A | `www` | `IP_VPS` |
| A | `staging` | `IP_VPS_STAGING` atau `IP_VPS` |
| A | `api` | `IP_VPS` |

### Vercel

Ikuti record persis yang diminta Vercel pada menu Domains. Umumnya root memakai
A record dan subdomain memakai CNAME, tetapi nilai dapat berubah.

## Proxy Cloudflare

- Oranye (proxied): website/API publik yang mendukung proxy.
- Abu-abu (DNS only): record yang diminta provider/verifikasi, mail, atau layanan
  yang tidak kompatibel proxy.

Untuk troubleshooting SSL pertama, DNS only dapat mempermudah diagnosis. Setelah
HTTPS origin sehat, aktifkan proxy bila diinginkan.

## SSL/TLS

Cloudflare → SSL/TLS → mode **Full (strict)** setelah origin punya sertifikat
valid. Jangan gunakan Flexible untuk aplikasi produksi; koneksi Cloudflare ke
origin menjadi HTTP.

Aktifkan:
- Always Use HTTPS,
- Automatic HTTPS Rewrites bila perlu,
- HSTS hanya setelah semua subdomain siap HTTPS (sulit dibalik cepat).

## Email DNS

Kalau kirim email domain, pasang record dari provider:
- MX,
- SPF (TXT),
- DKIM (TXT/CNAME),
- DMARC (TXT).

Salah DNS email membuat transaksi masuk spam.

## Verifikasi

```bash
nslookup domain.com
curl -I https://domain.com
```

Cek sertifikat di browser dan uji dari jaringan seluler.

## Checklist

- [ ] Nameserver Cloudflare Active
- [ ] Record A/CNAME benar
- [ ] Produksi dan staging terpisah
- [ ] SSL Full (strict)
- [ ] HTTPS aktif
- [ ] SPF/DKIM/DMARC terpasang bila kirim email
