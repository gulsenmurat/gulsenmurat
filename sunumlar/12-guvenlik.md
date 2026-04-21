---
title: "Linux Sunucu Güvenliği"
author: "Gülşen Murat"
date: "2024"
---

# 🔐 Linux Sunucu Güvenliği

**Temel Hardening, Güvenlik Duvarı ve İzleme**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Neden Sunucu Güvenliği?

### İstatistikler

```
İnternete açık bir sunucu:
• İlk SSH saldırısı: sunucu ayağa kalktıktan ~5 dakika sonra
• Günlük brute-force deneme: 1.000 – 50.000+
• Root şifresi ile açık port 22: birkaç saatte ele geçirilir
```

### Güvenlik Katmanları (Defense in Depth)

```
Dış Saldırı
     │
     ▼
[ Güvenlik Duvarı   ] ← UFW / firewalld: gereksiz portları kapat
     │
     ▼
[ SSH Hardening     ] ← Şifre giriş yok, anahtar tabanlı giriş
     │
     ▼
[ Fail2ban          ] ← Brute-force'u otomatik engelle
     │
     ▼
[ OS Güncellemeleri ] ← CVE yamalarını hemen uygula
     │
     ▼
[ Uygulama          ] ← En az yetki prensibi, root ile çalıştırma
     │
     ▼
[ İzleme / Log      ] ← Olayları tespit et, uyar
```

---

## Slayt 2 — İlk Kurulum Sonrası Yapılacaklar

```bash
# 1. Sistemi güncelle
sudo apt update && sudo apt upgrade -y
sudo apt install -y unattended-upgrades    # Otomatik güvenlik güncellemeleri
sudo dpkg-reconfigure --priority=low unattended-upgrades

# 2. Yeni sudo kullanıcısı oluştur (root ile oturma!)
sudo adduser devops
sudo usermod -aG sudo devops

# 3. Root SSH girişini kapat (yeni kullanıcı test edildikten sonra!)
# /etc/ssh/sshd_config içinde:
# PermitRootLogin no

# 4. Güvenlik araçlarını kur
sudo apt install -y \
    ufw fail2ban \
    auditd aide \
    rkhunter lynis \
    unattended-upgrades
```

---

## Slayt 3 — SSH Güvenliği

### /etc/ssh/sshd_config

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak  # Önce yedek al
sudo nano /etc/ssh/sshd_config
```

```ini
# ── Port ve Protokol ───────────────────────────────────────
Port 2222                      # Varsayılan 22'yi değiştir (güvenlik yoluyla gizleme)
Protocol 2                     # SSH v2 zorunlu

# ── Kimlik Doğrulama ───────────────────────────────────────
PermitRootLogin no             # Root ile giriş kapat
PasswordAuthentication no      # Şifre ile giriş kapat (SSH key zorunlu!)
PubkeyAuthentication yes       # Anahtar tabanlı giriş
AuthorizedKeysFile .ssh/authorized_keys

# ── Güvenlik ───────────────────────────────────────────────
MaxAuthTries 3                 # Max deneme sayısı
MaxSessions 5                  # Max eş zamanlı oturum
LoginGraceTime 30              # Giriş için max süre (saniye)
ClientAliveInterval 300        # 5 dakika sessizlikte bağlantıyı kes
ClientAliveCountMax 2

# ── Kısıtlamalar ────────────────────────────────────────────
AllowUsers devops gulsenmurat  # Sadece bu kullanıcılar SSH yapabilir
X11Forwarding no               # GUI yönlendirme kapat
AllowTcpForwarding no          # TCP tünelleme kapat
```

```bash
# Sözdizimi kontrol et ve yeniden başlat
sudo sshd -t
sudo systemctl restart sshd

# SSH key oluştur (istemci tarafında)
ssh-keygen -t ed25519 -a 100 -C "devops@sunucu"
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 devops@sunucu-ip
```

---

## Slayt 4 — UFW Güvenlik Duvarı (Ubuntu)

```bash
# UFW kur ve yapılandır
sudo apt install ufw -y

# Varsayılan politikalar: her şeyi reddet, ihtiyacı aç
sudo ufw default deny incoming
sudo ufw default allow outgoing

# İzin ver
sudo ufw allow 2222/tcp comment "SSH (özel port)"
sudo ufw allow 80/tcp  comment "HTTP"
sudo ufw allow 443/tcp comment "HTTPS"

# Belirli IP'ye izin ver
sudo ufw allow from 10.0.0.0/8 to any port 5432 comment "PostgreSQL - iç ağ"
sudo ufw allow from 192.168.1.100 to any port 2222 comment "Yönetim IP"

# UFW etkinleştir
sudo ufw enable
sudo ufw status verbose

# Kural sil
sudo ufw delete allow 80/tcp
sudo ufw delete 3    # Kural numarasıyla (ufw status numbered)

# UFW log
sudo ufw logging on
sudo tail -f /var/log/ufw.log
```

---

## Slayt 5 — firewalld (CentOS/AlmaLinux)

```bash
sudo systemctl enable --now firewalld

# Aktif zone
firewall-cmd --get-active-zones
firewall-cmd --list-all

# Servis ekle
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Özel port ekle
sudo firewall-cmd --permanent --add-port=2222/tcp

# Belirli IP'ye izin ver
sudo firewall-cmd --permanent --zone=trusted \
    --add-source=10.0.0.0/8

# Kural uygula
sudo firewall-cmd --reload

# Servisi kaldır
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload
```

---

## Slayt 6 — Fail2ban (Brute-Force Koruması)

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

```ini
# /etc/fail2ban/jail.local (jail.conf'u kopyalama, override et)
[DEFAULT]
bantime  = 1h           # Engelleme süresi
findtime = 10m          # Bu süre içinde
maxretry = 3            # Bu kadar başarısız deneme olursa engelle
ignoreip = 127.0.0.1/8 192.168.0.0/16  # Bu IP'leri engelleme

[sshd]
enabled  = true
port     = 2222         # SSH portunu değiştirdiysen
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 24h

[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 5

[nginx-limit-req]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 10
findtime = 1m
bantime  = 10m
```

```bash
# Fail2ban kontrol
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client get sshd banip     # Engellenen IP'ler

# IP'yi engelden çıkar
sudo fail2ban-client set sshd unbanip 1.2.3.4

# Log izle
sudo tail -f /var/log/fail2ban.log
```

---

## Slayt 7 — Otomatik Güvenlik Güncellemeleri

```bash
sudo apt install unattended-upgrades apt-listchanges -y
```

```ini
# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Package-Blacklist {
    // Güncellenmemesini istediğin paketler
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";   # Yeniden başlatmayı elle yap

Unattended-Upgrade::Mail "devops@orneksite.com";
Unattended-Upgrade::MailReport "on-change";
```

```ini
# /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

```bash
# Test et
sudo unattended-upgrades --dry-run --debug
```

---

## Slayt 8 — Kullanıcı ve Yetki Güvenliği

```bash
# Gereksiz kullanıcı hesaplarını kilitle
sudo usermod -L kullanici        # Hesabı kilitle
sudo passwd -l kullanici         # Şifreyle girişi engelle
sudo chsh -s /sbin/nologin servis_kullanici  # Shell atamasını engelle

# Sudo loglarını aktifleştir
echo "Defaults logfile=/var/log/sudo.log" | sudo tee -a /etc/sudoers.d/sudolog

# Sudo'da şifre tekrarını kısalt
echo "Defaults timestamp_timeout=5" | sudo tee -a /etc/sudoers.d/timeout

# Tek komuta sudo izni ver (komut kısıtlı sudo)
echo "devops ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx" \
    | sudo tee /etc/sudoers.d/devops-nginx

# Son giriş bilgisi
last
lastb           # Başarısız girişler
lastlog         # Tüm kullanıcıların son giriş

# Boş şifreli hesaplar
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
```

---

## Slayt 9 — Audit (Denetim) Logları

```bash
sudo apt install auditd -y
sudo systemctl enable --now auditd
```

```bash
# /etc/audit/rules.d/audit.rules
# Önemli dosyalara erişimi izle
-w /etc/passwd -p wa -k kimlik_degisiklikleri
-w /etc/shadow -p wa -k kimlik_degisiklikleri
-w /etc/sudoers -p wa -k sudo_degisiklik
-w /etc/ssh/sshd_config -p wa -k ssh_degisiklik

# Önemli komutları izle
-a always,exit -F path=/usr/bin/passwd -F perm=x -k sifre_degistir
-a always,exit -F path=/usr/sbin/useradd -F perm=x -k yeni_kullanici
-a always,exit -F path=/usr/bin/sudo -F perm=x -k sudo_kullanimi

# Ağ bağlantılarını izle
-a always,exit -F arch=b64 -S connect -k ag_baglantisi
```

```bash
# Kuralları uygula
sudo augenrules --load

# Audit log sorgula
sudo ausearch -k sudo_kullanimi
sudo ausearch -k kimlik_degisiklikleri --start today
sudo aureport --summary          # Özet rapor
sudo aureport --login --summary  # Giriş özeti
```

---

## Slayt 10 — Güvenlik Taraması

### Lynis (Sistem Güvenlik Denetimi)

```bash
sudo apt install lynis -y

# Tam sistem taraması
sudo lynis audit system

# Çıktı:
# [+] Hardening index: 68 [##########          ] 68/100
# Suggestion: Consider disabling root login via SSH [...]
```

### rkhunter (Rootkit Taraması)

```bash
sudo apt install rkhunter -y

# Veritabanını güncelle
sudo rkhunter --update

# Sistem taraması
sudo rkhunter --check --skip-keypress

# Günlük otomatik tarama (cron)
echo "0 3 * * * root /usr/bin/rkhunter --check --skip-keypress --report-warnings-only \
    | mail -s '[rkhunter] Sunucu Tarama' devops@orneksite.com" \
    | sudo tee /etc/cron.d/rkhunter
```

### AIDE (Dosya Bütünlüğü)

```bash
sudo apt install aide -y
sudo aideinit            # Başlangıç veritabanı oluştur
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Değişiklikleri kontrol et
sudo aide --check        # Farklılıkları göster
```

---

## Slayt 11 — Log İzleme ve Uyarı

```bash
# Başarısız SSH girişleri
sudo grep "Failed password" /var/log/auth.log | \
    awk '{print $11}' | sort | uniq -c | sort -nr | head

# Başarılı sudo kullanımı
sudo grep "COMMAND" /var/log/sudo.log | tail -20

# Nginx saldırı taraması
sudo grep -i "sqlmap\|nikto\|nmap\|masscan" /var/log/nginx/access.log

# Logwatch — Günlük e-posta özet
sudo apt install logwatch -y
sudo logwatch --output mail --mailto devops@orneksite.com --detail high
```

### Logrotate Yapılandırması

```ini
# /etc/logrotate.d/uygulama-logs
/var/log/uygulama/*.log {
    daily
    rotate 30               # 30 gün sakla
    compress                # gzip ile sıkıştır
    delaycompress           # Bir gün bekle, sonra sıkıştır
    missingok               # Log yoksa hata verme
    notifempty              # Boşsa döndürme
    sharedscripts
    postrotate
        nginx -s reopen     # Nginx'e yeni log dosyasını kullan
    endscript
}
```

---

## Slayt 12 — Güvenli Uygulama Çalıştırma

```bash
# Uygulamayı root ile ÇALIŞTIRSMA — ayrı kullanıcı oluştur
sudo useradd -r -s /sbin/nologin -d /var/lib/uygulama uygulama

# Systemd servisi — düşük yetki ile çalıştır
```

```ini
# /etc/systemd/system/uygulama.service
[Unit]
Description=Benim Uygulamam
After=network.target

[Service]
Type=simple
User=uygulama                   # Root değil!
Group=uygulama
WorkingDirectory=/var/www/uygulama
ExecStart=/usr/bin/node server.js
Restart=always

# Güvenlik kısıtlamaları
NoNewPrivileges=yes              # Yetki yükseltme kapat
ProtectSystem=strict             # Dosya sistemi salt okunur
ProtectHome=yes                  # /home dizinine erişim kapat
ReadWritePaths=/var/lib/uygulama  # Sadece bu dizine yaz
PrivateTmp=yes                   # Özel /tmp
CapabilityBoundingSet=           # Tüm Linux yeteneklerini kaldır

[Install]
WantedBy=multi-user.target
```

---

## Slayt 13 — Güvenlik Kontrol Listesi

```
SSH Güvenliği
  ✅ Root girişi kapalı (PermitRootLogin no)
  ✅ Şifre ile giriş kapalı (PasswordAuthentication no)
  ✅ SSH portu değiştirildi (22 → 2222)
  ✅ AllowUsers ile kısıtlandı

Güvenlik Duvarı
  ✅ UFW/firewalld etkin
  ✅ Sadece gerekli portlar açık (80, 443, 2222)
  ✅ Veritabanı portları dışa kapalı

Kullanıcı Yönetimi
  ✅ Ayrı sudo kullanıcısı oluşturuldu
  ✅ Gereksiz kullanıcılar kilitlendi
  ✅ Servisler ayrı kullanıcı ile çalışıyor

Güncellemeler
  ✅ Sistem güncel (apt upgrade)
  ✅ Otomatik güvenlik güncellemeleri etkin

İzleme
  ✅ Fail2ban kurulu ve yapılandırıldı
  ✅ Audit logları aktif
  ✅ Düzenli güvenlik taraması (lynis, rkhunter)
  ✅ Log rotasyonu yapılandırıldı
```

---

## Slayt 14 — Özet

### Öğrendiklerimiz ✅

- Defense in Depth (Derinlemesine Savunma) prensibi
- SSH hardening: anahtar tabanlı giriş, root engel, AllowUsers
- UFW (Ubuntu) ve firewalld (CentOS) yapılandırması
- Fail2ban ile brute-force koruması
- Otomatik güvenlik güncellemeleri
- Sudo kısıtlama ve denetim logları
- auditd ile sistem denetimi
- Lynis ve rkhunter güvenlik taraması
- AIDE ile dosya bütünlüğü
- Systemd güvenlik kısıtlamaları (NoNewPrivileges, ProtectSystem)
- Güvenlik kontrol listesi

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks |
| NIST Güvenlik Rehberleri | https://www.nist.gov/cyberframework |
| Lynis | https://cisofy.com/lynis |
| Ubuntu Security Guide | https://ubuntu.com/security/certifications/docs/usg |
| SSH Hardening Rehberi | https://www.ssh.com/academy/ssh/sshd-config |

> ⚠️ **Önemli Uyarı:** SSH yapılandırmasını değiştirirken mevcut oturumunu açık tut.
> Yeni bir terminal sekmesinde bağlantı testini başarılı gördükten sonra eski oturumu kapat.

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
