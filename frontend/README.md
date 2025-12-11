# 🚀 Vue + Traefik Docker Deployment

โปรเจค Vue.js พร้อม Traefik Reverse Proxy สำหรับ deploy ทั้ง Development และ Production

## 📁 โครงสร้างไฟล์

```
├── docker-compose.yml          # Base config สำหรับ dev
├── docker-compose.override.yml # Dev settings (auto merge)
├── docker-compose.prod.yml     # Production config
├── Dockerfile                  # Multi-stage build (Node → Nginx)
├── nginx.conf                  # Nginx config สำหรับ SPA
├── .dockerignore               # ไฟล์ที่ไม่ต้อง copy ตอน build
└── src/                        # Vue source code
```

---

## 🖥️ Development

### รันในเครื่อง

```bash
docker-compose up -d --build
```

### เข้าเว็บ

| URL | คำอธิบาย |
|-----|----------|
| http://vue.localhost | Vue App |
| http://localhost:8080 | Traefik Dashboard |

### หยุด Container

```bash
docker-compose down
```

### ดู Logs

```bash
docker-compose logs -f vue-app
```

---

## 🌐 Production Deployment

### ขั้นตอนที่ 1: เตรียม Server

ติดตั้ง Docker และ Docker Compose บน Server:

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y docker.io docker-compose

# เริ่ม Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### ขั้นตอนที่ 2: Clone โปรเจค

```bash
git clone <your-repo-url>
cd <project-folder>
```

### ขั้นตอนที่ 3: แก้ไข Domain

แก้ไขไฟล์ `docker-compose.prod.yml` บรรทัด 40:

```yaml
# เปลี่ยนเป็นโดเมนของคุณ
- "traefik.http.routers.vue-app.rule=Host(`yourdomain.com`) || Host(`www.yourdomain.com`)"
```

### ขั้นตอนที่ 4: รัน Production

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

### ตรวจสอบสถานะ

```bash
# ดู container ที่รันอยู่
docker-compose -f docker-compose.prod.yml ps

# ดู logs
docker-compose -f docker-compose.prod.yml logs -f
```

---

## ☁️ ตั้งค่า Cloudflare + HTTPS

### วิธีที่ 1: Cloudflare Proxy (แนะนำ ✅)

Cloudflare จะจัดการ SSL ให้อัตโนมัติ

#### 1. ตั้งค่า DNS ใน Cloudflare Dashboard

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | @ | `IP_SERVER` | ☁️ Proxied |
| A | www | `IP_SERVER` | ☁️ Proxied |

#### 2. ตั้งค่า SSL/TLS

ไปที่ **SSL/TLS** → เลือก **Full** หรือ **Full (Strict)**

#### 3. รัน Production

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

### วิธีที่ 2: Let's Encrypt (ไม่ใช้ Cloudflare)

ถ้าต้องการให้ Traefik ขอ SSL cert เอง:

1. แก้ไข `docker-compose.prod.yml` โดย uncomment บรรทัดเหล่านี้:

```yaml
# ใน traefik > command:
- "--certificatesresolvers.letsencrypt.acme.email=your-email@example.com"
- "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
- "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"

# ใน traefik > volumes:
- ./letsencrypt:/letsencrypt

# ใน vue-app > labels:
- "traefik.http.routers.vue-app.tls.certresolver=letsencrypt"
```

2. สร้างโฟลเดอร์สำหรับเก็บ cert:

```bash
mkdir letsencrypt
```

3. รัน Production

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

---

## 🔧 คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker-compose up -d --build` | รัน dev |
| `docker-compose down` | หยุด containers |
| `docker-compose logs -f` | ดู logs แบบ realtime |
| `docker-compose ps` | ดูสถานะ containers |
| `docker-compose exec vue-app sh` | เข้า shell ใน container |
| `docker system prune -a` | ลบ images/containers ที่ไม่ใช้ |

---

## 📊 เปรียบเทียบ Dev vs Production

| ส่วน | Development | Production |
|------|-------------|------------|
| Domain | `vue.localhost` | `yourdomain.com` |
| HTTPS | ❌ | ✅ |
| Dashboard | ✅ port 8080 | ❌ ปิด |
| Restart Policy | ❌ | ✅ always |
| HTTP→HTTPS | ❌ | ✅ redirect |

---

## 🛡️ Security Checklist (Production)

- [ ] เปลี่ยน domain ใน `docker-compose.prod.yml`
- [ ] ตั้งค่า Cloudflare SSL เป็น Full/Full (Strict)
- [ ] ไม่เปิด Traefik Dashboard (`--api.insecure=true` ถูกลบแล้ว)
- [ ] ใช้ `.env` สำหรับค่า sensitive (ไม่ hardcode)
- [ ] ตั้งค่า Firewall เปิดแค่ port 80, 443

---

## 🐛 Troubleshooting

### Container ไม่ start

```bash
# ดู logs
docker-compose logs traefik
docker-compose logs vue-app
```

### Port 80/443 ถูกใช้งานอยู่

```bash
# ตรวจสอบ port ที่ใช้งาน
sudo lsof -i :80
sudo lsof -i :443

# หยุด service ที่ใช้ port
sudo systemctl stop nginx
sudo systemctl stop apache2
```

### เว็บโหลดไม่ขึ้น

1. ตรวจสอบ DNS ชี้มาที่ IP ถูกต้อง
2. ตรวจสอบ Cloudflare Proxy เปิดอยู่
3. ตรวจสอบ SSL/TLS mode ใน Cloudflare

---

## 📝 License

MIT
