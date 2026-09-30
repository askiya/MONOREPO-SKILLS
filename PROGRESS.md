# PROGRESS.md — Penanda Posisi Belajar

Copy file ini ke folder project kamu. Centang saat **benar-benar selesai**, bukan
saat "kayaknya sudah".

Nama: `________________`
Jalur (A/B/C/D): `____`
Mulai tanggal: `________`

---

## Level 1 — Siap Coding

- [ ] Node.js terpasang (`node -v` keluar versi)
- [ ] npm jalan (`npm -v` keluar versi)
- [ ] Git terpasang + identitas diset (`git config user.name`)
- [ ] AI agent (Antigravity/lainnya) bisa baca & tulis file
- [ ] AI agent bisa jalankan terminal
- [ ] Akun GitHub aktif
- [ ] Repo pertama berhasil di-push

**Bukti:** screenshot output `node -v && npm -v && git --version` + URL repo GitHub.

---

## Level 2 — Dokumen

- [ ] `PRD.md` terisi semua 8 bagian
- [ ] `SDLC.md` punya fase + definisi selesai
- [ ] `DESIGN.md` punya warna, font, spacing, breakpoint
- [ ] `ARCHITECTURE.md` punya stack, folder, kontrak API, env
- [ ] `TASKS.md` punya task bernomor + "Selesai kalau"
- [ ] `AGENTS.md` ada di root project
- [ ] AI agent bisa merangkum semua dokumen tanpa salah

**Bukti:** 6 file di repo + screenshot rangkuman agent.
**Checkpoint mentor 1** → lihat `kelas/CHECKPOINT-MENTOR.md`

---

## Level 3 — Localhost

- [ ] `npm run dev` jalan tanpa error merah
- [ ] Halaman utama tampil sesuai `DESIGN.md`
- [ ] Minimal 3 halaman jadi
- [ ] Responsive diuji di 375px / 768px / 1280px
- [ ] State loading / empty / error ada
- [ ] Backend: minimal 1 endpoint GET jalan
- [ ] Backend: minimal 1 endpoint POST dengan validasi server
- [ ] Database terhubung, migrasi sukses
- [ ] Register + login berfungsi
- [ ] Route member terproteksi (tanpa login → redirect)
- [ ] `npm run lint` 0 error
- [ ] `npm run build` sukses

> Jalur A: lewati baris backend, database, dan auth.

**Bukti:** screenshot `localhost:3000`, output `npm run build`, URL commit.
**Checkpoint mentor 2**

---

## Level 4 — Online

- [ ] Kode ter-push ke GitHub tanpa secret
- [ ] Deploy staging berhasil, URL bisa dibuka
- [ ] Environment variables terisi di platform deploy
- [ ] Alur kritis jalan di staging (bukan cuma lokal)
- [ ] Minimal 3 orang sudah mencoba dan kasih feedback
- [ ] Domain sendiri sudah mengarah
- [ ] HTTPS aktif, `http://` redirect ke `https://`
- [ ] Backup database aktif
- [ ] Restore backup pernah diuji minimal sekali
- [ ] Deploy produksi berhasil

**Bukti:** URL staging + URL produksi + screenshot gembok HTTPS + bukti backup.
**Checkpoint mentor 3 & 4**

---

## Level 5 — Siap Jualan (opsional)

- [ ] Payment sandbox berhasil end-to-end
- [ ] Webhook signature diverifikasi
- [ ] Webhook idempotent (kirim dua kali tidak dobel)
- [ ] Transaksi kecil produksi berhasil
- [ ] Email transaksional terkirim (tidak masuk spam)
- [ ] Monitoring uptime aktif
- [ ] Privacy policy & terms terpasang
- [ ] Jadwal maintenance bulanan dicatat

---

## Catatan Kendala

Tulis di sini tiap kali buntu, supaya tidak lupa saat tanya mentor.

| Tanggal | Bab | Masalah | Status |
|---|---|---|---|
| | | | |
| | | | |
