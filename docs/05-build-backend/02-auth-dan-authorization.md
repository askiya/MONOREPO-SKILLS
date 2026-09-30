# 02 — Authentication dan Authorization

## Tujuan

Membedakan siapa pengguna (authentication) dan apa yang boleh dilakukan
(authorization).

## Pilihan Sederhana

- Butuh Google/GitHub login: pakai NextAuth/Auth.js.
- Hanya email+password dan API sendiri: session cookie atau JWT yang dirancang
  hati-hati.
- Jangan membuat crypto sendiri.

## Aturan Keamanan

- Password di-hash Argon2id atau bcrypt; tidak pernah plaintext.
- Session/token lewat cookie `HttpOnly`, `Secure` di produksi, `SameSite` sesuai
  flow. Hindari token sensitif di `localStorage` kalau cookie session cukup.
- Rate-limit login, register, forgot password.
- Pesan reset password tidak membocorkan apakah email terdaftar.
- Otorisasi diperiksa server pada setiap endpoint sensitif.
- UI menyembunyikan tombol bukan perlindungan.

## Matriks Role

| Aksi | Publik | Member | Admin |
|---|---:|---:|---:|
| Lihat katalog | ya | ya | ya |
| Beli produk | tidak | ya | ya |
| Lihat library sendiri | tidak | ya | ya |
| CRUD produk | tidak | tidak | ya |
| Kelola user | tidak | tidak | ya |

Masukkan matriks ke `ARCHITECTURE.md`.

## Prompt Agent

```text
Implementasikan proteksi route sesuai matriks role di ARCHITECTURE.md.
Centralize pengecekan session dan role; jangan copy-paste. Tambah test 401 untuk
tanpa login dan 403 untuk role salah. Jangan mengubah UI selain redirect yang
diperlukan. Jalankan test dan build.
```

## Checklist

- [ ] Password di-hash
- [ ] Cookie/session aman
- [ ] Endpoint cek role di server
- [ ] 401 dan 403 dibedakan
- [ ] Rate limit endpoint auth
- [ ] Secret hanya environment variable
