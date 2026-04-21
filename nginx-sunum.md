# 🚀 Nginx: Kurulum, Yapılandırma ve Çalışma Mantığı

> **Kapsamlı Teknik Sunum**  
> Hazırlayan: Gülşen Murat

---

## 📋 İçindekiler

1. [Nginx Nedir?](#1-nginx-nedir)
2. [Nginx Nasıl Çalışır?](#2-nginx-nasıl-çalışır)
3. [Apache ile Farkları](#3-apache-ile-farkları)
4. [Kurulum Öncesi Hazırlık](#4-kurulum-öncesi-hazırlık)
5. [Ubuntu / Debian Üzerine Kurulum](#5-ubuntu--debian-üzerine-kurulum)
6. [CentOS / RHEL / AlmaLinux Üzerine Kurulum](#6-centos--rhel--almalinux-üzerine-kurulum)
7. [Nginx Dizin Yapısı](#7-nginx-dizin-yapısı)
8. [Temel Yapılandırma Dosyası](#8-temel-yapılandırma-dosyası)
9. [Sanal Sunucu (Virtual Host) Kurulumu](#9-sanal-sunucu-virtual-host-kurulumu)
10. [SSL / TLS Kurulumu (Let's Encrypt)](#10-ssl--tls-kurulumu-lets-encrypt)
11. [Reverse Proxy Yapılandırması](#11-reverse-proxy-yapılandırması)
12. [Load Balancer Kullanımı](#12-load-balancer-kullanımı)
13. [Güvenlik Ayarları](#13-güvenlik-ayarları)
14. [Temel Nginx Komutları](#14-temel-nginx-komutları)
15. [Log Yönetimi](#15-log-yönetimi)
16. [Sık Karşılaşılan Hatalar](#16-sık-karşılaşılan-hatalar)
17. [Özet ve Kaynaklar](#17-özet-ve-kaynaklar)

---

## 1. Nginx Nedir?

**Nginx** (okunuş: "engine-x"), Igor Sysoev tarafından 2004 yılında geliştirilen, açık kaynak kodlu yüksek performanslı bir:

- 🌐 **Web sunucusu**
- 🔄 **Ters proxy (Reverse Proxy)**
- ⚖️ **Yük dengeleyici (Load Balancer)**
- 📦 **HTTP önbelleği (HTTP Cache)**
- 📧 **Mail proxy sunucusu**

olarak kullanılabilir.

### Neden Nginx?

| Özellik | Detay |
|--------|-------|
| Yüksek eş zamanlılık | 10.000+ eş zamanlı bağlantıyı düşük kaynak kullanımıyla yönetir |
| Düşük bellek tüketimi | Her bağlantı için ayrı process açmaz |
| Hızlı statik dosya servisi | Statik içerikleri çok hızlı iletir |
| Modüler yapı | Eklentilerle genişletilebilir |
| Aktif topluluk | Geniş dokümantasyon ve destek |

---

## 2. Nginx Nasıl Çalışır?

### Olay Tabanlı (Event-Driven) Mimari

Nginx, **asenkron, olay tabanlı** bir mimari kullanır. Bu sayede her istek için ayrı bir process (süreç) oluşturmaz; bunun yerine az sayıda **worker process** ile çok sayıda isteği aynı anda işler.

```
                        ┌────────────────────────────┐
                        │         Nginx               │
                        │                             │
İstemci 1 ─────────┐   │  ┌──────────────────────┐  │
İstemci 2 ─────────┼──►│  │   Master Process      │  │
İstemci 3 ─────────┤   │  │  (Yapılandırma yönet) │  │
      ...           │   │  └──────────┬───────────┘  │
İstemci N ─────────┘   │             │               │
                        │  ┌──────────▼───────────┐  │
                        │  │   Worker Process 1    │  │
                        │  │   Worker Process 2    │  │
                        │  │       ...             │  │
                        │  │   Worker Process N    │  │
                        │  └──────────────────────┘  │
                        └────────────────────────────┘
```

### Master – Worker Süreci

| Süreç | Görevi |
|-------|--------|
| **Master Process** | Yapılandırma dosyasını okur, worker processleri yönetir, sinyalleri dinler |
| **Worker Process** | Gerçek HTTP isteklerini işler, sayısı genellikle CPU çekirdeği sayısına eşittir |
| **Cache Manager** | Önbellek yönetimi yapar |
| **Cache Loader** | Disk üzerindeki önbelleği belleğe yükler |

### İstek İşleme Akışı

```
1. İstemci TCP bağlantısı kurar
2. Worker process bağlantıyı non-blocking I/O ile kabul eder
3. HTTP isteği parse edilir
4. İlgili location bloğu bulunur
5. İstek işlenir (statik dosya, proxy, vb.)
6. Yanıt istemciye gönderilir
7. Bağlantı (keep-alive ise) açık tutulur
```

---

## 3. Apache ile Farkları

| Özellik | Nginx | Apache |
|---------|-------|--------|
| Mimari | Olay tabanlı, asenkron | Process/Thread tabanlı |
| Eş zamanlı bağlantı | Çok yüksek | Orta |
| Bellek kullanımı | Düşük | Yüksek |
| Statik dosya hızı | Çok hızlı | Hızlı |
| Dinamik içerik | Proxy ile | Doğrudan modüller ile |
| .htaccess desteği | Yok | Var |
| Yapılandırma | Merkezi | Dizin bazlı da olabilir |

---

## 4. Kurulum Öncesi Hazırlık

### Sistem Gereksinimleri

- 64-bit Linux işletim sistemi (Ubuntu 20.04+, Debian 10+, CentOS 7+, AlmaLinux 8+)
- Minimum 512 MB RAM (üretim için 1 GB+ önerilir)
- Root veya sudo yetkisi
- Açık portlar: **80 (HTTP)**, **443 (HTTPS)**

### Güvenlik Duvarı Kontrolü

```bash
# Mevcut portları kontrol et
sudo ss -tlnp | grep -E '80|443'

# UFW ile port açma (Ubuntu/Debian)
sudo ufw allow 'Nginx Full'
sudo ufw status

# firewalld ile port açma (CentOS/RHEL)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Eski Web Sunucusunu Durdur

```bash
# Apache çalışıyorsa durdur
sudo systemctl stop apache2    # Ubuntu/Debian
sudo systemctl stop httpd      # CentOS/RHEL

# 80 portunu kullanan süreci bul
sudo lsof -i :80
```

---

## 5. Ubuntu / Debian Üzerine Kurulum

### 5.1 Paket Deposu Güncelleme ve Kurulum

```bash
# Sistem paketlerini güncelle
sudo apt update && sudo apt upgrade -y

# Nginx'i yükle
sudo apt install nginx -y
```

### 5.2 Kurulum Doğrulama

```bash
# Nginx sürümünü kontrol et
nginx -v
# Çıktı: nginx version: nginx/1.24.0

# Detaylı sürüm bilgisi (modüller dahil)
nginx -V
```

### 5.3 Servisi Başlatma ve Etkinleştirme

```bash
# Nginx servisini başlat
sudo systemctl start nginx

# Sistem açılışında otomatik başlamasını sağla
sudo systemctl enable nginx

# Servis durumunu kontrol et
sudo systemctl status nginx
```

Beklenen çıktı:
```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-01 10:00:00 UTC; 5s ago
```

### 5.4 Tarayıcıdan Test

Sunucu IP adresini tarayıcıya yaz:  
`http://<sunucu-ip-adresi>`

"**Welcome to nginx!**" sayfasını görürsen kurulum başarılıdır. ✅

### 5.5 Nginx Resmi Deposundan Kurulum (Güncel Sürüm İçin)

```bash
# Bağımlılıkları yükle
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring -y

# Nginx imzalama anahtarını indir
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

# Kararlı (stable) depoyu ekle
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu $(lsb_release -cs) nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

# Paket listesini güncelle ve kur
sudo apt update
sudo apt install nginx -y
```

---

## 6. CentOS / RHEL / AlmaLinux Üzerine Kurulum

### 6.1 EPEL ve Nginx Kurulumu

```bash
# EPEL deposunu ekle
sudo dnf install epel-release -y

# Nginx'i yükle
sudo dnf install nginx -y
```

### 6.2 Resmi Nginx Deposu ile Kurulum

```bash
# /etc/yum.repos.d/nginx.repo dosyasını oluştur
sudo tee /etc/yum.repos.d/nginx.repo <<EOF
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF

# Nginx'i yükle
sudo dnf install nginx -y
```

### 6.3 Servisi Başlatma

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### 6.4 SELinux Ayarları

```bash
# SELinux nginx'in ağ bağlantısına izin ver
sudo setsebool -P httpd_can_network_connect 1

# Proxy için
sudo setsebool -P httpd_can_network_connect_db 1
```

---

## 7. Nginx Dizin Yapısı

```
/etc/nginx/
├── nginx.conf              # Ana yapılandırma dosyası
├── conf.d/                 # Ek yapılandırma dosyaları (.conf uzantılı)
│   └── default.conf
├── sites-available/        # Tanımlanan sanal sunucular (Ubuntu/Debian)
│   └── default
├── sites-enabled/          # Aktif sanal sunucular (symlink)
│   └── default -> ../sites-available/default
├── modules-available/      # Kullanılabilir modüller
├── modules-enabled/        # Aktif modüller
├── snippets/               # Yeniden kullanılabilir yapılandırma parçaları
├── mime.types              # MIME türleri eşleştirme
└── fastcgi_params          # FastCGI parametreleri

/var/log/nginx/
├── access.log              # Erişim logları
└── error.log               # Hata logları

/var/www/html/              # Varsayılan web kök dizini (Ubuntu/Debian)
/usr/share/nginx/html/      # Varsayılan web kök dizini (CentOS/RHEL)
```

---

## 8. Temel Yapılandırma Dosyası

`/etc/nginx/nginx.conf` dosyasının detaylı açıklaması:

```nginx
# Worker process sayısı - genellikle CPU çekirdeği sayısına eşit ayarlanır
worker_processes auto;

# Her worker'ın açabileceği maksimum dosya tanımlayıcı sayısı
worker_rlimit_nofile 65535;

# Hata log seviyesi: debug | info | notice | warn | error | crit
error_log /var/log/nginx/error.log warn;

# Master process PID dosyası
pid /run/nginx.pid;

# ── events bloğu ────────────────────────────────────────────────
events {
    # Her worker'ın aynı anda işleyebileceği max bağlantı sayısı
    worker_connections 1024;

    # Tüm bağlantıları tek seferde kabul et (performans artışı)
    multi_accept on;

    # Linux'ta en verimli bağlantı işleme yöntemi
    use epoll;
}

# ── http bloğu ──────────────────────────────────────────────────
http {
    # MIME türlerini dahil et
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Log formatı tanımlama
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # Erişim logu
    access_log  /var/log/nginx/access.log  main;

    # Dosya gönderimini kernel seviyesinde yap (performans)
    sendfile        on;

    # TCP paketlerini birleştir (sendfile ile birlikte kullan)
    tcp_nopush      on;

    # TCP gecikmesini azalt
    tcp_nodelay     on;

    # Keep-alive bağlantı zaman aşımı (saniye)
    keepalive_timeout  65;

    # Yanıt başlığında Nginx sürümünü gizle
    server_tokens off;

    # Gzip sıkıştırma
    gzip  on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    # conf.d dizinindeki tüm .conf dosyalarını dahil et
    include /etc/nginx/conf.d/*.conf;

    # Ubuntu/Debian için
    include /etc/nginx/sites-enabled/*;
}
```

---

## 9. Sanal Sunucu (Virtual Host) Kurulumu

### 9.1 Yeni Site Yapılandırması Oluşturma

```bash
sudo nano /etc/nginx/sites-available/orneksite.com
```

```nginx
server {
    # HTTP trafiği için port dinle
    listen 80;
    listen [::]:80;  # IPv6

    # Alan adı tanımla
    server_name orneksite.com www.orneksite.com;

    # Web kök dizini
    root /var/www/orneksite.com/html;

    # Varsayılan dizin dosyası
    index index.html index.htm index.php;

    # Erişim ve hata logları
    access_log /var/log/nginx/orneksite.com.access.log;
    error_log  /var/log/nginx/orneksite.com.error.log warn;

    # URL eşleştirme bloğu
    location / {
        # Önce dosyayı, sonra dizini, sonra 404 döndür
        try_files $uri $uri/ =404;
    }

    # Statik dosyalar için önbellekleme
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # .htaccess dosyalarına erişimi engelle
    location ~ /\.ht {
        deny all;
    }

    # PHP desteği (php-fpm kurulu ise)
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
```

### 9.2 Siteyi Etkinleştirme

```bash
# Symlink oluştur (sites-enabled'a ekle)
sudo ln -s /etc/nginx/sites-available/orneksite.com /etc/nginx/sites-enabled/

# Web dizinini oluştur
sudo mkdir -p /var/www/orneksite.com/html

# Test sayfası oluştur
echo "<h1>Merhaba, Nginx çalışıyor!</h1>" | sudo tee /var/www/orneksite.com/html/index.html

# Nginx sahipliğini düzelt
sudo chown -R www-data:www-data /var/www/orneksite.com

# Yapılandırma sözdizimini test et
sudo nginx -t

# Nginx'i yeniden yükle (servisi durdurmadan)
sudo systemctl reload nginx
```

---

## 10. SSL / TLS Kurulumu (Let's Encrypt)

### 10.1 Certbot Kurulumu

```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx -y

# CentOS/RHEL
sudo dnf install certbot python3-certbot-nginx -y
```

### 10.2 Sertifika Alma

```bash
# Nginx eklentisi ile otomatik kurulum
sudo certbot --nginx -d orneksite.com -d www.orneksite.com

# Sadece sertifika al, yapılandırmaya dokunma
sudo certbot certonly --nginx -d orneksite.com
```

### 10.3 Certbot Çıktısı ve Otomatik Yenileme

```bash
# Yenileme testi
sudo certbot renew --dry-run

# Crontab ile otomatik yenileme (her gün iki kez kontrol)
# Certbot genellikle /etc/cron.d/certbot dosyasını otomatik oluşturur
cat /etc/cron.d/certbot
# 0 */12 * * * root certbot -q renew --no-self-upgrade
```

### 10.4 Manuel SSL Yapılandırması

```nginx
server {
    listen 80;
    server_name orneksite.com www.orneksite.com;

    # HTTP'yi HTTPS'e yönlendir
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name orneksite.com www.orneksite.com;

    # SSL sertifikaları
    ssl_certificate     /etc/letsencrypt/live/orneksite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orneksite.com/privkey.pem;

    # Güvenli SSL ayarları
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # HSTS başlığı (1 yıl)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/orneksite.com/chain.pem;

    # SSL oturum önbelleği
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    root /var/www/orneksite.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## 11. Reverse Proxy Yapılandırması

Nginx'i bir uygulama sunucusunun önüne (Node.js, Python, Java vb.) proxy olarak kullanma:

```nginx
server {
    listen 80;
    server_name api.orneksite.com;

    # İstemci IP'sini arka uç uygulamaya ilet
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    location / {
        # Arka uç uygulama adresi (örn. Node.js 3000 portunda)
        proxy_pass http://localhost:3000;

        # Zaman aşımı değerleri
        proxy_connect_timeout  60s;
        proxy_send_timeout     60s;
        proxy_read_timeout     60s;

        # WebSocket desteği
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Tampon boyutları
        proxy_buffer_size          128k;
        proxy_buffers              4 256k;
        proxy_busy_buffers_size    256k;
    }

    # Statik dosyaları doğrudan Nginx servis etsin
    location /static/ {
        alias /var/www/orneksite.com/static/;
        expires 30d;
    }
}
```

---

## 12. Load Balancer Kullanımı

```nginx
http {
    # Upstream (arka uç sunucu grubu) tanımla
    upstream uygulama_grubu {

        # Yük dengeleme yöntemleri:
        # (round-robin - varsayılan, yazılmasına gerek yok)
        # least_conn;   # En az bağlantılı sunucuya yönlendir
        # ip_hash;      # Aynı IP her zaman aynı sunucuya gitsin

        server 10.0.0.1:3000 weight=3;   # Ağırlıklı - 3 kat fazla trafik alır
        server 10.0.0.2:3000 weight=1;
        server 10.0.0.3:3000 backup;     # Diğerleri çökerse devreye girer

        # Bağlantı sağlığı ayarları
        keepalive 32;
    }

    server {
        listen 80;
        server_name orneksite.com;

        location / {
            proxy_pass http://uygulama_grubu;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

---

## 13. Güvenlik Ayarları

### 13.1 Temel Güvenlik Başlıkları

```nginx
server {
    # Nginx sürümünü gizle
    server_tokens off;

    # Tıklama hırsızlığını önle
    add_header X-Frame-Options "SAMEORIGIN" always;

    # MIME türü algılamayı kapat
    add_header X-Content-Type-Options "nosniff" always;

    # XSS koruması
    add_header X-XSS-Protection "1; mode=block" always;

    # Referrer politikası
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
}
```

### 13.2 İstek Hız Sınırlama (Rate Limiting)

```nginx
http {
    # Hız sınırlama bölgesi tanımla (IP başına 10 istek/saniye)
    limit_req_zone $binary_remote_addr zone=genel:10m rate=10r/s;

    server {
        location /api/ {
            # Aşımda 5 isteği kuyruğa al, fazlasını 429 ile reddet
            limit_req zone=genel burst=5 nodelay;
            limit_req_status 429;

            proxy_pass http://localhost:3000;
        }

        location /login {
            # Login için daha sıkı limit: 1 istek/saniye
            limit_req zone=genel burst=3;
        }
    }
}
```

### 13.3 Bağlantı Sınırlama

```nginx
http {
    # IP başına max bağlantı bölgesi
    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
        # IP başına max 10 eş zamanlı bağlantı
        limit_conn addr 10;
    }
}
```

### 13.4 Belirli IP'leri Engelleme

```nginx
location /yonetim {
    allow 192.168.1.0/24;   # Yerel ağa izin ver
    allow 203.0.113.5;      # Belirli IP'ye izin ver
    deny  all;              # Diğer herkesi engelle
}
```

---

## 14. Temel Nginx Komutları

```bash
# ── Servis Yönetimi ─────────────────────────────────────────────
sudo systemctl start nginx      # Nginx'i başlat
sudo systemctl stop nginx       # Nginx'i durdur
sudo systemctl restart nginx    # Yeniden başlat (bağlantı kesilir)
sudo systemctl reload nginx     # Yapılandırmayı yeniden yükle (bağlantı kesilmez)
sudo systemctl status nginx     # Durum kontrol et
sudo systemctl enable nginx     # Sistem açılışında başlamasını etkinleştir
sudo systemctl disable nginx    # Sistem açılışında başlamasını devre dışı bırak

# ── Nginx Doğrudan Komutları ────────────────────────────────────
nginx -t                        # Yapılandırma sözdizimini test et
nginx -T                        # Test et ve tüm yapılandırmayı göster
nginx -s reload                 # Yapılandırmayı yeniden yükle
nginx -s stop                   # Nginx'i hemen durdur (fast shutdown)
nginx -s quit                   # Nginx'i bekleyerek durdur (graceful shutdown)
nginx -s reopen                 # Log dosyalarını yeniden aç
nginx -v                        # Sürüm numarasını göster
nginx -V                        # Sürüm + derleme parametrelerini göster

# ── Süreç Kontrolü ──────────────────────────────────────────────
ps aux | grep nginx             # Nginx süreçlerini listele
sudo kill -HUP $(cat /run/nginx.pid)   # Master'a reload sinyali gönder
sudo kill -QUIT $(cat /run/nginx.pid)  # Graceful shutdown
```

---

## 15. Log Yönetimi

### 15.1 Log Dosyaları

```bash
# Erişim loglarını canlı izle
sudo tail -f /var/log/nginx/access.log

# Hata loglarını izle
sudo tail -f /var/log/nginx/error.log

# Son 100 satır
sudo tail -100 /var/log/nginx/access.log

# Belirli bir kelimeye göre filtrele
sudo grep "404" /var/log/nginx/access.log | tail -20

# IP başına istek sayısını say
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10

# En çok istenen URL'ler
sudo awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10
```

### 15.2 Log Rotasyon

Ubuntu/Debian'da `/etc/logrotate.d/nginx` dosyası otomatik oluşturulur:

```
/var/log/nginx/*.log {
    daily           # Günlük rotasyon
    missingok       # Log yoksa hata verme
    rotate 14       # 14 günlük log sakla
    compress        # Eski logları sıkıştır
    delaycompress   # Bir öncekini henüz sıkıştırma
    notifempty      # Boş log dosyasını rotasyona alma
    create 0640 www-data adm   # Yeni dosya izinleri
    sharedscripts
    postrotate
        # Nginx'e log dosyalarını yeniden açmasını söyle
        if [ -f /run/nginx.pid ]; then
            kill -USR1 $(cat /run/nginx.pid)
        fi
    endscript
}
```

---

## 16. Sık Karşılaşılan Hatalar

### ❌ 502 Bad Gateway

**Sebep:** Nginx arka uç uygulamaya (Node.js, PHP-FPM vb.) ulaşamıyor.

```bash
# PHP-FPM servisini kontrol et
sudo systemctl status php8.1-fpm

# Node.js uygulaması çalışıyor mu?
sudo ss -tlnp | grep 3000

# Nginx hata loguna bak
sudo tail -50 /var/log/nginx/error.log
```

---

### ❌ 403 Forbidden

**Sebep:** Dosya/dizin izinleri hatalı.

```bash
# Dosya izinlerini düzelt
sudo chown -R www-data:www-data /var/www/orneksite.com
sudo chmod -R 755 /var/www/orneksite.com
sudo chmod -R 644 /var/www/orneksite.com/html/*.html
```

---

### ❌ 413 Request Entity Too Large

**Sebep:** Yüklenen dosya boyutu `client_max_body_size` sınırını aşıyor.

```nginx
server {
    # Maksimum yükleme boyutunu artır (örn. 50 MB)
    client_max_body_size 50M;
}
```

---

### ❌ "nginx: [emerg] bind() to 0.0.0.0:80 failed"

**Sebep:** 80 portu başka bir süreç tarafından kullanılıyor.

```bash
# 80 portunu kullanan süreci bul ve durdur
sudo lsof -i :80
sudo systemctl stop apache2   # Örneğin Apache çalışıyorsa
```

---

### ❌ Yapılandırma Testi Başarısız

```bash
# Test çalıştır
sudo nginx -t

# Örnek hata mesajı:
# nginx: [emerg] unknown directive "servr_name" in /etc/nginx/sites-available/default:5
# → Yazım hatası var, düzelt ve tekrar test et
```

---

## 17. Özet ve Kaynaklar

### 🎯 Öğrendiklerimiz

✅ Nginx'in olay tabanlı mimarisini  
✅ Ubuntu/Debian ve CentOS/RHEL üzerine kurulumu  
✅ Temel yapılandırma dosyasının yapısını  
✅ Sanal sunucu (Virtual Host) oluşturmayı  
✅ SSL/TLS sertifika kurulumunu  
✅ Reverse proxy ve load balancer yapılandırmasını  
✅ Güvenlik ayarlarını  
✅ Log yönetimini  
✅ Sık karşılaşılan hataları ve çözümlerini  

### 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Nginx Dokümantasyonu | https://nginx.org/en/docs/ |
| Nginx Wiki | https://www.nginx.com/resources/wiki/ |
| Certbot (Let's Encrypt) | https://certbot.eff.org/ |
| Mozilla SSL Yapılandırma Üreticisi | https://ssl-config.mozilla.org/ |
| DigitalOcean Nginx Rehberleri | https://www.digitalocean.com/community/tags/nginx |

---

> **💡 İpucu:** Her yapılandırma değişikliğinden sonra `sudo nginx -t` ile sözdizimi kontrolü yapın, ardından `sudo systemctl reload nginx` ile servisi yeniden yükleyin. Bu sayede servis kesintisi olmadan değişiklikler uygulanır.

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
