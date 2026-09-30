# 01 — Menulis PRD (Product Requirements Document)

## Tujuan

Membuat satu dokumen yang menjawab: apa yang dibangun, untuk siapa, mengapa, dan
batasan apa yang berlaku. Tanpa PRD, agent menebak fitur.

## Isi PRD

| Bagian | Isi |
|---|---|
| 1. Ringkasan | satu paragraf: apa ini, untuk siapa |
| 2. Masalah | masalah spesifik yang diselesaikan |
| 3. Pengguna | 2–3 persona utama |
| 4. Daftar fitur | fitur per fase (MVP, V2, nanti) |
| 5. Batasan | yang tidak dikerjakan |
| 6. Metrik sukses | kapan produk dianggap berhasil |
| 7. Dependensi | layanan luar yang dibutuhkan |
| 8. Risiko | hal yang bisa gagal |

## Contoh Bagian Fitur

```markdown
## 4. Fitur

### MVP (fase 1)
- [ ] F-001: Landing page — hero, fitur, pricing, CTA
- [ ] F-002: Daftar akun (email + password)
- [ ] F-003: Login + session
- [ ] F-004: Dashboard user — profil + pesanan
- [ ] F-005: Katalog produk — list + detail
- [ ] F-006: Checkout → Xendit invoice → email konfirmasi

### Fase 2
- [ ] F-007: Dashboard admin — CRUD produk
- [ ] F-008: Upload konten kursus
- [ ] F-009: Halaman kursus viewer

### Nanti
- [ ] F-010: Referral
- [ ] F-011: Notifikasi push
```

## Cara Menulis

1. Buka `templates/PRD.md`, salin ke root project.
2. Isi dari atas ke bawah. Jangan lompat bagian.
3. Setiap fitur punya nomor (F-001) dan tindakan yang bisa diverifikasi.
4. Tulis yang TIDAK termasuk di bagian Batasan — penting supaya agent tahu batas.
5. Baca ulang besoknya. Kalau ada yang tidak masuk akal, ubah sebelum coding.

## Kesalahan Umum

- Fitur ditulis terlalu abstrak: "manajemen konten" — yang mana?
- Tidak ada fase; semua harus langsung ada hari pertama.
- Tidak ada persona; project terasa tanpa pengguna nyata.
- PRD berisi kode atau nama tabel; itu urusan ARCHITECTURE.

## Cara Agent Membaca PRD

```text
Baca PRD.md. Tanpa mengubah apa pun, jawab:
1. Apa produk ini dan untuk siapa?
2. Apa 5 fitur MVP?
3. Apa yang tidak masuk scope?
4. Risiko terbesar?
5. Informasi apa yang masih kosong?
```

Kalau rangkuman agent salah, PRD yang perlu diperbaiki.

## Checklist

- [ ] PRD mengisi semua 8 bagian
- [ ] Setiap fitur punya nomor unik
- [ ] Fitur dikelompokkan per fase
- [ ] Batasan ditulis eksplisit
- [ ] Agent bisa merangkum PRD dengan benar
