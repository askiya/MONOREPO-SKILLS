# 01 — Email Transaksional

Email transaksional = email otomatis akibat aksi user: verifikasi akun, reset
password, struk pembelian, notifikasi order.

## Jangan Pakai SMTP Gmail Pribadi

Alasan:
- limit harian kecil,
- akun bisa dikunci,
- deliverability buruk saat volume naik,
- kredensial pribadi di server aplikasi.

Pakai layanan email transaksional (Resend, Brevo, Mailgun, Amazon SES, Postmark,
atau SMTP dari hosting untuk volume kecil). Periksa harga dan kuota di situs
resmi masing-masing — angka cepat berubah.

## Alur Minimum

1. Daftar layanan.
2. Verifikasi domain pengirim (bukan cuma alamat email).
3. Pasang DNS: SPF, DKIM, DMARC.
4. Simpan API key di environment variable.
5. Kirim email uji ke Gmail, Outlook, Yahoo.
6. Cek tidak masuk spam.

## DNS yang Wajib

### SPF
Memberi tahu penerima server mana yang boleh mengirim atas nama domainmu.

```text
Type : TXT
Name : @
Value: v=spf1 include:PENYEDIA-EMAIL.com ~all
```

Satu domain **hanya boleh punya satu record SPF**. Kalau sudah ada, gabungkan
`include:` ke record yang ada, jangan buat TXT kedua.

### DKIM
Tanda tangan kriptografis agar isi email terbukti asli.

```text
Type : TXT / CNAME  (ikuti instruksi penyedia)
Name : [selector]._domainkey
Value: [nilai dari dashboard penyedia]
```

### DMARC
Memberi tahu penerima apa yang dilakukan kalau SPF/DKIM gagal.

```text
Type : TXT
Name : _dmarc
Value: v=DMARC1; p=none; rua=mailto:laporan@domainmu.com
```

Mulai dari `p=none` (hanya memantau). Setelah yakin SPF+DKIM lulus konsisten,
naikkan ke `p=quarantine` lalu `p=reject`.

## Verifikasi

Kirim email ke alamat Gmail, buka email → **Show original**:

```text
SPF   : PASS
DKIM  : PASS
DMARC : PASS
```

Kalau ada yang FAIL, email berisiko masuk spam.

Cek DNS:
```bash
nslookup -type=TXT domainmu.com
nslookup -type=TXT _dmarc.domainmu.com
```

## Aturan Isi Email

- Selalu ada teks plain, jangan HTML saja.
- Subject jelas, bukan clickbait.
- Alamat pengirim pakai domain sendiri (`noreply@domainmu.com`).
- Reply-to mengarah ke alamat yang benar-benar dibaca.
- Untuk email marketing: wajib ada link unsubscribe. Email transaksional dan
  marketing sebaiknya dipisah (domain/subdomain berbeda) agar reputasi terjaga.

## Keamanan

- API key email hanya di environment variable, tidak pernah di kode client.
- Jangan kirim password dalam bentuk plain di email.
- Link reset password: token acak, sekali pakai, kedaluwarsa (contoh 30 menit).
- Rate limit endpoint "lupa password" agar tidak jadi alat spam.
- Jangan bocorkan apakah email terdaftar ("kalau email terdaftar, kami kirim
  instruksi") — hindari enumerasi akun.

## Checklist

- [ ] Layanan email transaksional dipilih
- [ ] Domain pengirim terverifikasi
- [ ] SPF, DKIM, DMARC terpasang dan PASS
- [ ] API key di environment variable
- [ ] Email uji sampai ke Gmail tanpa masuk spam
- [ ] Token reset password sekali pakai + kedaluwarsa
- [ ] Rate limit di endpoint pengiriman email
