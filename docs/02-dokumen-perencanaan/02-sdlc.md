# 02 — Menulis SDLC (Tahapan Pengembangan)

## Tujuan

Menetapkan tahapan kerja: apa yang terjadi di setiap fase, siapa yang melakukan
apa, dan kapan dianggap selesai.

## Format

```markdown
# SDLC.md

## Fase 1 — Perencanaan
Durasi: 1–2 hari
Input: ide, riset kompetitor
Output: PRD.md final
Selesai kalau: PRD dibaca ulang dan setiap fitur bisa dijawab "kenapa?"

## Fase 2 — Desain & Arsitektur
Durasi: 1 hari
Input: PRD
Output: DESIGN.md, ARCHITECTURE.md, TASKS.md
Selesai kalau: agent bisa merangkum 3 dokumen tanpa kontradiksi

## Fase 3 — Build MVP
Durasi: 5–10 hari
Input: semua dokumen + TASKS.md
Output: app berjalan di localhost
Selesai kalau: F-001 – F-006 berfungsi, build hijau

## Fase 4 — Testing
Durasi: 1–2 hari
Input: app localhost
Output: lint + build + test hijau, bug kritis 0
Selesai kalau: checklist testing tercentang semua

## Fase 5 — Deploy Staging
Durasi: 1 hari
Input: build hijau
Output: URL publik staging
Selesai kalau: bisa diakses 3 orang luar

## Fase 6 — Produksi
Durasi: 1–2 hari
Input: staging teruji
Output: domain sendiri, HTTPS, payment
Selesai kalau: transaksi percobaan berhasil end-to-end
```

## Kenapa Butuh SDLC

Tanpa SDLC:
- "Kapan ini selesai?" jawabannya selalu "nanti".
- Fase testing dilewat karena sudah capek.
- Deploy produksi dilakukan sebelum staging.

SDLC = kontrak dengan diri sendiri.

## Tips

- Durasi boleh diubah, tapi ukur realistis.
- Definisi "selesai" wajib bisa dites, bukan perasaan.
- Tulis SDLC setelah PRD, sebelum DESIGN.md.
- Kalau project sampingan, buat juga SDLC meski lebih santai.

## Checklist

- [ ] SDLC mencakup minimal 5 fase
- [ ] Setiap fase punya input, output, dan definisi selesai
- [ ] Durasi realistis
- [ ] Urutan fase tidak melompat (tidak deploy sebelum test)
