# KomikMangaku Mirror Proxy

Full mirror proxy untuk `komik.mangaku.asia` dengan perbaikan SEO otomatis. Bisa di-deploy ke **Railway**, **Render**, atau **VPS pribadi**.

## Fitur SEO yang Diperbaiki

| Masalah GSC | Solusi |
|---|---|
| **Duplikat / Kanonis berbeda** | Semua `<link rel="canonical">` di-rewrite ke domain mirror |
| **Tidak ditemukan (404)** | Redirect internal di-follow otomatis, 404 dari origin diteruskan dengan benar |
| **Halaman dengan pengalihan** | Redirect dari origin di-follow & di-rewrite ke domain mirror |
| **Data terstruktur Breadcrumb** | JSON-LD BreadcrumbList di-parse & URL di-fix ke domain mirror |
| **Data terstruktur tidak dapat diurai** | Semua JSON-LD di-parse ulang, URL di-fix, format diperbaiki |

## Fitur Tambahan

- Rewrite semua URL di HTML (href, src, srcset, data-src, dll)
- Rewrite Open Graph & Twitter Card meta tags
- Rewrite inline CSS & JavaScript yang mengandung URL origin
- Rewrite structured data (JSON-LD) secara mendalam (deep)
- In-memory cache untuk performa
- Gzip compression
- Rate limiting
- ETag untuk menghindari duplicate content
- Proper `robots.txt` & `x-robots-tag`
- Health check endpoint (`/health`)

## Environment Variables

| Variable | Default | Keterangan |
|---|---|---|
| `ORIGIN_HOST` | `komik.mangaku.asia` | Hostname situs origin |
| `ORIGIN_PROTOCOL` | `https` | Protocol origin |
| `MIRROR_DOMAIN` | *(auto dari Host header)* | **Domain mirror kamu (WAJIB diisi)** |
| `PORT` | `3000` | Port server |
| `CACHE_TTL` | `300` | Cache TTL dalam detik |
| `MAX_CACHE_ENTRIES` | `500` | Maks entry cache di memory |
| `RATE_LIMIT` | `200` | Request limit per IP per 15 menit |

## Deploy ke Railway

1. Push repo ini ke GitHub
2. Buka [railway.app](https://railway.app) → New Project → Deploy from GitHub
3. Set environment variable `MIRROR_DOMAIN` = domain kamu
4. Railway otomatis detect Node.js dan deploy

Atau pakai CLI:
```bash
npm i -g @railway/cli
railway login
railway init
railway up
```

## Deploy ke Render

1. Push repo ini ke GitHub
2. Buka [render.com](https://render.com) → New Web Service → Connect repo
3. Render akan pakai `render.yaml` otomatis
4. Set `MIRROR_DOMAIN` di environment variables

## Deploy ke VPS (Ubuntu/Debian)

```bash
# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Clone & setup
git clone https://github.com/Mpratama260304/komikmangaku.git
cd komikmangaku
npm install

# Set environment
export MIRROR_DOMAIN=yourdomain.com
export PORT=3000

# Run dengan PM2 (production)
npm install -g pm2
pm2 start server.js --name komikmangaku
pm2 save
pm2 startup
```

### Nginx Reverse Proxy (opsional, untuk VPS)

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Deploy dengan Docker

```bash
docker build -t komikmangaku .
docker run -d -p 3000:3000 \
  -e MIRROR_DOMAIN=yourdomain.com \
  -e ORIGIN_HOST=komik.mangaku.asia \
  --name komikmangaku \
  komikmangaku
```

## Testing Lokal

```bash
npm install
MIRROR_DOMAIN=localhost:3000 node server.js
# Buka http://localhost:3000
```