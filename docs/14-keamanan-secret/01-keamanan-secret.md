# 01 — Keamanan Secret dan Environment

## Tujuan

Secret tidak bocor ke GitHub, browser, log, atau screenshot.

## Apa yang Termasuk Secret

- API key/provider key
- Database URL + password
- JWT/session secret
- OAuth client secret
- Webhook signing secret
- SSH private key
- Cookie/session file
- Service account JSON

## Aturan Penyimpanan

| Lingkungan | Simpan di |
|---|---|
| Lokal | `.env` (masuk `.gitignore`) |
| Vercel/Netlify/Render | dashboard Environment Variables |
| Coolify | environment variables resource |
| Docker manual | `.env` di server, permission ketat |
| GitHub Actions | GitHub Actions Secrets |

`.env.example` boleh masuk Git, tapi hanya nama + placeholder:

```env
DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/DATABASE
NEXTAUTH_SECRET=<GENERATE_RANDOM_32_BYTES>
XENDIT_SECRET_KEY=<FROM_PROVIDER_DASHBOARD>
```

## Client vs Server

Variabel berprefix `NEXT_PUBLIC_` terkirim ke browser. Jangan pernah menaruh
secret di variabel publik. API key server hanya dibaca route/server code.

## Generate Secret

```bash
openssl rand -base64 32
```

Jangan pakai contoh dari tutorial untuk produksi.

## Sebelum Commit

```bash
git status
git diff --cached
```

Cari `.env`, key, token, password, private key, dump DB, file sesi.

## Kalau Secret Terlanjur Ter-push

1. Anggap bocor — jangan debat "repo private".
2. Cabut/rotate secret di provider segera.
3. Hapus file dari index Git.
4. Bersihkan history bila perlu (git-filter-repo/BFG).
5. Paksa kolaborator re-clone setelah history rewrite.
6. Cek log provider untuk penyalahgunaan.

Menghapus file lalu commit tidak menghapus secret dari history.

## Security Minimum Produksi

- HTTPS wajib.
- Dependency lockfile masuk Git.
- Jalankan audit dependency, nilai temuan secara manual.
- Security headers sesuai aplikasi.
- Rate limit auth/webhook.
- Backup terenkripsi + restore test.
- Akses produksi least privilege.
- Jangan log password/token/body payment sensitif.

## Checklist

- [ ] `.env*` diabaikan kecuali `.env.example`
- [ ] Secret hanya environment/server
- [ ] Tidak ada `NEXT_PUBLIC_` untuk secret
- [ ] `git diff --cached` diperiksa
- [ ] Prosedur rotate dipahami
