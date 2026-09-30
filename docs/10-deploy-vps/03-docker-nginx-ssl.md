# 03 — Docker, Nginx, dan SSL Manual

## Tujuan

Alternatif tanpa panel: menjalankan app dengan Docker, reverse proxy Nginx, dan
sertifikat Let's Encrypt.

## Dockerfile Next.js (standalone)

Aktifkan `output: "standalone"` di `next.config.js`.

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
EXPOSE 3000
CMD ["node", "server.js"]
```

## docker-compose

```yaml
services:
  web:
    build: .
    restart: unless-stopped
    env_file: .env
    ports:
      - "127.0.0.1:3000:3000"
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Port app hanya di-bind ke `127.0.0.1` agar tidak terbuka langsung ke internet.
Database tidak diekspos ke publik.

Jalankan:

```bash
docker compose up -d --build
docker compose logs -f web
```

## Nginx Reverse Proxy

`/etc/nginx/sites-available/app.conf`:

```nginx
server {
    listen 80;
    server_name domain-kamu.com www.domain-kamu.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Aktifkan:

```bash
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## SSL Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d domain-kamu.com -d www.domain-kamu.com
```

Arahkan DNS ke IP server sebelum menjalankan certbot. Uji perpanjangan:

```bash
sudo certbot renew --dry-run
```

## Update Rilis

```bash
git pull
docker compose up -d --build
docker compose logs -f web
```

Rollback: checkout commit sebelumnya lalu build ulang. Simpan backup database
sebelum rilis yang mengubah skema.

## Checklist

- [ ] Container berjalan dan restart otomatis
- [ ] Port app tidak terbuka publik
- [ ] Nginx proxy benar
- [ ] HTTPS aktif dan auto-renew teruji
- [ ] Prosedur update dan rollback jelas
