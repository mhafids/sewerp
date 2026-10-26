# Standar DevOps & Infrastruktur SewERP

Dokumen ini memuat panduan deployment, konfigurasi lingkungan, arsitektur container Docker, strategi backup, dan otomatisasi CI/CD untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas mengonfigurasi Dockerfile, menyusun orkestrasi Docker Compose, mengatur reverse proxy Nginx, MinIO S3 storage, atau skrip backup database:
- `troubleshooting`: Mendiagnosis kendala startup container, kegagalan healthcheck Docker, dan troubleshooting koneksi reverse proxy Nginx ke Go binary.
- `autonomous-investigation`: Meneliti praktik terbaik hardening infrastruktur, efisiensi ukuran container multi-stage (Distroless < 30MB), dan otomasi rotasi backup `pg_dump`.
- `infosec`: Memverifikasi keamanan SSL/TLS Nginx, isolasi jaringan bridge internal Docker, larangan root privileges pada container, dan pengamanan kredensial MinIO/S3.

---

## 1. Arsitektur Infrastruktur & Deployment

SewERP dirancang agar dapat dijalankan dengan mudah pada VPS tunggal (skala pabrik mandiri) maupun kluster cloud (skala multi-pabrik enterprise):

```text
       [ Klien Browser / Mobile Tablet Pabrik ]
                         │
                         ▼
             [ Nginx / Cloudflare Proxy ]
             (SSL Termination, Rate Limit)
                         │
         ┌───────────────┴───────────────┐
         ▼                               ▼
[ Frontend Static SPA ]         [ Backend Go API ]
(Nginx Alpine / Pages)          (Distroless Binary)
                                         │
                         ┌───────────────┴───────────────┐
                         ▼                               ▼
               [ PostgreSQL 16 ]                 [ MinIO / S3 ]
            (Transaksional & River)           (Tech Pack, File Pola,
                                                PDF SPK & Surat Jalan)
```

---

## 2. Containerisasi Docker Multi-Stage

### Backend Dockerfile (`backend/Dockerfile`)
Menggunakan build multi-stage Go dengan image akhir berbasis Google Distroless untuk keamanan dan ukuran image minimal (< 30 MB):
```dockerfile
# Stage 1: Build
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/bin/sewerp-api ./cmd/api

# Stage 2: Runtime Minimal
FROM gcr.io/distroless/static-debian12:nonroot
WORKDIR /app
COPY --from=builder /app/bin/sewerp-api /app/sewerp-api
EXPOSE 8080
USER nonroot:nonroot
ENTRYPOINT ["/app/sewerp-api"]
```

### Frontend Dockerfile (`frontend/Dockerfile`)
Build SolidJS via Bun/Vite dan sajikan menggunakan Nginx Alpine unprivileged:
```dockerfile
# Stage 1: Build Frontend
FROM oven/bun:latest AS builder
WORKDIR /app
COPY package.json bun.lockb* ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build

# Stage 2: Web Server
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 3. Konfigurasi Lingkungan (12-Factor App)

Seluruh konfigurasi disuntikkan melalui *Environment Variables*. Nilai rahasia dilarang keras di-hardcode ke repositori:

| Variabel | Deskripsi | Contoh Nilai |
| :--- | :--- | :--- |
| `APP_ENV` | Lingkungan aplikasi | `development`, `staging`, `production` |
| `HTTP_PORT` | Port binding backend | `8080` |
| `DATABASE_URL` | Koneksi PostgreSQL | `postgres://user:pass@localhost:5432/sewerp?sslmode=disable` |
| `JWT_SECRET` | Kunci rahasia otentikasi token | `supersecretkey_min_32_characters_long` |
| `S3_ENDPOINT` | Host Object Storage | `minio:9000` atau `s3.ap-southeast-1.amazonaws.com` |
| `S3_BUCKET_DOCS` | Bucket penyimpanan berkas | `sewerp-production-assets` |
| `WA_GATEWAY_URL` | Webhook URL integrasi WhatsApp | `https://api.wagateway.local/v1/send` |

---

## 4. Prosedur Backup & Ketahanan Data

1. **Daily Database Backup (`pg_dump`)**:
   - Menjalankan cron job harian pada jam 02:00 dini hari (waktu sepi pabrik).
   - Mengompresi dump PostgreSQL dan mengenkripsinya dengan GPG sebelum dikirim ke offsite cold-storage (S3 Glacier / secondary server).
2. **Object Storage Replication**:
   - Berkas tech pack pola baju (`.dxf`, `.pdf`) dan template cetak disalin secara berkala antar bucket.

---

## 5. Health Check & Observabilitas

Backend menyediakan dua endpoint standar orkestrasi:
- `GET /healthz`: Memeriksa apakah proses aplikasi berjalan (liveness probe).
- `GET /readyz`: Memeriksa konektivitas aktif ke PostgreSQL dan MinIO Object Storage (readiness probe).

---

## 6. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur Sistem: [architecture.md](./architecture.md)
- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🗄️ Standar Database: [dbadmin.md](./dbadmin.md)
- 🧪 Standar Pengujian & QA: [qa.md](./qa.md)
- 🔧 Kebutuhan Teknis Sistem: [technical_requirements.md](./technical_requirements.md)
