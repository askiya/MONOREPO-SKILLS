# 06 — Domain dan SSL Frontend/API

## Tujuan

Memisahkan domain frontend dan API, mengaktifkan HTTPS, mengatur CORS, serta memastikan service internal tidak terekspos.

## Prasyarat

Resource Coolify sehat dan dapat diakses internal. Domain dikelola di Cloudflare atau DNS provider lain.

## 1. Rancang Domain

| Hostname | Service | Publik |
|---|---|---|
| `app.example.com` | Next.js | ya |
| `api.example.com` | FastAPI | ya |
| `ollama` | model runtime | tidak |
| `postgres` | DB/vector | tidak |

Pisahkan API agar policy CORS, rate limit, log, dan scaling jelas.

## 2. Buat DNS

Di Cloudflare:

| Tipe | Nama | Nilai | Proxy awal |
|---|---|---|---|
| A | `app` | IP VPS | DNS only |
| A | `api` | IP VPS | DNS only |

Atau gunakan CNAME target yang diberikan Coolify. Cek sumber resmi Coolify untuk mode domain aktif.

```bash
dig +short app.example.com
dig +short api.example.com
```

**Hasil yang diharapkan:** keduanya mengarah ke origin yang benar.

## 3. Pasang Domain di Coolify

- Web: `https://app.example.com`
- API: `https://api.example.com`
- Container port web: `3000`
- Container port API: `8000`
- Health API: `/health`

Coolify/proxy menerbitkan sertifikat otomatis. Jangan expose `11434` atau `5432`.

**Hasil yang diharapkan:** sertifikat aktif dan routing masuk ke service benar.

## 4. Konfigurasi CORS FastAPI

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST"],
    allow_headers=["Authorization", "Content-Type", "X-Tenant-ID"],
)
```

Jangan gunakan `*` bersama credential. Tenant ID dari header bukan bukti auth; server wajib menurunkan tenant dari identitas yang terverifikasi.

## 5. Cookie dan Streaming

Jika auth memakai cookie lintas subdomain:

- `Secure=true`;
- `HttpOnly=true` untuk session;
- `SameSite=Lax` atau `None` hanya bila flow mengharuskan;
- domain cookie dibatasi sekecil mungkin;
- CSRF protection untuk request state-changing.

Untuk SSE/streaming, pastikan proxy timeout cukup dan buffering tidak memutus stream. Test dengan request nyata.

## 6. Verifikasi HTTPS dan CORS

```bash
curl --fail --head https://app.example.com
curl --fail https://api.example.com/health
curl -i -X OPTIONS https://api.example.com/v1/ask \
  -H 'Origin: https://app.example.com' \
  -H 'Access-Control-Request-Method: POST'
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null | openssl x509 -noout -issuer -subject -dates
```

**Hasil yang diharapkan:** HTTPS valid; preflight mengizinkan origin frontend tepat; origin asing tidak mendapat izin.

Setelah origin HTTPS valid, proxy Cloudflare boleh diaktifkan dengan SSL/TLS **Full (strict)**. Atur batas upload/timeouts sesuai produk; jangan mematikan proteksi tanpa pengukuran.

## 7. Rate Limit dan Header

Pasang rate limit terpisah untuk ingest dan ask. Tambahkan header keamanan, batas request body, timeout upstream, dan request ID. Jangan cache respons privat atau prompt pada CDN.

## Masalah Umum

| Gejala | Perbaikan |
|---|---|
| sertifikat gagal | cek DNS, port 80/443, domain mapping Coolify |
| CORS di browser | cocokkan scheme/host; cek preflight dan header |
| stream putus | cek proxy buffering dan timeout |
| redirect loop | Cloudflare Full (strict), bukan Flexible |
| Ollama publik | hapus domain/port mapping dan rotasi bila terpapar |

## Checklist

- [ ] Frontend dan API memakai hostname terpisah.
- [ ] PostgreSQL serta Ollama tidak memiliki domain/port publik.
- [ ] Sertifikat TLS valid pada dua hostname.
- [ ] HTTP dialihkan ke HTTPS.
- [ ] CORS hanya mengizinkan frontend produksi.
- [ ] Auth menentukan tenant; header tenant tidak dipercaya sendiri.
- [ ] Streaming dan upload diuji melalui domain publik.
- [ ] Rate limit, body limit, timeout, dan header keamanan aktif.
