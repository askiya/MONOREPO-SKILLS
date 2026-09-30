# 01 — Troubleshooting Error Umum

## Prinsip

Jangan kirim ke agent "masih error". Ambil bukti mentah: command, output penuh,
langkah reproduksi, expected vs actual, file terkait.

## `npm install` Gagal

1. Cek versi: `node -v && npm -v`.
2. Pastikan versi sesuai `package.json`/`.nvmrc`.
3. Jangan langsung hapus lockfile.
4. Baca baris error pertama yang relevan, bukan cuma baris terakhir.
5. Kalau peer dependency, cari package mana yang konflik.

Prompt:
```text
Analisis error npm berikut. Jangan mengubah dependency dulu. Baca package.json
dan lockfile, identifikasi root cause, lalu usulkan diff minimum.
<tempel output penuh>
```

## `npm run build` Gagal

- TypeScript: buka file+line dari error, jangan ubah ke `any` sebagai obat.
- Module not found: cek path, kapitalisasi, dependency terpasang.
- Env missing: tambahkan ke dashboard deploy dan `.env.example`, bukan kode.
- Build kehabisan RAM: tambah swap/build di platform lain; jangan sembarang
  mematikan typecheck.

## Localhost Tidak Bisa Dibuka

```bash
npm run dev
```

Cek:
- URL dan port dari output,
- proses masih hidup,
- port dipakai proses lain,
- firewall lokal,
- app bind ke `localhost`/`0.0.0.0` sesuai kebutuhan.

## Deploy Build Sukses tapi 500

1. Buka runtime/container logs.
2. Bandingkan env lokal vs deploy (nama, bukan nilai).
3. Cek migrasi DB.
4. Cek koneksi DB/firewall.
5. Cek start command dan port.

## Domain Tidak Akses

```bash
nslookup domain.com
curl -I http://domain.com
curl -I https://domain.com
```

Pisahkan masalah:
- DNS salah: IP hasil lookup bukan server.
- HTTP gagal: proxy/firewall/container.
- HTTPS gagal: sertifikat/mode Cloudflare.
- App 500: runtime log.

## Database Connection Error

- Host/port/user/database benar.
- Password dengan karakter khusus sudah URL-encoded.
- IP server diizinkan provider DB.
- SSL mode sesuai provider.
- Connection pool sesuai serverless/runtime.

Jangan print `DATABASE_URL` utuh ke chat/log.

## Agent Loop Salah Tiga Kali

Berhenti. Lakukan:

1. `git diff` — pahami apa yang berubah.
2. Revert perubahan gagal ke checkpoint aman.
3. Pecah masalah menjadi reproduksi kecil.
4. Kumpulkan error mentah.
5. Mulai chat agent baru dengan konteks bersih.

## Format Laporan

```text
TUJUAN:
COMMAND:
OUTPUT PENUH:
LANGKAH REPRODUKSI:
EXPECTED:
ACTUAL:
FILE TERKAIT:
PERUBAHAN TERAKHIR:
```

## Checklist

- [ ] Error mentah dikumpulkan
- [ ] Masalah dipisah per layer
- [ ] Secret disensor
- [ ] Root cause diperbaiki, bukan error disembunyikan
