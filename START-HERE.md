# START HERE — Mulai dari Sini

Halaman ini satu-satunya yang wajib kamu baca duluan. Jangan buka folder lain
sebelum selesai halaman ini.

---

## Langkah 1 — Jawab: Kamu Mau Bikin Apa?

| Kalau jawabanmu... | Jalurmu |
|---|---|
| "Belum tahu, mau bisa dulu" | **Jalur A — Pemula** |
| "Web yang ada login + database" | **Jalur B — Fullstack** |
| "Sudah punya hosting cPanel/domain" | **Jalur C — Shared Hosting** |
| "Mau jualan beneran, ada pembayaran" | **Jalur D — Production** |

Kalau ragu antara dua jalur, **ambil yang lebih rendah**. Naik jalur itu gampang,
turun jalur itu bikin patah semangat.

---

## Langkah 2 — Cek OS Kamu

| OS | Catatan |
|---|---|
| Windows 10/11 | Semua bab jalan. Terminal pakai Git Bash atau PowerShell |
| macOS | Semua bab jalan. Pasang Homebrew dulu |
| Linux | Semua bab jalan |
| Chromebook / HP saja | Belum cukup. Perlu laptop untuk kelas ini |

Detail pemasangan: [`docs/00-mulai-dari-sini/02-prasyarat-alat.md`](docs/00-mulai-dari-sini/02-prasyarat-alat.md)

---

## Langkah 3 — Pilih Jalur Belajar

### 🟢 Jalur A — Pemula

**Target:** Landing page online yang bisa dibagikan ke orang.
**Deploy:** Cloudflare Pages / Vercel (gratis)
**Waktu realistis:** 3–5 hari santai
**Biaya:** Rp 0

**▶ Mulai dari sini:** [`docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md`](docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md)

Urutan bab:
```
00 (semua) → 01 (semua) → 02/03-design-md → 03/01-memilih-stack
→ 04 (semua) → 07 → 08 → 09/01-deploy-staging
```
Lewati dulu: backend, database, payment, VPS.

---

### 🔵 Jalur B — Fullstack

**Target:** Next.js + database + login + dashboard.
**Deploy:** Vercel + Neon/Supabase
**Waktu realistis:** 2–4 minggu
**Biaya:** Rp 0 (tier gratis) → naik saat ramai

**▶ Mulai dari sini:** [`docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md`](docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md)

Urutan bab: `00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 12`

---

### 🟡 Jalur C — Shared Hosting (cPanel)

**Target:** Situs/aplikasi kecil di hosting yang sudah kamu beli.
**Deploy:** cPanel
**Waktu realistis:** 1–2 minggu
**Biaya:** harga hosting + domain

**▶ Mulai dari sini:** [`docs/11-hosting-cpanel/01-beli-hosting.md`](docs/11-hosting-cpanel/01-beli-hosting.md)

Urutan bab:
```
00 → 01 → 02 → 04 → 07 → 08
→ 11/01 → 11/02 → 11/03 → 11/04 → 11/05 → 11/06
→ 14/03 (hardening cPanel)
```

---

### 🔴 Jalur D — Production

**Target:** Produk jualan: fullstack + payment + domain sendiri + backup.
**Deploy:** VPS + Coolify (atau Vercel + VPS)
**Waktu realistis:** 4–8 minggu
**Biaya:** VPS + domain (+ fee payment gateway)

**▶ Mulai dari sini:** [`docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md`](docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md)

Urutan bab: semua `00`–`15`, tidak ada yang dilewati.

---

## Langkah 4 — Siapkan Tiga File Ini

Sebelum praktik, copy ke folder project kamu:

1. `templates/AGENTS.md` → aturan untuk AI agent
2. `PROGRESS.md` (dari repo ini) → penanda posisi belajarmu
3. `templates/PRD.md` → mulai isi ide produkmu

---

## Langkah 5 — Lihat Contoh Jadi Dulu

Sebelum menulis dokumen sendiri, **baca contoh yang sudah diisi**:

[`examples/toko-produk-digital/`](examples/toko-produk-digital/)

Di sana ada satu project utuh: dari ide mentah → PRD → task → prompt → error →
cara lapor error → localhost → deploy. Ini yang paling cepat bikin paham.

---

## Peta Cepat Repo

| Butuh | Buka |
|---|---|
| Tidak paham istilah | [`GLOSSARY.md`](GLOSSARY.md) |
| Bingung pilih hosting | [`docs/09-deploy-gratis/03-pilih-hosting-decision-tree.md`](docs/09-deploy-gratis/03-pilih-hosting-decision-tree.md) |
| Error dan buntu | [`docs/15-troubleshooting/`](docs/15-troubleshooting/) |
| Mau tanya mentor | [`BANTUAN.md`](BANTUAN.md) |
| Cek posisi belajar | [`PROGRESS.md`](PROGRESS.md) |
| Butuh prompt | [`prompts/README.md`](prompts/README.md) |
| File konfigurasi deploy | [`deployment-examples/`](deployment-examples/) |
| Uji paham | [`kelas/KUIS.md`](kelas/KUIS.md) |
| Setoran ke mentor | [`kelas/CHECKPOINT-MENTOR.md`](kelas/CHECKPOINT-MENTOR.md) |

---

## Tiga Aturan yang Bikin Kamu Tidak Nyasar

1. **Satu bab, satu praktik, satu bukti.** Jangan baca 5 bab sekaligus.
2. **Belum jalan di localhost, jangan deploy.**
3. **Buntu lebih dari 30 menit, buka [`BANTUAN.md`](BANTUAN.md).** Jangan diam
   tiga hari.

---

## Checklist Sebelum Lanjut

- [ ] Saya sudah pilih jalur (A/B/C/D)
- [ ] OS saya sudah siap
- [ ] Saya sudah copy `PROGRESS.md`
- [ ] Saya sudah lihat `examples/toko-produk-digital/`
- [ ] Saya tahu ke mana bertanya kalau buntu

Kalau lima kotak di atas tercentang, buka bab pertama jalurmu.
