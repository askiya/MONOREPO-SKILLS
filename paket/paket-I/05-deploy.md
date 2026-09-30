# 05 — Deploy VPS dengan Docker Compose

## Tujuan

Menjalankan stack produksi satu-node dengan firewall, Compose produksi, Traefik, healthcheck, restart policy, dan rollback yang jelas.

## Prasyarat

- Ubuntu LTS terbaru yang didukung provider, akses SSH key, user `sudo`.
- DNS belum dipindah sampai validasi origin selesai.
- Backup/snapshot VPS sebelum perubahan besar.

## 1. Amankan Akses Sebelum Firewall

Jangan tutup sesi SSH aktif.

1. Buat user deploy dan pasang public key.
2. Buka terminal kedua dan buktikan login key berhasil.
3. Izinkan port SSH, HTTP, HTTPS.
4. Baru aktifkan firewall.
5. Setelah login key kembali diuji, nonaktifkan password login dan root login.

```bash
sudo adduser deploy
sudo usermod -aG sudo deploy
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

**Hasil yang diharapkan:** terminal kedua tetap bisa SSH; hanya SSH, `80`, dan `443` diizinkan.

## 2. Instal Docker Resmi

Ikuti repositori resmi Docker untuk versi Ubuntu aktif: <https://docs.docker.com/engine/install/ubuntu/>. Jangan memakai script acak.

```bash
docker --version
docker compose version
```

**Hasil yang diharapkan:** Docker Engine dan Compose plugin menampilkan versi.

## 3. Siapkan Direktori dan Secret

```bash
sudo mkdir -p /opt/multiservice/{app,data/traefik,backups}
sudo chown -R deploy:deploy /opt/multiservice
cd /opt/multiservice/app
```

Kirim source melalui mekanisme rilis tim. Buat `/opt/multiservice/app/.env` langsung di server dengan permission `600`:

```bash
chmod 600 .env
```

Nilai minimum: password DB acak kuat, `DATABASE_URL`, email ACME, hostname, dan token monitor. Jangan taruh secret di `compose.yaml` atau image.

## 4. Compose Produksi

Tambahkan pada semua service aplikasi:

```yaml
restart: unless-stopped
security_opt:
  - no-new-privileges:true
```

Tambahkan healthcheck HTTP untuk `web` dan `api`, healthcheck proses untuk data service, batas log, serta label Traefik:

```yaml
labels:
  - traefik.enable=true
  - traefik.http.routers.api.rule=Host(`${API_HOST}`)
  - traefik.http.routers.api.entrypoints=websecure
  - traefik.http.routers.api.tls.certresolver=letsencrypt
  - traefik.http.services.api.loadbalancer.server.port=4000
```

Traefik menggunakan volume Docker socket read-only dan `acme.json` permission `600`. Pin image ke versi stabil, bukan `latest`.

## 5. Deploy

```bash
cd /opt/multiservice/app
touch data/traefik/acme.json
chmod 600 data/traefik/acme.json
docker compose -f compose.yaml -f compose.prod.yaml config --quiet
docker compose -f compose.yaml -f compose.prod.yaml pull
docker compose -f compose.yaml -f compose.prod.yaml build
docker compose -f compose.yaml -f compose.prod.yaml up -d
docker compose -f compose.yaml -f compose.prod.yaml ps
```

**Hasil yang diharapkan:** seluruh container `Up`; service dengan healthcheck berstatus `healthy`.

## 6. Verifikasi Sebelum DNS

Dari VPS:

```bash
curl --fail http://127.0.0.1/ping
```

Jika router membutuhkan hostname:

```bash
curl --fail -H 'Host: api.example.com' http://127.0.0.1/health
```

**Hasil yang diharapkan:** Traefik merutekan request ke API sehat. Setelah ini lanjutkan DNS pada bab berikut.

## 7. Uji Restart dan Rollback

```bash
sudo systemctl restart docker
cd /opt/multiservice/app
docker compose -f compose.yaml -f compose.prod.yaml ps
```

**Hasil yang diharapkan:** semua layanan kembali hidup karena `restart: unless-stopped`.

Rollback:

1. Pilih tag image/rilis terakhir yang sehat.
2. Pulihkan schema hanya bila migrasi tidak backward-compatible dan backup telah diuji.
3. Jalankan `docker compose up -d`.
4. Ulangi healthcheck dan smoke test.

## Masalah Umum

| Gejala | Perbaikan |
|---|---|
| permission Docker ditolak | gunakan `sudo` atau atur grup Docker dengan memahami hak setara root |
| Traefik tidak melihat service | pastikan satu jaringan `proxy` dan label aktif |
| restart loop | baca `docker compose logs --tail=200 SERVICE` |
| migrasi balapan | jalankan migrasi sebagai job satu kali sebelum replica aplikasi |
| disk cepat penuh | aktifkan rotasi log dan alarm disk |

## Checklist

- [ ] Login SSH key diuji sebelum password login dinonaktifkan.
- [ ] Firewall hanya membuka port yang diperlukan.
- [ ] Docker dan Compose berasal dari sumber resmi.
- [ ] `.env` permission `600` dan tidak masuk image/repo.
- [ ] Semua image dipin ke versi atau digest.
- [ ] Healthcheck dan `restart: unless-stopped` aktif.
- [ ] PostgreSQL dan Redis tidak mempublikasikan port.
- [ ] Stack pulih setelah Docker direstart.
- [ ] Prosedur rollback tertulis dan dapat dijalankan.
