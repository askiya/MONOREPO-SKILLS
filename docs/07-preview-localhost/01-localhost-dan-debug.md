# 01 — Preview Localhost

## Tujuan

Menjalankan app lengkap di mesin lokal, melihat UI, dan mencatat bug sebelum
deploy.

## Langkah

### 1. Jalankan Dev Server

```bash
npm run dev
```

Buka `http://localhost:3000`. Kalau port bentrok:

```bash
PORT=3001 npm run dev
```

### 2. Cek Browser Console

Buka DevTools (F12 / Cmd+Opt+I). Tab **Console** — kalau ada merah, catat
seluruh pesan error dan perbaiki sebelum lanjut. Warning kuning boleh ditunda
tapi tetap dicatat.

### 3. Cek Network

Tab **Network** — perhatikan:
- Request yang gagal (status 400/401/403/404/500).
- Request tanpa response (pending > 10 detik).
- File besar yang tidak perlu (gambar mentah > 1 MB).

### 4. Cek Responsive

Toggle responsive mode:
- 375px (HP kecil)
- 768px (tablet)
- 1280px (desktop)

Yang harus dicek:
- Teks tidak keluar layar.
- Tombol bisa ditekan.
- Menu mobile muncul dan bisa dipakai.
- Form tidak tertutup keyboard di HP.

### 5. Cek Alur Kritis

Coba jalur utama end-to-end:

1. Landing → klik CTA.
2. Register → login.
3. Dashboard tampil.
4. Katalog → detail → checkout (pakai data uji).
5. Admin CRUD satu item.

Catat temuan di format bug:

```
BUG-001
Halaman: /produk
Langkah: buka halaman, scroll ke bawah
Expected: pagination tampil
Actual: hanya 10 item, tidak ada tombol lanjut
Console: (kosong)
```

## Jangan Deploy Kalau

- Ada error merah di console.
- Halaman utama tidak bisa dibuka.
- Alur kritis ada yang macet.
- `npm run build` gagal.

## Checklist

- [ ] Dev server jalan tanpa error merah
- [ ] Alur kritis 5 langkah bisa dilalui
- [ ] Uji 3 viewport
- [ ] Bug dicatat, bukan dilupakan
- [ ] `npm run build` sukses
