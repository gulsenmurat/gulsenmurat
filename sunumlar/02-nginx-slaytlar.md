---
title: "Nginx: Kurulum ve Yapılandırma"
author: "Gülşen Murat"
date: "2024"
---

# 🚀 Nginx

**Web Sunucusu, Reverse Proxy ve Load Balancer**

> Hazırlayan: Gülşen Murat  
> 📄 Detaylı teknik referans: `nginx-sunum.md`

---

## Slayt 1 — Nginx Nedir?

### Nginx (engine-x)

Igor Sysoev tarafından 2004'te geliştirildi. Bugün dünyanın en yaygın web sunucularından biri.

```
Kullanım Alanları:
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Web Sunucusu  │  │  Reverse Proxy  │  │ Load Balancer   │
│   (HTTP/HTTPS)  │  │  (Node, Python) │  │ (Yük Dağıtımı) │
└─────────────────┘  └─────────────────┘  └─────────────────┘
┌─────────────────┐  ┌─────────────────┐
│   HTTP Cache    │  │  Mail Proxy     │
│   (Önbellekleme)│  │  (SMTP/IMAP)   │
└─────────────────┘  └─────────────────┘
```

### Neden Nginx?

- ⚡ 10.000+ eş zamanlı bağlantı — düşük bellek ile
- 📁 Statik dosya servisinde rakipsiz hız
- 🔄 Uygulama sunucuları önünde mükemmel proxy
- 🌍 Dünya genelinde %34 pazar payı (Netcraft 2024)

---

## Slayt 2 — Nasıl Çalışır? (Mimari)

### Olay Tabanlı (Event-Driven) Mimari

```
Apache (Thread/Process)          Nginx (Event-Driven)
─────────────────────            ────────────────────
İstek 1 → Process 1              İstek 1 ─┐
İstek 2 → Process 2              İstek 2 ─┤→ Worker 1 (epoll)
İstek 3 → Process 3              İstek 3 ─┤   → non-blocking I/O
İstek N → Process N              İstek N ─┘
(RAM tüketimi: yüksek)           (RAM tüketimi: çok düşük)
```

### Master – Worker Modeli

| Süreç | Sayı | Görev |
|-------|------|-------|
| **Master** | 1 | Yapılandırma okuma, worker yönetimi |
| **Worker** | CPU çekirdeği sayısı | Gerçek HTTP isteklerini işler |
| **Cache Manager** | 1 | Önbellek dosyalarını yönetir |

---

## Slayt 3 — Kurulum (Ubuntu/Debian)

### Adım 1: Sistemi Güncelle

```bash
sudo apt update && sudo apt upgrade -y
```

### Adım 2: Nginx Kur

```bash
sudo apt install nginx -y
```

### Adım 3: Servisi Başlat

```bash
sudo systemctl start nginx
sudo systemctl enable nginx     # Açılışta otomatik başlat
```

### Adım 4: Kontrol Et

```bash
sudo systemctl status nginx
curl http://localhost            # "Welcome to nginx!" görmelisin
```

### Adım 5: Güvenlik Duvarı

```bash
sudo ufw allow 'Nginx Full'     # 80 ve 443 portlarını aç
sudo ufw status
```

---

## Slayt 4 — Kurulum (CentOS/AlmaLinux)

```bash
# EPEL deposunu ekle
sudo dnf install epel-release -y

# Nginx kur
sudo dnf install nginx -y

# Servisi başlat
sudo systemctl start nginx
sudo systemctl enable nginx

# Güvenlik duvarı (firewalld)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# SELinux izni (reverse proxy için)
sudo setsebool -P httpd_can_network_connect 1
```

---

## Slayt 5 — Dizin Yapısı

```
/etc/nginx/
├── nginx.conf              ← Ana yapılandırma
├── conf.d/                 ← Ek .conf dosyaları (CentOS tarzı)
├── sites-available/        ← Tanımlı siteler (Ubuntu tarzı)
├── sites-enabled/          ← Aktif siteler (symlink)
└── snippets/               ← Yeniden kullanılabilir parçalar

/var/log/nginx/
├── access.log              ← Her HTTP isteği
└── error.log               ← Hatalar

/var/www/html/              ← Varsayılan web kökü
```

### Önemli Komutlar

```bash
sudo nginx -t               # Sözdizimi testi ✅
sudo systemctl reload nginx # Servisi durdurmadan güncelle
sudo nginx -V               # Sürüm + derlenmiş modüller
```

---

## Slayt 6 — nginx.conf Temel Yapısı

```nginx
# Kaç worker process açılsın? (auto = CPU çekirdeği sayısı)
worker_processes auto;

events {
    worker_connections 1024;   # Worker başına max bağlantı
    use epoll;                 # Linux'ta en verimli yöntem
}

http {
    include       mime.types;
    sendfile      on;           # Kernel seviyesinde dosya gönder
    keepalive_timeout 65;       # Keep-alive zaman aşımı
    server_tokens off;          # Nginx sürümünü gizle
    gzip on;                    # Yanıtları sıkıştır

    # Server blokları buraya gelir
    include /etc/nginx/sites-enabled/*;
}
```

> **Blok Hiyerarşisi:** `http` → `server` → `location`

---

## Slayt 7 — Sanal Sunucu (Virtual Host)

### /etc/nginx/sites-available/orneksite.com

```nginx
server {
    listen 80;
    server_name orneksite.com www.orneksite.com;

    root /var/www/orneksite.com;
    index index.html index.php;

    access_log /var/log/nginx/orneksite.access.log;
    error_log  /var/log/nginx/orneksite.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Siteyi Etkinleştir

```bash
# Sembolik bağlantı oluştur
sudo ln -s /etc/nginx/sites-available/orneksite.com \
           /etc/nginx/sites-enabled/

sudo nginx -t && sudo systemctl reload nginx
```

---

## Slayt 8 — SSL/TLS (Let's Encrypt)

### Certbot Kur

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Sertifika Al (Nginx otomatik yapılandırır)

```bash
sudo certbot --nginx -d orneksite.com -d www.orneksite.com
```

### Otomatik Yenileme

```bash
sudo certbot renew --dry-run    # Test
# Cron otomatik kurulur → /etc/cron.d/certbot
```

### Sonuç: Nginx Yapılandırması

```nginx
server {
    listen 443 ssl http2;
    ssl_certificate     /etc/letsencrypt/live/orneksite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orneksite.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    add_header Strict-Transport-Security "max-age=31536000" always;
}
```

---

## Slayt 9 — Reverse Proxy

Nginx'i Node.js, Python, Java gibi bir uygulama sunucusunun önüne koy:

```
İstemci → Nginx (:80/:443) → Node.js (:3000)
```

```nginx
server {
    listen 80;
    server_name api.orneksite.com;

    location / {
        proxy_pass http://localhost:3000;

        # İstemci bilgisini ilet
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket desteği
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## Slayt 10 — Load Balancer

```nginx
http {
    upstream uygulama_grubu {
        # Round-robin (varsayılan): sırayla dağıt
        server 10.0.0.1:3000 weight=3;   # 3x fazla trafik
        server 10.0.0.2:3000 weight=1;
        server 10.0.0.3:3000 backup;     # Diğerleri çökünce devreye girer
    }

    server {
        listen 80;
        location / {
            proxy_pass http://uygulama_grubu;
        }
    }
}
```

### Yük Dengeleme Algoritmaları

| Yöntem | Direktif | Kullanım |
|--------|----------|----------|
| Round-Robin | (varsayılan) | Genel amaçlı |
| En Az Bağlantı | `least_conn;` | Uzun süreli bağlantılar |
| IP Hash | `ip_hash;` | Oturum tutarlılığı |

---

## Slayt 11 — Rate Limiting (Hız Sınırlama)

```nginx
http {
    # Kural tanımla: IP başına saniyede 10 istek
    limit_req_zone $binary_remote_addr zone=genel:10m rate=10r/s;

    server {
        location /api/ {
            limit_req zone=genel burst=20 nodelay;
            limit_req_status 429;
            proxy_pass http://localhost:3000;
        }

        location /login {
            # Login için sıkı limit
            limit_req zone=genel burst=3;
        }
    }
}
```

### Diğer Güvenlik Başlıkları

```nginx
server_tokens off;                                        # Sürüm gizle
add_header X-Frame-Options "SAMEORIGIN" always;           # Clickjacking
add_header X-Content-Type-Options "nosniff" always;       # MIME sniffing
add_header X-XSS-Protection "1; mode=block" always;       # XSS
```

---

## Slayt 12 — Temel Komutlar Referansı

```bash
# Servis
sudo systemctl start|stop|restart|reload|status nginx

# Yapılandırma
sudo nginx -t               # Sözdizimi test et
sudo nginx -T               # Tüm yapılandırmayı göster

# Loglar
sudo tail -f /var/log/nginx/access.log      # Canlı erişim log
sudo tail -f /var/log/nginx/error.log       # Canlı hata log
sudo grep "404" /var/log/nginx/access.log   # 404 hatalarını filtrele

# IP başına istek sayısı
sudo awk '{print $1}' /var/log/nginx/access.log \
    | sort | uniq -c | sort -nr | head -10
```

---

## Slayt 13 — Sık Karşılaşılan Hatalar

| Hata | Sebep | Çözüm |
|------|-------|-------|
| **502 Bad Gateway** | Arka uç uygulama çalışmıyor | `systemctl status php-fpm` / `ss -tlnp \| grep 3000` |
| **403 Forbidden** | Dosya izinleri hatalı | `chown -R www-data /var/www/` |
| **413 Too Large** | Dosya boyutu sınırı aşıldı | `client_max_body_size 50M;` ekle |
| **Port 80 meşgul** | Başka uygulama (Apache) çalışıyor | `sudo lsof -i :80` ile bul, durdur |
| **Sözdizimi hatası** | nginx.conf yazım yanlışı | `sudo nginx -t` ile satır numarası öğren |

---

## Slayt 14 — Özet

### Öğrendiklerimiz ✅

- Nginx'in olay tabanlı mimarisi neden daha verimli?
- Ubuntu ve CentOS/AlmaLinux kurulum adımları
- nginx.conf yapısı ve temel direktifler
- Virtual Host ile birden fazla site yönetimi
- Let's Encrypt ile ücretsiz SSL sertifikası
- Reverse proxy ve WebSocket desteği
- Load balancer ile yük dağıtımı
- Rate limiting ile DDoS koruması
- Yaygın hataları okuma ve çözme

### Sonraki Adım

📄 Daha fazla detay için: `nginx-sunum.md` dosyasını inceleyin

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Nginx Dokümantasyonu | https://nginx.org/en/docs/ |
| Mozilla SSL Yapılandırıcı | https://ssl-config.mozilla.org/ |
| Certbot | https://certbot.eff.org/ |
| DigitalOcean Nginx Rehberleri | https://www.digitalocean.com/community/tags/nginx |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
