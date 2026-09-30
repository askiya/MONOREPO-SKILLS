# 01 — Setup VPS dari Nol

## Tujuan

VPS siap pakai: aman, punya Docker, dan bisa menjalankan aplikasi produksi.

## Pilih VPS

| Provider | Catatan |
|---|---|
| Hetzner, Contabo | murah, Eropa |
| DigitalOcean, Linode, Vultr | dokumentasi banyak |
| Biznet, IDCloudHost, Niagahoster | server Indonesia, latensi rendah |
| Tencent/Alibaba via reseller | cek dukungan dan lokasi region |

Spesifikasi minimum realistis untuk satu app + PostgreSQL: 2 vCPU, 4 GB RAM,
40 GB SSD. 1 GB RAM sering kehabisan memori saat build.

OS: Ubuntu LTS.

## Langkah Awal (urut, jangan dilompati)

Bagian ini mengubah akses server. Kalau salah urutan, kamu bisa terkunci dari
VPS sendiri. Lakukan satu per satu dan jangan tutup sesi SSH pertama sebelum
sesi kedua terbukti berhasil.

### 1. Login pertama

```bash
ssh root@IP_SERVER
```

### 2. Update sistem

```bash
apt update && apt upgrade -y
```

### 3. Buat user non-root

```bash
adduser deploy
usermod -aG sudo deploy
```

### 4. Pasang SSH key untuk user baru

Di komputer lokal:

```bash
ssh-keygen -t ed25519 -C "deploy@project"
ssh-copy-id deploy@IP_SERVER
```

Uji di terminal **baru**:

```bash
ssh deploy@IP_SERVER
```

Jangan lanjut kalau login ini belum berhasil.

### 5. Matikan login root dan password

Edit `/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
```

Terapkan:

```bash
sudo systemctl restart ssh
```

Pastikan sesi lama tetap terbuka sampai login baru diverifikasi.

### 6. Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

Jangan aktifkan UFW sebelum port SSH diizinkan.

### 7. Fail2ban dan swap

```bash
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

Swap 2 GB bila RAM kecil:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 8. Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker deploy
```

Logout dan login lagi, lalu uji:

```bash
docker run --rm hello-world
```

## Checklist

- [ ] User non-root dengan SSH key berfungsi
- [ ] Login root/password dimatikan setelah key diuji
- [ ] UFW aktif dengan SSH diizinkan
- [ ] Fail2ban aktif
- [ ] Swap ada bila RAM kecil
- [ ] Docker berjalan
