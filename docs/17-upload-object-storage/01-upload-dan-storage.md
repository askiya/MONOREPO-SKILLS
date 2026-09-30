# 01 — Upload File dan Object Storage

## Jangan Simpan Upload di Disk Aplikasi

Alasan:
- container/serverless punya filesystem sementara — file hilang saat redeploy,
- disk VPS penuh menyebabkan aplikasi mati,
- sulit di-backup dan di-scale.

Pakai object storage: Cloudflare R2, Amazon S3, Backblaze B2, atau Supabase
Storage. Periksa harga/kuota terkini di situs resmi.

## Alur Aman: Signed URL

Jangan salurkan file besar lewat server aplikasi. Pola yang benar:

```text
1. Client minta izin upload  → POST /api/upload/sign
2. Server validasi (login? tipe file? ukuran?)
3. Server buat presigned URL berumur pendek (5-15 menit)
4. Client upload LANGSUNG ke storage pakai URL itu
5. Client lapor selesai        → POST /api/upload/confirm
6. Server simpan metadata (key, ukuran, pemilik) ke database
```

Keuntungan: server tidak menahan file besar, bandwidth hemat, lebih cepat.

## Validasi Wajib di Server

Jangan percaya apa pun dari client.

| Yang divalidasi | Kenapa |
|---|---|
| User login & berhak | cegah upload anonim |
| Ekstensi + MIME type | cegah upload skrip |
| Ukuran maksimum | cegah disk/kuota habis |
| Nama file di-generate ulang | cegah path traversal (`../../`) |
| Magic bytes file (kalau kritis) | ekstensi bisa dipalsukan |
| Rate limit per user | cegah abuse |

Nama file: **jangan** pakai nama asli dari user. Generate UUID + ekstensi yang
sudah divalidasi.

```text
Salah : uploads/foto profil (1).jpg
Benar : uploads/u_123/9f2b1c4e-....jpg
```

## Bucket Publik vs Privat

| Jenis file | Bucket | Cara akses |
|---|---|---|
| Foto profil, gambar produk | publik | URL langsung / CDN |
| E-book, invoice, dokumen berbayar | privat | signed URL berumur pendek |

**Jangan** menaruh produk digital berbayar di bucket publik. URL akan disebar.

Untuk produk berbayar: cek entitlement user di server, baru terbitkan signed URL
yang berlaku beberapa menit.

## Gambar

- Batasi dimensi maksimum, resize di server/CDN, jangan simpan 12MP mentah.
- Pakai format modern (WebP/AVIF) bila memungkinkan.
- Simpan varian thumbnail supaya listing ringan.
- Selalu isi `alt` untuk aksesibilitas.

## CORS

Upload langsung dari browser membutuhkan konfigurasi CORS di bucket:
izinkan origin aplikasi kamu saja, bukan `*`.

## Checklist

- [ ] File user tidak disimpan di disk aplikasi
- [ ] Upload memakai signed URL, bukan lewat server
- [ ] Validasi login, tipe, dan ukuran di server
- [ ] Nama file di-generate ulang (UUID)
- [ ] Bucket privat untuk konten berbayar
- [ ] CORS dibatasi ke origin aplikasi
- [ ] Rate limit upload per user
- [ ] Metadata file tersimpan di database
