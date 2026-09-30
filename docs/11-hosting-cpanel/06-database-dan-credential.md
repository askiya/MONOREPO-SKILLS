# 06 — Database & Environment Credential di cPanel

## Tujuan

Database siap, kredensial aman, aplikasi terhubung.

## Buat Database MySQL

### Cara Cepat: Database Wizard

1. cPanel → **MySQL Database Wizard**.
2. **Step 1** — Nama database: `myapp` → jadi `username_myapp`.
3. **Step 2** — Buat user:
   - Username: `myappuser` → jadi `username_myappuser`
   - Password: klik **Password Generator**, simpan di password manager
4. **Step 3** — Privileges: centang **ALL PRIVILEGES**.
5. **Next Step** → selesai.

**Catat empat hal ini:**

```
DB_HOST     = localhost
DB_NAME     = username_myapp
DB_USER     = username_myappuser
DB_PASSWORD = <hasil generator>
```

Nama database dan user **selalu diawali prefix username cPanel**. Jangan lupa
prefix-nya saat menulis connection string.

### PostgreSQL

Kalau hosting menyediakan **PostgreSQL Databases**, langkahnya sama.
Kalau tidak ada, pilihan:
- pakai MySQL,
- pakai PostgreSQL eksternal (Neon/Supabase) — koneksi dari cPanel ke luar
  kadang diblokir, cek dulu.

## Connection String

MySQL:

```
mysql://username_myappuser:PASSWORD@localhost:3306/username_myapp
```

PostgreSQL:

```
postgresql://username_myappuser:PASSWORD@localhost:5432/username_myapp
```

**Password dengan karakter khusus** (`@`, `#`, `/`, `:`) harus di-URL-encode:

| Karakter | Encode |
|---|---|
| `@` | `%40` |
| `#` | `%23` |
| `/` | `%2F` |
| `:` | `%3A` |
| `?` | `%3F` |
| `&` | `%26` |

Lebih aman: generate password yang hanya berisi huruf + angka.

## Import Database

### Via phpMyAdmin

1. cPanel → **phpMyAdmin**.
2. Pilih database di sidebar kiri.
3. Tab **Import** → Choose File → pilih `.sql`.
4. **Go**.

Batasan ukuran upload biasanya 50MB. Kalau lebih besar:
- kompres jadi `.sql.gz` (phpMyAdmin bisa baca),
- atau split file SQL,
- atau minta bantuan support hosting.

### Via Terminal (kalau SSH tersedia)

```bash
mysql -u username_myappuser -p username_myapp < backup.sql
```

## Simpan Credential dengan Benar

### ❌ SALAH

```
public_html/.env                    ← bisa diakses publik
public_html/config.js               ← bisa dibaca browser
public_html/backup.sql              ← database bocor
kode: const pass = "rahasia123"     ← masuk Git
```

### ✅ BENAR

**Untuk Node.js App:**
cPanel → Setup Node.js App → Environment Variables → Add.

**Untuk PHP:**
File `.env` **di luar** `public_html`:

```
/home/username/config/.env          ← aman
/home/username/public_html/         ← folder publik
```

Lalu baca dari PHP:

```php
$env = parse_ini_file('/home/username/config/.env');
$dbPass = $env['DB_PASSWORD'];
```

**Atau** pakai `.htaccess` untuk memblokir akses file env:

```apache
<Files ".env">
  Order allow,deny
  Deny from all
</Files>
```

Tapi cara paling aman tetap: **taruh di luar `public_html`**.

## Tes Koneksi

Buat file tes sementara `public_html/dbtest.php`:

```php
<?php
$conn = new mysqli('localhost', 'username_myappuser', 'PASSWORD', 'username_myapp');
if ($conn->connect_error) {
    die('Gagal: ' . $conn->connect_error);
}
echo 'Koneksi berhasil';
$conn->close();
```

Buka `https://domainmu.com/dbtest.php`.

**HAPUS file ini segera setelah tes.** Jangan biarkan file tes berisi password
di folder publik.

## Remote Database Access

Kalau ingin akses DB dari komputer lokal (DBeaver/pgAdmin):

1. cPanel → **Remote MySQL**.
2. Tambah IP publik kamu (cek di whatismyip.com).
3. Connect dari client:

```
Host: domainmu.com (atau IP hosting)
Port: 3306
User: username_myappuser
Password: <password>
Database: username_myapp
```

**Jangan** tambahkan `%` (semua IP) — itu membuka database ke seluruh internet.

## Backup Database

### Manual

cPanel → **Backup** → **Download a MySQL Database Backup** → pilih database.

Simpan hasil download di luar server (Google Drive, komputer lokal).

### Otomatis via Cron

cPanel → **Cron Jobs** → tambah:

```
0 2 * * * mysqldump -u username_myappuser -pPASSWORD username_myapp > /home/username/backups/db_$(date +\%Y\%m\%d).sql
```

Jadwal: setiap hari jam 02:00.

**Catatan keamanan:** password di cron command bisa terlihat proses lain.
Lebih aman pakai `.my.cnf`:

`/home/username/.my.cnf` (chmod 600):

```ini
[mysqldump]
user=username_myappuser
password=PASSWORD
```

Lalu cron:

```
0 2 * * * mysqldump username_myapp > /home/username/backups/db_$(date +\%Y\%m\%d).sql
```

Hapus backup lama supaya disk tidak penuh:

```
0 3 * * 0 find /home/username/backups -name "db_*.sql" -mtime +14 -delete
```

## Checklist

- [ ] Database dan user dibuat
- [ ] Privileges ALL diberikan
- [ ] Credential dicatat di password manager
- [ ] Connection string benar (termasuk prefix username)
- [ ] Password URL-encoded kalau ada karakter khusus
- [ ] Credential TIDAK di `public_html`
- [ ] File tes koneksi sudah dihapus
- [ ] Remote MySQL dibatasi IP tertentu (bukan `%`)
- [ ] Backup otomatis aktif
- [ ] Backup disalin ke luar server
