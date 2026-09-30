# 02 — DNS Record & Arahkan Domain ke Hosting

## Tujuan

Domain mengarah ke hosting cPanel, bisa diakses di browser.

## Skenario

### A. Domain dibeli di tempat yang sama dengan hosting

DNS biasanya sudah otomatis. Cek:

```bash
nslookup domainmu.com
```

Kalau IP yang keluar = IP hosting, sudah benar. Lanjut ke SSL.

### B. Domain beli terpisah (registrar lain)

Dua opsi:

#### Opsi 1: Pakai Nameserver Hosting

1. Login registrar domain.
2. Cari menu **Nameservers** / **DNS Management**.
3. Ganti nameserver ke milik hosting (info ada di email aktivasi), contoh:

```
ns1.providerhosting.com
ns2.providerhosting.com
```

4. Simpan. Tunggu propagasi (menit sampai 48 jam).
5. Verifikasi:

```bash
nslookup -type=NS domainmu.com
```

#### Opsi 2: Pakai Cloudflare (disarankan)

1. Daftar Cloudflare → Add Site → masukkan domain.
2. Cloudflare beri dua nameserver → pasang di registrar.
3. Di Cloudflare, tambah DNS record:

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `@` | `IP_HOSTING` | Proxied ☁️ |
| A | `www` | `IP_HOSTING` | Proxied ☁️ |
| CNAME | `staging` | `domainmu.com` | Proxied ☁️ |

IP hosting ada di cPanel → sidebar kanan → **Shared IP Address**.

4. Tunggu status **Active** di Cloudflare.
5. SSL/TLS → **Full (Strict)** (setelah hosting punya SSL).

### C. Subdomain untuk staging

Di cPanel:
1. **Domains** → **Create a New Domain** (cPanel terbaru) atau **Subdomains**.
2. Subdomain: `staging`, domain: `domainmu.com`.
3. Document Root: `public_html/staging` (atau folder terpisah).

Di Cloudflare / DNS registrar:

```
A  staging  →  IP_HOSTING
```

## Verifikasi DNS

```bash
# Cek A record
nslookup domainmu.com

# Cek dari DNS publik
nslookup domainmu.com 8.8.8.8

# Cek HTTPS
curl -I https://domainmu.com
```

Kalau masih lama, coba bersihkan cache DNS lokal:

```bash
# Windows
ipconfig /flushdns

# macOS
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

## Troubleshooting DNS

| Gejala | Kemungkinan |
|---|---|
| `NXDOMAIN` | Nameserver belum propagasi / salah |
| IP bukan milik hosting | A record salah |
| Website "It works!" default | DNS benar tapi belum upload file |
| Timeout | Firewall hosting / IP salah |
| Redirect loop | Mode SSL Cloudflare salah (Flexible vs Full) |

## Checklist

- [ ] Nameserver atau A record sudah diarahkan
- [ ] `nslookup` menunjukkan IP hosting yang benar
- [ ] Subdomain staging dibuat (kalau perlu)
- [ ] Propagasi selesai, domain bisa diakses
