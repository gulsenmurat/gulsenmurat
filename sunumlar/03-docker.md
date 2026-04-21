---
title: "Docker: Konteyner Teknolojisi"
author: "Gülşen Murat"
date: "2024"
---

# 🐳 Docker

**Konteyner Teknolojisi: Kurulum, Kullanım ve İyi Pratikler**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Docker Nedir?

### Docker

2013'te açık kaynak olarak yayınlanan **konteyner tabanlı sanallaştırma** platformu.

```
Geleneksel Sanallaştırma          Docker (Konteynerler)
────────────────────────          ─────────────────────
┌──────┐ ┌──────┐ ┌──────┐       ┌──────┐ ┌──────┐ ┌──────┐
│App A │ │App B │ │App C │       │App A │ │App B │ │App C │
├──────┤ ├──────┤ ├──────┤       ├──────┤ ├──────┤ ├──────┤
│Guest │ │Guest │ │Guest │       │Libs  │ │Libs  │ │Libs  │
│OS    │ │OS    │ │OS    │       └──────┘ └──────┘ └──────┘
├──────┴─┴──────┴─┴──────┤       ┌────────────────────────┐
│     Hypervisor          │       │     Docker Engine       │
├────────────────────────┤       ├────────────────────────┤
│     Host OS             │       │     Host OS (Linux)     │
├────────────────────────┤       ├────────────────────────┤
│     Donanım             │       │     Donanım             │
└────────────────────────┘       └────────────────────────┘
Ağır (GB), Yavaş başlangıç       Hafif (MB), Saniyeler içinde başlar
```

---

## Slayt 2 — Temel Kavramlar

### Görüntü (Image)

Uygulamanın **salt okunur şablonu**. Dockerfile ile oluşturulur.

### Konteyner (Container)

Image'dan oluşturulan **çalışan örnek**. Bir image'dan N tane container açılabilir.

### Kayıt Defteri (Registry)

Image'ların saklandığı depo. En popüleri: **Docker Hub** (hub.docker.com)

### Katmanlı Yapı

```
┌──────────────────────┐  ← Uygulama katmanı (nginx.conf)
├──────────────────────┤  ← Bağımlılık katmanı (apt install curl)
├──────────────────────┤  ← Temel uygulama (nginx kurulum)
└──────────────────────┘  ← Temel image (ubuntu:22.04)
```

Her `RUN`, `COPY`, `ADD` komutu yeni bir katman oluşturur. Katmanlar **önbelleğe** alınır.

---

## Slayt 3 — Kurulum (Ubuntu/Debian)

### Eski Docker Sürümlerini Temizle

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

### Resmi Docker Deposunu Ekle

```bash
# Bağımlılıklar
sudo apt install ca-certificates curl gnupg lsb-release -y

# GPG anahtarı
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
    | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Depoyu ekle
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
    | sudo tee /etc/apt/sources.list.d/docker.list
```

### Docker'ı Kur

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

---

## Slayt 4 — Kurulum Sonrası Ayarlar

### Servisi Etkinleştir

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

### Kurulumu Doğrula

```bash
sudo docker run hello-world
# "Hello from Docker!" mesajı görünmeli
```

### Sudo Olmadan Kullan

```bash
# Kullanıcını docker grubuna ekle
sudo usermod -aG docker $USER

# Oturumu yeniden aç veya:
newgrp docker

# Test
docker run hello-world   # sudo olmadan
```

---

## Slayt 5 — Temel Docker Komutları

### Image Yönetimi

```bash
docker pull nginx                   # Image indir
docker pull nginx:1.25              # Belirli sürüm
docker images                       # Yerel image'ları listele
docker rmi nginx                    # Image sil
docker image prune                  # Kullanılmayan image'ları temizle
docker image prune -a               # Tüm kullanılmayan image'ları temizle
```

### Konteyner Çalıştırma

```bash
docker run nginx                            # Ön planda çalıştır (Ctrl+C ile dur)
docker run -d nginx                         # Arka planda çalıştır (detached)
docker run -d -p 8080:80 nginx             # Port yönlendir (host:konteyner)
docker run -d --name benim-nginx nginx      # İsim ver
docker run -it ubuntu bash                  # Etkileşimli terminal
docker run --rm ubuntu echo "Merhaba"       # Çıkınca sil
```

---

## Slayt 6 — Konteyner Yönetimi

```bash
# Listeleme
docker ps                   # Çalışan konteynerler
docker ps -a                # Tüm konteynerler (durdurulmuş dahil)

# Kontrol
docker start benim-nginx    # Başlat
docker stop benim-nginx     # Durdur (SIGTERM)
docker kill benim-nginx     # Zorla durdur (SIGKILL)
docker restart benim-nginx  # Yeniden başlat
docker rm benim-nginx       # Sil (önce durdurulmuş olmalı)
docker rm -f benim-nginx    # Zorla sil (çalışıyorsa da)

# İnceleme
docker logs benim-nginx             # Log göster
docker logs -f benim-nginx          # Canlı log izle
docker logs --tail 50 benim-nginx   # Son 50 satır
docker exec -it benim-nginx bash    # Konteyner içine gir
docker inspect benim-nginx          # Detaylı JSON bilgi
docker stats                        # Kaynak kullanımı (CPU, RAM)
```

---

## Slayt 7 — Dockerfile Yazmak

### Temel Örnek: Node.js Uygulaması

```dockerfile
# Temel image seç
FROM node:18-alpine

# Çalışma dizini oluştur
WORKDIR /app

# Bağımlılık dosyalarını kopyala (önbellek optimizasyonu)
COPY package*.json ./

# Bağımlılıkları yükle
RUN npm ci --only=production

# Uygulama dosyalarını kopyala
COPY . .

# Port belirt (sadece belgeleme amaçlı)
EXPOSE 3000

# Konteyner başladığında çalışacak komut
CMD ["node", "server.js"]
```

### Build Et ve Çalıştır

```bash
docker build -t benim-uygulama:1.0 .
docker run -d -p 3000:3000 benim-uygulama:1.0
```

---

## Slayt 8 — Dockerfile En İyi Pratikler

### 1. Küçük Base Image Kullan

```dockerfile
# ❌ Kaçın
FROM ubuntu:22.04   # ~77 MB

# ✅ Tercih et
FROM alpine:3.18    # ~7 MB
FROM node:18-alpine # ~180 MB (node+alpine)
```

### 2. Katman Sayısını Azalt

```dockerfile
# ❌ Her komut yeni katman
RUN apt update
RUN apt install curl -y
RUN apt install git -y

# ✅ Birleştir
RUN apt update && apt install -y curl git \
    && rm -rf /var/lib/apt/lists/*
```

### 3. .dockerignore Kullan

```
# .dockerignore
node_modules/
.git/
*.log
.env
README.md
```

### 4. Non-Root Kullanıcı

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

---

## Slayt 9 — Volume (Veri Kalıcılığı)

Konteyner silinince veriler kaybolur. **Volume** ile veriyi kalıcı yap.

```bash
# Adlandırılmış volume oluştur
docker volume create postgres-data

# Volume ile çalıştır
docker run -d \
  --name postgres-db \
  -v postgres-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=sifre123 \
  postgres:15

# Volume listele
docker volume ls

# Volume bilgisi
docker volume inspect postgres-data

# Bind mount (yerel dizin ↔ konteyner)
docker run -d \
  -v /home/gulsenmurat/web:/var/www/html \
  -p 80:80 \
  nginx
```

---

## Slayt 10 — Network (Ağ)

```bash
# Ağ listele
docker network ls

# Özel ağ oluştur
docker network create benim-ag

# Ağa bağlı konteyner çalıştır
docker run -d --name web --network benim-ag nginx
docker run -d --name db  --network benim-ag postgres

# Aynı ağdaki konteynerler, adıyla birbirini bulabilir
# web konteyneri içinden: ping db → çalışır!

# Ağ detayları
docker network inspect benim-ag

# Konteynerden ağı ayır
docker network disconnect benim-ag web
```

---

## Slayt 11 — Docker Compose

Birden fazla konteyneri tek YAML dosyasıyla yönet.

### docker-compose.yml

```yaml
version: "3.9"

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - app
    restart: unless-stopped

  app:
    build: ./backend        # Dockerfile'dan build et
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=güvenli_sifre
    restart: unless-stopped

volumes:
  postgres-data:
```

---

## Slayt 12 — Docker Compose Komutları

```bash
# Servisleri başlat
docker compose up -d            # Arka planda başlat
docker compose up --build       # Önce build et, sonra başlat

# Durumu kontrol et
docker compose ps               # Servis durumları
docker compose logs             # Tüm loglar
docker compose logs -f web      # web servisi canlı log

# Servis yönetimi
docker compose stop             # Durdur (silme)
docker compose start            # Tekrar başlat
docker compose restart web      # Sadece web servisini yeniden başlat

# Temizlik
docker compose down             # Durdur + konteynerleri sil
docker compose down -v          # + Volume'ları da sil

# Çalışan servise komut gönder
docker compose exec web bash
docker compose exec db psql -U admin mydb
```

---

## Slayt 13 — Docker Hub'a Image Gönderme

```bash
# Docker Hub'a giriş yap
docker login

# Image'ı etiketle (tag)
# Format: kullaniciadi/image-adi:sürüm
docker tag benim-uygulama:1.0 gulsenmurat/benim-uygulama:1.0
docker tag benim-uygulama:1.0 gulsenmurat/benim-uygulama:latest

# Push et
docker push gulsenmurat/benim-uygulama:1.0
docker push gulsenmurat/benim-uygulama:latest

# Başka sunucudan çek
docker pull gulsenmurat/benim-uygulama:1.0
```

### Özel Registry (Opsiyonel)

```bash
# Kendi registry'ini çalıştır
docker run -d -p 5000:5000 --name registry registry:2

# Push et
docker tag benim-uygulama localhost:5000/benim-uygulama
docker push localhost:5000/benim-uygulama
```

---

## Slayt 14 — Güvenlik İpuçları

### 1. Image'ı Güncel Tut

```bash
docker pull nginx:alpine        # Her zaman güncel base image
docker scout cves nginx:alpine  # CVE taraması (docker scout kurulu ise)
```

### 2. Ortam Değişkenlerini .env ile Yönet

```bash
# .env dosyası
DB_PASSWORD=güvenli_sifre

# docker-compose.yml
env_file:
  - .env
```

### 3. Read-Only Dosya Sistemi

```bash
docker run --read-only -v /tmp nginx
```

### 4. Kaynak Sınırla

```bash
docker run -d \
  --memory="512m" \
  --cpus="1.0" \
  nginx
```

---

## Slayt 15 — Sistem Temizliği

```bash
# Ne kadar yer kaplıyor?
docker system df

# Kullanılmayan her şeyi temizle
docker system prune              # Durdurulmuş konteynerler, ağlar, image'lar
docker system prune -a           # Çalışmayan tüm image'ları da sil
docker system prune -a --volumes # + Volume'ları da sil

# Tek tek temizlik
docker container prune           # Durdurulmuş konteynerler
docker image prune               # Etiketlenmemiş (dangling) image'lar
docker volume prune              # Kullanılmayan volume'lar
docker network prune             # Kullanılmayan ağlar

# Belirli bir konteyner ve volume'unu sil
docker rm -v eski-konteyner
```

---

## Slayt 16 — Özet

### Öğrendiklerimiz ✅

- Docker nedir, VM'den farkı nedir?
- Image, Container, Registry, Volume kavramları
- Ubuntu/Debian ve CentOS kurulumu
- Temel docker komutları (run, ps, logs, exec)
- Dockerfile yazımı ve katman optimizasyonu
- Volume ile veri kalıcılığı
- Docker Compose ile çok servisli uygulama
- Docker Hub'a image gönderme
- Güvenlik ve temizlik pratikleri

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Docker Dokümantasyonu | https://docs.docker.com |
| Docker Hub | https://hub.docker.com |
| Play with Docker (tarayıcıda dene) | https://labs.play-with-docker.com |
| Docker Compose Referansı | https://docs.docker.com/compose/compose-file/ |
| Dockerfile Best Practices | https://docs.docker.com/develop/develop-images/dockerfile_best-practices/ |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
