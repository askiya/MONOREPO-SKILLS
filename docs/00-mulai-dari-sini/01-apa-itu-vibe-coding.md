# 01 — Apa Itu Vibe Coding

## Tujuan

Paham posisi kamu saat vibe coding: kamu **bukan** pengetik kode, kamu **arsitek
+ reviewer**. AI agent yang mengetik.

## Definisi Kerja

Vibe coding = membangun software dengan cara:

1. Menulis niat produk dalam dokumen yang jelas.
2. Menyerahkan dokumen itu ke AI agent yang punya akses ke folder project.
3. Agent menulis kode, menjalankan perintah, memperbaiki error.
4. Kamu membaca hasil, mencoba di localhost, minta revisi.

Yang **bukan** vibe coding: ngetik "bikinin web toko online" lalu berharap jadi.
Itu namanya berharap.

## Kenapa Dokumen Menentukan Segalanya

AI agent mengisi kekosongan dengan tebakan. Tidak ada PRD → dia tebak fitur.
Tidak ada DESIGN.md → dia pakai warna bawaan framework. Tidak ada ARCHITECTURE.md
→ dia bikin struktur folder acak yang bulan depan bikin kamu pusing.

| Tanpa dokumen | Dengan dokumen |
|---|---|
| Fitur meleset dari kebutuhan | Fitur sesuai daftar |
| UI tiap halaman beda gaya | UI konsisten satu token |
| Struktur folder berantakan | Struktur bisa diprediksi |
| Revisi berputar-putar | Revisi terarah |

## Pembagian Peran

| Peran | Siapa |
|---|---|
| Menentukan masalah & pengguna | Kamu |
| Menentukan stack & batasan | Kamu (dibantu bab 03) |
| Menulis kode | AI agent |
| Menjalankan build/test | AI agent |
| Menilai hasil di layar | Kamu |
| Menekan tombol deploy produksi | Kamu |

## Tiga Level Kualitas Prompt

**Level 1 — buruk**
> bikinin website jualan

**Level 2 — lumayan**
> Bikin halaman katalog produk Next.js, ambil data dari `/api/products`,
> tampilkan grid 3 kolom.

**Level 3 — benar**
> Baca `PRD.md` bagian 4.2 dan `DESIGN.md`. Implementasikan halaman
> `/produk` sesuai task `T-012` di `TASKS.md`. Pakai komponen `ProductCard`
> yang sudah ada di `components/`. Jangan buat komponen baru. Setelah selesai,
> jalankan `npm run build` dan laporkan hasilnya.

Level 3 selalu lebih cepat selesai, meski ngetiknya lebih lama.

## Kesalahan Pemula

1. Prompt sepanjang satu paragraf untuk 12 fitur sekaligus.
2. Tidak pernah membuka hasilnya di browser, langsung minta fitur berikutnya.
3. Menerima kode yang tidak dipahami sama sekali.
4. Menaruh API key di kode lalu push ke GitHub publik.
5. Deploy ke VPS padahal di localhost saja masih error.
6. Tidak pernah commit, lalu satu perubahan salah menghapus kerja seminggu.

## Checklist

- [ ] Saya paham tugas saya adalah arsitek + reviewer
- [ ] Saya tahu dokumen dulu, kode belakangan
- [ ] Saya siap membaca diff, bukan cuma menerima
- [ ] Saya bisa menulis prompt level 3
