# 04 — Preview & Testing

## Tujuan

Aplikasi terbukti jalan di localhost, lolos lint, lolos build, dan sudah dites
manual di berbagai ukuran layar. Tidak ada error yang dibawa ke produksi.

## Sebelum Mulai

- Build fase 3 selesai ([`03-build.md`](03-build.md))

Referensi:
- [`../../docs/07-preview-localhost/01-localhost-dan-debug.md`](../../docs/07-preview-localhost/01-localhost-dan-debug.md)
- [`../../docs/08-testing/01-testing-sebelum-deploy.md`](../../docs/08-testing/01-testing-sebelum-deploy.md)

---

## Langkah 1 — Jalankan development server

```bash
npm run dev
```

**Expected output:**
```
  ▲ Next.js 14.x.x
  - Local:        http://localhost:3000
  - Environments: .env

 ✓ Ready in 2.1s
```

Kalau port 3000 terpakai:
```bash
npm run dev -- -p 3001
```

---

## Langkah 2 — Tes manual setiap halaman

Buka setiap route dan cek satu per satu.

| Route | Yang dicek | Lolos kalau |
|---|---|---|
| `/` | Halaman utama tampil | Tidak ada error di console browser |
| `/register` | Form daftar | Bisa submit, user masuk database |
| `/login` | Form login | Login sukses → redirect `/dashboard` |
| `/dashboard` | Halaman terproteksi | Tanpa login → redirect ke `/login` |
| `/route-ngawur` | 404 | Muncul halaman 404, bukan error |

### Cek console browser

Buka DevTools (`F12`) → tab **Console**.

- 0 error merah → lolos
- Ada error merah → catat pesannya, kirim ke agent untuk diperbaiki

### Cek Network

DevTools → tab **Network**:
- Semua request status 200 / 304 / 307
- Tidak ada 500 (server error)
- Tidak ada request ke `localhost` yang di-hardcode

---

## Langkah 3 — Tes responsive

DevTools → toggle device toolbar (`Ctrl+Shift+M`).

Tes di 3 ukuran minimal:

| Ukuran | Lebar | Yang dicek |
|---|---|---|
| Mobile | 375px | Tidak ada horizontal scroll, teks terbaca, tombol bisa ditekan |
| Tablet | 768px | Layout menyesuaikan, tidak ada elemen tumpang tindih |
| Desktop | 1440px | Konten tidak melebar berlebihan, ada max-width |

**Bukti kelulusan:** screenshot di 375px dan 1440px.

---

## Langkah 4 — Lint

```bash
npm run lint
```

**Expected output (lolos):**
```
✔ No ESLint warnings or errors
```

Kalau ada error, kirim ke agent:
```
Jalankan npm run lint, perbaiki semua error dan warning.
Jangan matikan rule ESLint untuk menyembunyikan error —
perbaiki kodenya. Laporkan apa yang kamu ubah.
```

---

## Langkah 5 — Type check

```bash
npx tsc --noEmit
```

**Expected output (lolos):** tidak ada output sama sekali (exit code 0).

Kalau ada error TypeScript, perbaiki dulu. Error type yang dibiarkan akan
menggagalkan build di Vercel.

---

## Langkah 6 — Production build

Ini test paling penting. Build produksi lebih ketat daripada `npm run dev`.

```bash
npm run build
```

**Expected output (lolos):**
```
 ✓ Compiled successfully
 ✓ Linting and checking validity of types
 ✓ Collecting page data
 ✓ Generating static pages
 ✓ Finalizing page optimization

Route (app)                    Size     First Load JS
┌ ○ /                          1.2 kB   89 kB
├ ○ /login                     2.1 kB   91 kB
...
```

**Gagal kalau** muncul:
```
Failed to compile.
```

> Kalau `npm run build` gagal di localhost, deploy di Vercel **pasti** gagal juga.
> Jangan deploy sebelum build lokal hijau.

### Tes hasil build

```bash
npm run start
```

Buka `http://localhost:3000` — ini versi produksi. Tes ulang login dan dashboard.

---

## Langkah 7 — Test otomatis (opsional untuk pemula)

Kalau project punya test:
```bash
npm test
```

Belum ada test? Minta agent buat satu test dasar:
```
Buat satu test untuk API route /api/register menggunakan Vitest:
- Kasus sukses: email baru, password valid → 201
- Kasus gagal: email sudah terdaftar → 409
- Kasus gagal: password kurang dari 8 karakter → 400

Tambahkan script "test" di package.json. Jangan install framework
test selain Vitest.
```

---

## Debug: Ketika Ada Error

Baca decision tree lengkap:
[`../../docs/15-troubleshooting/02-decision-tree.md`](../../docs/15-troubleshooting/02-decision-tree.md)

Alur cepat:

```text
Ada error?
│
├── Error muncul di terminal saat npm run dev
│     → Baca baris pertama error, cari nama file + nomor baris
│     → Kirim error lengkap ke agent (jangan diringkas)
│
├── Error muncul di console browser
│     → Cek apakah Client Component pakai server-only code (Prisma, fs)
│     → Cek apakah env var yang dipakai di client sudah prefix NEXT_PUBLIC_
│
├── Halaman blank putih
│     → Cek console browser, biasanya error rendering
│     → Cek apakah komponen return undefined
│
├── Login gagal terus
│     → Cek NEXTAUTH_SECRET dan NEXTAUTH_URL di .env
│     → Cek password di DB apakah hash bcrypt (mulai dengan $2a$ atau $2b$)
│
└── npm run build gagal tapi npm run dev jalan
      → Hampir selalu error TypeScript atau ESLint
      → Jalankan npx tsc --noEmit untuk lihat detailnya
```

### Cara lapor error ke agent

Format yang benar:
```
Saya dapat error ini saat [aksi apa]:

[PASTE ERROR LENGKAP, TERMASUK STACK TRACE]

File yang terlibat: [nama file]
Yang saya harapkan: [hasil yang benar]
```

Jangan: "errornya gagal build tolong benerin". Agent tidak bisa menebak.

Format lapor bug lengkap: [`../../BANTUAN.md`](../../BANTUAN.md)

---

## Checklist

- [ ] `npm run dev` jalan tanpa error
- [ ] Semua route dites manual, tidak ada 500
- [ ] Console browser 0 error merah
- [ ] Register → user masuk database
- [ ] Login → redirect ke `/dashboard`
- [ ] `/dashboard` tanpa login → redirect ke `/login`
- [ ] Password di database berbentuk hash bcrypt, bukan plaintext
- [ ] Responsive dites di 375px, 768px, 1440px
- [ ] Screenshot 375px dan 1440px disimpan
- [ ] `npm run lint` — 0 error, 0 warning
- [ ] `npx tsc --noEmit` — 0 error
- [ ] `npm run build` — sukses
- [ ] `npm run start` — versi produksi jalan, login masih berfungsi
- [ ] Semua perubahan di-commit
