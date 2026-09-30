# 05 — Deploy Manual: VPS + Docker + Nginx + CI/CD

## Tujuan

VPS aman menjalankan stack Docker Compose; Nginx meneruskan web dan API; GitHub Actions melakukan deploy via SSH dengan prosedur verifikasi dan rollback.

## Prasyarat

- Semua langkah `04-preview.md` lulus.
- Repo ada di GitHub.
- Domain sudah dimiliki.

> Rujukan wajib:
>
> - [`../../docs/10-deploy-vps/01-setup-vps.md`](../../docs/10-deploy-vps/01-setup-vps.md)
> - [`../../docs/10-deploy-vps/03-docker-nginx-ssl.md`](../../docs/10-deploy-vps/03-docker-nginx-ssl.md)
> - [`../../deployment-examples/docker/`](../../deployment-examples/docker/)
> - [`../../deployment-examples/nginx/`](../../deployment-examples/nginx/)

## 1. Beli VPS

Gunakan Ubuntu LTS, minimal 2 vCPU, 4 GB RAM, dan 40–60 GB SSD. Pilih Hetzner, DigitalOcean, Tencent Cloud, Vultr, Linode, atau provider lokal sesuai region pengguna. Harga berubah; cek situs provider.

## 2. Hardening SSH — Ikuti Urutan

Bagian ini dapat mengunci kamu dari server bila urutannya salah. Jangan tutup sesi root pertama sebelum user deploy terbukti bisa login dengan SSH key di terminal kedua.

**Langkah 1.** Login pertama dan update sistem.

```bash
ssh root@IP_SERVER
apt update && apt upgrade -y
```

**Langkah 2.** Buat user deploy.

```bash
adduser deploy
usermod -aG sudo deploy
```

**Langkah 3.** Dari komputer lokal, buat dan salin key.

```bash
ssh-keygen -t ed25519 -C "deploy@project"
ssh-copy-id deploy@IP_SERVER
```

**Langkah 4.** Buka terminal kedua dan buktikan login key berhasil.

```bash
ssh deploy@IP_SERVER
```

Jangan lanjut jika perintah ini gagal. Perbaiki key dahulu; terminal root pertama tetap terbuka sebagai jalur pemulihan.

**Langkah 5.** Setelah key terbukti bekerja, edit `/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart ssh
```

Buka terminal ketiga dan verifikasi login `deploy` sekali lagi. Tutup sesi root hanya setelah verifikasi ini lulus.

## 3. Firewall dan Fail2ban

Izinkan SSH sebelum mengaktifkan firewall. Membalik urutan akan memutus akses kamu.

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

**Hasil yang diharapkan:** UFW `active`; hanya OpenSSH, 80, dan 443 diizinkan; Fail2ban `active`.

## 4. Instal Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker deploy
```

Logout lalu login kembali agar grup aktif:

```bash
exit
ssh deploy@IP_SERVER
docker run --rm hello-world
docker compose version
```

**Hasil yang diharapkan:** pesan `Hello from Docker!` dan versi Compose tercetak.

## 5. Siapkan Direktori Produksi

```bash
sudo mkdir -p /opt/myapp
sudo chown deploy:deploy /opt/myapp
cd /opt/myapp
git clone https://github.com/USERNAME/REPOSITORY.git .
```

Jika repo privat, gunakan deploy key read-only GitHub. Jangan menyimpan personal access token di command history.

Buat `.env` langsung di server:

```bash
touch .env
chmod 600 .env
nano .env
```

Isi value produksi:

```dotenv
POSTGRES_USER=myapp
POSTGRES_PASSWORD=<ISI_SENDIRI>
POSTGRES_DB=myapp
DATABASE_URL=postgresql://myapp:<ISI_SENDIRI>@db:5432/myapp
NEXT_PUBLIC_API_URL=https://api.domain-kamu.com
PORT=4000
```

Gunakan password yang sama pada `POSTGRES_PASSWORD` dan bagian password `DATABASE_URL`. Jangan commit `.env`.

## 6. Jalankan Docker Compose

```bash
cd /opt/myapp
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs --tail=100 web api
```

**Hasil yang diharapkan:** semua service `Up`, database `healthy`, dan tidak ada restart loop.

Verifikasi dari server:

```bash
curl -s http://127.0.0.1:3000
curl -s http://127.0.0.1:4000/health
```

API harus mengembalikan `{"status":"ok"}`. Web mengembalikan HTML.

Jalankan migrasi produksi:

```bash
docker compose exec api npx prisma migrate deploy
```

## 7. Instal Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

Buat `/etc/nginx/sites-available/myapp.conf`:

```nginx
server {
    listen 80;
    server_name domain-kamu.com www.domain-kamu.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name api.domain-kamu.com;

    location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan hanya setelah syntax lulus:

```bash
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/myapp.conf
sudo nginx -t
sudo systemctl reload nginx
```

**Hasil yang diharapkan:** `syntax is ok` dan `test is successful`. Jika `nginx -t` gagal, jangan reload. Perbaiki file lebih dulu.

## 8. Arahkan DNS Sebelum Certbot

Di provider DNS, buat:

| Type | Name | Content |
|---|---|---|
| A | `@` | `IP_SERVER` |
| A | `www` | `IP_SERVER` |
| A | `api` | `IP_SERVER` |

Tunggu sampai DNS mengarah ke IP server:

```bash
nslookup domain-kamu.com
nslookup api.domain-kamu.com
```

Jangan menjalankan Certbot sebelum kedua nama mengembalikan IP server; validasi Let's Encrypt akan gagal.

## 9. Pasang Certbot SSL

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx \
  -d domain-kamu.com \
  -d www.domain-kamu.com \
  -d api.domain-kamu.com
```

Pilih redirect HTTP ke HTTPS saat ditanya.

Uji auto-renew:

```bash
sudo certbot renew --dry-run
```

**Hasil yang diharapkan:** `Congratulations, all simulated renewals succeeded`.

## 10. GitHub Actions Deploy via SSH

Buat SSH key khusus CI di komputer lokal. Jangan gunakan private key pribadi:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ./github-actions-deploy
ssh-copy-id -i ./github-actions-deploy.pub deploy@IP_SERVER
```

Masukkan private key `github-actions-deploy` ke GitHub repository → **Settings → Secrets and variables → Actions** sebagai `VPS_SSH_KEY`. Tambahkan juga:

| Secret | Nilai |
|---|---|
| `VPS_HOST` | IP server |
| `VPS_USER` | `deploy` |

Buat `.github/workflows/deploy.yml`:

```yaml
name: Deploy Production

on:
  push:
    branches: [main]

concurrency:
  group: production
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm run lint --workspaces
      - run: npm test --workspaces

      - name: Konfigurasi SSH
        run: |
          mkdir -p ~/.ssh
          printf '%s' "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H "${{ secrets.VPS_HOST }}" >> ~/.ssh/known_hosts

      - name: Deploy
        run: |
          ssh "${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }}" '
            set -e
            cd /opt/myapp
            git fetch origin main
            git checkout main
            git reset --hard origin/main
            docker compose build
            docker compose run --rm api npx prisma migrate deploy
            docker compose up -d --remove-orphans
            docker compose ps
            curl -fsS http://127.0.0.1:4000/health
          '
```

> `ssh-keyscan` di contoh ini mengambil host key saat job berjalan. Untuk keamanan lebih kuat, simpan fingerprint host yang sudah diverifikasi sebagai secret dan tulis nilai itu ke `known_hosts`.

Push ke `main`, lalu buka tab **Actions** di GitHub.

**Hasil yang diharapkan:** lint dan test lulus; SSH deploy sukses; health check exit `0`.

## 11. Rollback

Sebelum rilis yang mengubah skema, ambil backup (lihat `07-maintenance.md`). Catat commit sebelumnya:

```bash
git rev-parse HEAD
```

Jika rilis rusak:

```bash
cd /opt/myapp
git log --oneline -5
git checkout COMMIT_SEBELUMNYA
docker compose up -d --build
docker compose ps
curl -fsS http://127.0.0.1:4000/health
```

Rollback aplikasi tidak otomatis mengembalikan skema database. Jika migrasi destruktif sudah berjalan, restore backup ke database uji dahulu, verifikasi, lalu baru putuskan restore produksi.

## Kegagalan Umum

| Gejala | Perbaikan |
|---|---|
| SSH Action `Permission denied` | Cek public key ada di `~/.ssh/authorized_keys`, permission `700` untuk `.ssh`, `600` untuk key |
| Container restart loop | `docker compose logs --tail=200 SERVICE` |
| `502 Bad Gateway` | Pastikan service hidup dan bind `127.0.0.1:3000/4000` |
| Nginx gagal reload | Jalankan `sudo nginx -t`, perbaiki syntax |
| Certbot gagal validasi | Cek DNS sudah ke IP server dan port 80 terbuka |
| Deploy kehabisan disk | `docker system df`, lalu prune image tak terpakai |
| Migrasi gagal | Jangan lanjut; baca error dan pulihkan dari backup bila perlu |

## Checklist

- [ ] SSH key user non-root terbukti bekerja sebelum password/root dimatikan.
- [ ] UFW hanya membuka SSH, 80, 443; Fail2ban aktif.
- [ ] Docker dan Compose berjalan untuk user deploy.
- [ ] `.env` produksi permission `600`, tidak ada di repo.
- [ ] Compose sehat; DB tidak publish port publik.
- [ ] Migrasi produksi sukses.
- [ ] Nginx config lulus `nginx -t` dan web/API ter-route benar.
- [ ] Certbot sukses; auto-renew dry run lulus.
- [ ] GitHub Actions lint, test, deploy, health check lulus.
- [ ] Commit rollback diketahui dan prosedur rollback diuji di staging.
