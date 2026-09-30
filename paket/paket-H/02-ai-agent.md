# H2 — Setup Hermes Agent untuk Monorepo

> 🟠 LANJUTAN · Target waktu: 45–60 menit · Jalankan dari root monorepo

## Tujuan

Hermes Agent bekerja dari root, membaca aturan lintas workspace, dan memverifikasi dampak perubahan tanpa merusak app lain.

Dokumentasi Hermes berubah. Gunakan referensi resmi terbaru: <https://hermes-agent.nousresearch.com/docs>.

## Langkah 1 — Buka Terminal di Root

```bash
cd santriverse
pwd
```

**Output yang diharapkan:** path berakhir `santriverse`, bukan `apps/web` atau `apps/api`.

Cek dokumen:

```bash
ls AGENTS.md docs package.json turbo.json
```

Jika `turbo.json` belum dibuat, itu wajar sampai langkah build; `AGENTS.md` dan `docs` harus sudah ada.

## Langkah 2 — Instal/Periksa Hermes Agent

Ikuti installer sesuai OS pada dokumentasi resmi. Setelah tersedia:

```bash
hermes --version
hermes --help
```

**Output yang diharapkan:** nomor versi dan bantuan CLI. Jangan menebak flag bila versi lokal berbeda; ikuti `hermes --help`.

Jalankan Hermes dari root melalui perintah chat/interaktif yang tercantum pada versi terpasang.

## Langkah 3 — Uji Root Awareness

Prompt pertama:

```text
Baca AGENTS.md dan seluruh docs/. Jangan ubah file.
Jelaskan struktur monorepo, arah dependency, ownership workspace,
kontrak API, environment variable tiap app, dan urutan deploy.
Sebutkan file yang menjadi sumber tiap jawaban.
```

**Hasil yang diharapkan:** Hermes mengenali `apps/web`, `apps/api`, `packages/shared`; tidak menganggap repo sebagai satu Next.js app.

## Langkah 4 — Pakai Prompt dengan Scope Eksplisit

Template pekerjaan:

```text
Kerjakan hanya task H05.
Scope utama: packages/shared.
Consumer yang wajib diperiksa: apps/web dan apps/api.
Baca AGENTS.md dan docs/04-ARCHITECTURE.md dahulu.
Jangan ubah workspace di luar scope kecuali perlu untuk menjaga build;
jika perlu, jelaskan sebelum mengubah.
Verifikasi: test shared, typecheck dua consumer, lalu turbo build dari root.
Laporkan file berubah dan output perintah nyata.
```

Untuk task app tunggal:

```text
Kerjakan hanya task H06 di apps/api.
packages/shared boleh dibaca, jangan diubah.
apps/web jangan diubah.
Jalankan test apps/api dan turbo build terfilter.
```

## Langkah 5 — Review Dampak Sebelum Eksekusi

Untuk perubahan shared, gunakan dua tahap:

```text
Analisis dampak perubahan OrderInputSchema. Jangan edit file.
Daftar consumer, potensi breaking change, urutan perubahan, dan test wajib.
```

Setelah rencana benar:

```text
Kerjakan rencana yang disetujui. Jaga backward compatibility.
Berhenti bila perubahan memerlukan API breaking change yang tidak ada di dokumen.
```

## Langkah 6 — Izinkan Perintah, Bukan Secret

Hermes boleh menjalankan lint/test/build. Jangan pernah memberinya secret produksi di chat. Gunakan `.env.example` dengan placeholder dan platform dashboard untuk nilai nyata.

Perintah aman umum:

```bash
npm install
npx turbo run lint test build
npm --workspace apps/api test
npm --workspace apps/web run build
```

Tinjau perintah destruktif (hapus folder, reset DB, force push) secara manual. Jangan izinkan bila tidak diperlukan.

## Pola Delegasi

| Pekerjaan | Scope | Verifikasi minimum |
|---|---|---|
| UI | `apps/web` | lint + build web |
| Route API | `apps/api` | test API + build API |
| Schema | `packages/shared` + consumer | test shared + typecheck/build web & API |
| Root config | seluruh repo | `turbo run lint test build` |
| Deploy config | app target | build app + baca platform settings |

## Error Umum

| Gejala | Penyebab | Solusi |
|---|---|---|
| Hermes tidak melihat app lain | Dijalankan dari subfolder | Keluar; jalankan dari root |
| Mengubah semua workspace | Scope prompt tidak jelas | Tulis scope dan consumer eksplisit |
| Shared mengimpor API | Agent mengejar jalan cepat | Tegakkan dependency rule |
| Klaim sukses tanpa root build | Hanya test app | Minta output turbo dari root |
| Memakai command Hermes lama | Dokumentasi/versi berubah | Cek docs resmi + `hermes --help` |

## Checklist

- [ ] Hermes CLI tersedia dan versi terbaca
- [ ] Hermes dijalankan dari root monorepo
- [ ] Agent membaca AGENTS.md dan docs
- [ ] Agent menjelaskan arah dependency dengan benar
- [ ] Prompt task menyebut scope dan consumer
- [ ] Secret produksi tidak masuk chat/source
- [ ] Perubahan shared selalu diverifikasi pada dua consumer

➡️ Lanjut ke **[03-build.md](03-build.md)**.
