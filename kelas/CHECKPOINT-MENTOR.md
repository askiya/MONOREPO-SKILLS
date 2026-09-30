# Checkpoint Mentor

Empat titik setoran wajib sebelum lanjut ke fase berikutnya.
Mentor memeriksa; member yang belum lolos tidak melanjutkan.

---

## Checkpoint 1 — PRD Review

**Kapan:** setelah Level 2 PROGRESS.md selesai (semua dokumen terisi).

### Yang Disetor Member

- [ ] `PRD.md` (atau `01-PRD.md`)
- [ ] `SDLC.md`
- [ ] `DESIGN.md`
- [ ] `ARCHITECTURE.md`
- [ ] `TASKS.md`
- [ ] `AGENTS.md`
- [ ] Bukti: screenshot agent merangkum dokumen tanpa salah

### Yang Diperiksa Mentor

- [ ] Setiap fitur MVP punya alasan "kenapa" dan "untuk siapa"
- [ ] Out of scope ditulis eksplisit
- [ ] Stack dan alasan ditulis, bukan copy-paste tanpa konteks
- [ ] DESIGN.md punya token warna/font/spacing
- [ ] TASKS.md punya definisi selesai per task
- [ ] AGENTS.md punya aturan keras
- [ ] Tidak ada kontradiksi antar-dokumen

### Kriteria Lolos

Semua centang terisi. Dokumen yang "isinya nanti" tidak lolos.

---

## Checkpoint 2 — Localhost

**Kapan:** setelah Level 3 PROGRESS.md selesai (app jalan lokal).

### Yang Disetor Member

- [ ] URL repo GitHub (private OK, undang mentor)
- [ ] Screenshot halaman utama desktop (1280px)
- [ ] Screenshot halaman katalog/fitur utama mobile (375px)
- [ ] Screenshot halaman login dengan error feedback
- [ ] Output `npm run lint` (0 error)
- [ ] Output `npm run build` (sukses)
- [ ] Output `npm run test` (hijau)
- [ ] Bukti `git status` bersih (tidak ada file liar)

### Yang Diperiksa Mentor

- [ ] App jalan tanpa error merah di console
- [ ] UI mengikuti DESIGN.md (warna, font, spacing)
- [ ] Loading / empty / error state ada di halaman data
- [ ] Tidak ada tombol/kontrol mati
- [ ] Register + login berfungsi
- [ ] Route member terproteksi
- [ ] Input divalidasi server, bukan hanya client
- [ ] `.env` tidak ada di repo

### Kriteria Lolos

Build hijau + app bisa dicoba mentor di mesin sendiri setelah clone + setup .env.

---

## Checkpoint 3 — Staging

**Kapan:** setelah deploy staging berhasil.

### Yang Disetor Member

- [ ] URL staging publik
- [ ] Akun uji (email/password khusus tes, bukan milik asli)
- [ ] Daftar environment variable (nama saja, tanpa nilai)
- [ ] QA checklist `checklists/qa-manual.md` sudah dicentang sendiri
- [ ] Feedback dari minimal 3 tester (bisa screenshot / catatan)

### Yang Diperiksa Mentor

- [ ] URL bisa dibuka dari jaringan luar (bukan cuma lokal)
- [ ] Register → login → alur utama jalan
- [ ] Error tidak membocorkan stack trace
- [ ] Database staging terpisah dari yang nanti produksi
- [ ] Secret tidak terbaca di build log / client bundle
- [ ] Feedback tester ditindaklanjuti (minimal dicatat kalau belum diperbaiki)

### Kriteria Lolos

URL staging bisa dipakai mentor dari HP tanpa bantuan member.

---

## Checkpoint 4 — Produksi

**Kapan:** setelah deploy produksi + domain.

### Yang Disetor Member

- [ ] URL domain produksi
- [ ] Screenshot gembok HTTPS valid di browser
- [ ] Bukti `curl -I https://domain` → 200
- [ ] Bukti `curl -I http://domain` → 301 redirect ke https
- [ ] Bukti backup DB: file + tanggal
- [ ] Bukti restore drill: screenshot restore di DB uji berhasil
- [ ] Bukti payment sandbox end-to-end (kalau ada payment)
- [ ] Output monitoring uptime (alert aktif)

### Yang Diperiksa Mentor

- [ ] Domain + HTTPS valid
- [ ] HTTP redirect ke HTTPS
- [ ] Alur kritis berfungsi di produksi
- [ ] DB produksi terpisah dari staging
- [ ] Secret produksi berbeda dari staging
- [ ] Backup terjadwal aktif
- [ ] Restore pernah diuji nyata
- [ ] Payment webhook idempotent (kalau ada)
- [ ] Privacy policy / terms ada (kalau jualan)

### Kriteria Lolos

Produk bisa dipakai orang sungguhan. Backup dan restore terbukti jalan.

---

## Catatan untuk Mentor

- Jangan loloskan checkpoint hanya karena member "kayaknya sudah paham".
  Minta bukti: screenshot, output, URL.
- Checkpoint boleh direvisi dan disetorkan ulang.
- Satu kegagalan bukan akhir — tulis feedback spesifik, minta perbaikan.
- Jangan periksa secret; minta member sensor.
