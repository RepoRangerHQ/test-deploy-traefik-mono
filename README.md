# 🚀 Vue + Hono + Traefik Docker Deployment

Fullstack project พร้อม Traefik Reverse Proxy

| Service | Tech Stack | Port |
|---------|-----------|------|
| Frontend | Vue.js + Vite + Nginx | 3000 |
| Backend | Hono + Node.js | 4000 |
| Proxy | Traefik | 80/443 |

---

## 📁 โครงสร้างโปรเจค

```
├── frontend/               # Vue.js App
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── .dockerignore
│   └── src/
├── backend/                # Hono API
│   ├── Dockerfile
│   ├── .dockerignore
│   └── src/
├── docker-compose.yml      # Development
├── docker-compose.prod.yml # Production
└── README.md
```

---

## 🖥️ Development

### รันทั้งระบบ

```bash
docker-compose up -d --build
```

### เข้าเว็บ

| URL | คำอธิบาย |
|-----|----------|
| http://vue.localhost | Frontend (Vue) |
| http://api.localhost | Backend (Hono API) |
| http://localhost:8080 | Traefik Dashboard |

### หยุด Containers

```bash
docker-compose down
```

### ดู Logs

```bash
# ดูทั้งหมด
docker-compose logs -f

# ดูเฉพาะ service
docker-compose logs -f frontend
docker-compose logs -f backend
```

### Build ใหม่เฉพาะ service

```bash
docker-compose up -d --build frontend
docker-compose up -d --build backend
```

---

## 🌐 Production Deployment

### ขั้นตอนที่ 1: แก้ไข Domain

แก้ไขไฟล์ `docker-compose.prod.yml`:

```yaml
# Frontend (บรรทัด ~36)
- "traefik.http.routers.frontend.rule=Host(`yourdomain.com`)"

# Backend (บรรทัด ~54)
- "traefik.http.routers.backend.rule=Host(`api.yourdomain.com`)"
```

### ขั้นตอนที่ 2: รัน Production

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

### ตรวจสอบสถานะ

```bash
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs -f
```

---

## ☁️ Cloudflare + HTTPS

### ตั้งค่า DNS ใน Cloudflare

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | @ | `IP_SERVER` | ☁️ Proxied |
| A | www | `IP_SERVER` | ☁️ Proxied |
| A | api | `IP_SERVER` | ☁️ Proxied |

### ตั้งค่า SSL/TLS

ไปที่ **SSL/TLS** → เลือก **Full** หรือ **Full (Strict)**

---

## 🔧 คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker-compose up -d --build` | รัน dev |
| `docker-compose -f docker-compose.prod.yml up -d --build` | รัน prod |
| `docker-compose down` | หยุด containers |
| `docker-compose logs -f` | ดู logs |
| `docker-compose ps` | ดูสถานะ |
| `docker system prune -a` | ลบ images ที่ไม่ใช้ |

---

## 📊 เปรียบเทียบ Dev vs Production

| ส่วน | Development | Production |
|------|-------------|------------|
| Frontend URL | `vue.localhost` | `yourdomain.com` |
| Backend URL | `api.localhost` | `api.yourdomain.com` |
| HTTPS | ❌ | ✅ |
| Dashboard | ✅ port 8080 | ❌ ปิด |
| Restart Policy | ❌ | ✅ always |

---

## 🛡️ Security Checklist (Production)

- [ ] เปลี่ยน domain ใน `docker-compose.prod.yml`
- [ ] ตั้งค่า Cloudflare SSL เป็น Full/Full (Strict)
- [ ] ตั้งค่า CORS ใน backend ให้รับเฉพาะ domain ที่ต้องการ
- [ ] ตั้งค่า Firewall เปิดแค่ port 80, 443

---

## 🐛 Troubleshooting

### Container ไม่ start

```bash
docker-compose logs traefik
docker-compose logs frontend
docker-compose logs backend
```

### เว็บโหลดไม่ขึ้น

1. ตรวจสอบ DNS ชี้มาที่ IP ถูกต้อง
2. ตรวจสอบ Cloudflare Proxy เปิดอยู่
3. ตรวจสอบ SSL/TLS mode ใน Cloudflare

### API ติดต่อไม่ได้

1. ตรวจสอบ CORS settings ใน backend
2. ตรวจสอบ URL ที่ frontend เรียก API

---

## 📝 License

MIT

