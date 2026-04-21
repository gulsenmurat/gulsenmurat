---
title: "Linux Temelleri"
author: "Gülşen Murat"
date: "2024"
---

# 🐧 Linux Temelleri

**Sıfırdan Sistem Yönetimine**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Linux Nedir?

### Linux Nedir?

- **Açık kaynaklı**, Unix benzeri bir işletim sistemi çekirdeği
- 1991'de **Linus Torvalds** tarafından geliştirildi
- Bugün dünyanın en çok kullanılan sunucu OS'u
- Android, süperbilgisayarlar, bulut altyapıları hepsi Linux üzerinde

### Neden Linux Öğrenmeliyiz?

| Alan | Linux Kullanımı |
|------|-----------------|
| Web Sunucuları | %96+ |
| Bulut (AWS, GCP, Azure) | %90+ |
| Süperbilgisayarlar | %100 |
| Android Cihazlar | %100 |

---

## Slayt 2 — Dağıtım (Distro) Seçimi

### Popüler Linux Dağıtımları

```
         Linux Kernel
              │
    ┌─────────┼─────────┐
    │         │         │
  Debian    Red Hat    Arch
    │         │         │
  Ubuntu    CentOS    Manjaro
  Mint      AlmaLinux
            Fedora
```

### Yeni Başlayanlar İçin Öneriler

- 🟠 **Ubuntu** – En geniş topluluk desteği, kolay kurulum
- 🟢 **Linux Mint** – Windows'a en yakın deneyim
- 🔵 **Fedora** – Güncel yazılımlar, Red Hat tabanı

---

## Slayt 3 — Terminal ve Kabuk (Shell)

### Terminal Nedir?

Terminale komut yazarak işletim sistemiyle konuşuruz.

```bash
kullanici@sunucu:~$
│         │       │
│         │       └── Geçerli dizin (~ = ev dizini)
│         └────────── Sunucu adı
└──────────────────── Kullanıcı adı
```

### Temel Kabuklar

| Kabuk | Açıklama |
|-------|----------|
| **bash** | En yaygın kullanılan, varsayılan |
| **zsh** | bash uyumlu, ekstra özellikler |
| **sh** | POSIX standart kabuk |
| **fish** | Kullanıcı dostu, renkli |

```bash
# Hangi kabuk kullandığını öğren
echo $SHELL
```

---

## Slayt 4 — Dosya Sistemi Hiyerarşisi

### Linux Dizin Yapısı

```
/                    ← Kök dizin (root)
├── bin/             ← Temel komutlar (ls, cp, mv...)
├── etc/             ← Yapılandırma dosyaları
├── home/            ← Kullanıcı ev dizinleri
│   └── gülşen/      ← Senin ev dizinin (~)
├── var/             ← Değişken veri (log, cache)
│   └── log/         ← Log dosyaları
├── usr/             ← Kullanıcı programları
│   ├── bin/         ← Kurulu program dosyaları
│   └── local/       ← Yerel kurulumlar
├── tmp/             ← Geçici dosyalar (yeniden başlatmada silinir)
├── root/            ← Root kullanıcısının ev dizini
├── dev/             ← Aygıt dosyaları
├── proc/            ← Çalışan süreç bilgileri
└── sys/             ← Sistem bilgileri
```

---

## Slayt 5 — Temel Dosya Komutları

### Navigasyon

```bash
pwd               # Geçerli dizini göster
ls                # Dosyaları listele
ls -la            # Gizli dosyalar dahil detaylı liste
cd /etc           # Dizin değiştir
cd ~              # Ev dizinine git
cd ..             # Bir üst dizine git
cd -              # Önceki dizine geri dön
```

### Oluşturma / Kopyalama / Taşıma

```bash
touch dosya.txt           # Boş dosya oluştur
mkdir yeni-klasor         # Dizin oluştur
mkdir -p a/b/c            # İç içe dizin oluştur

cp dosya.txt yedek.txt    # Dosya kopyala
cp -r klasor/ yedek/      # Dizin kopyala

mv dosya.txt /tmp/        # Dosya taşı
mv eski.txt yeni.txt      # Dosyayı yeniden adlandır

rm dosya.txt              # Dosya sil
rm -rf klasor/            # Dizin sil (dikkatli!)
```

---

## Slayt 6 — Dosya İçeriği Görüntüleme

```bash
# Tüm dosyayı göster
cat /etc/hostname

# Sayfa sayfa göster
less /var/log/syslog
# (q ile çıkış, boşluk ile ileri sayfa)

# İlk 10 satır
head -10 /var/log/syslog

# Son 10 satır
tail -10 /var/log/syslog

# Canlı izle (log takibi için)
tail -f /var/log/nginx/access.log

# İçinde ara
grep "hata" /var/log/syslog
grep -r "127.0.0.1" /etc/nginx/    # Dizinde özyinelemeli ara
```

---

## Slayt 7 — Dosya İzinleri

### İzin Yapısı

```
-rwxr-xr--  1  gulsenmurat  users  1234  Jan 1  dosya.sh
│├──┤├──┤├──┤
││  │  │  └── Diğerleri (others): r-- = 4
││  │  └───── Grup (group):       r-x = 5
││  └──────── Sahip (owner):      rwx = 7
│└─────────── Dosya türü (- = dosya, d = dizin, l = link)
```

### İzin Değerleri

| Sembol | Değer | Anlamı |
|--------|-------|--------|
| r | 4 | Okuma |
| w | 2 | Yazma |
| x | 1 | Çalıştırma |
| - | 0 | İzin yok |

```bash
chmod 755 script.sh     # rwxr-xr-x
chmod 644 dosya.txt     # rw-r--r--
chmod +x script.sh      # Çalıştırma izni ekle

chown gulsenmurat:users dosya.txt    # Sahip değiştir
chown -R www-data /var/www/          # Özyinelemeli
```

---

## Slayt 8 — Kullanıcı ve Grup Yönetimi

```bash
# Kullanıcı işlemleri
whoami                          # Geçerli kullanıcı
id                              # Kullanıcı ve grup bilgisi
cat /etc/passwd                 # Tüm kullanıcılar

sudo useradd -m -s /bin/bash ali    # Yeni kullanıcı oluştur
sudo passwd ali                     # Şifre belirle
sudo userdel -r ali                 # Kullanıcı sil

# Grup işlemleri
groups                          # Üye olduğun gruplar
sudo groupadd gelistiriciler     # Grup oluştur
sudo usermod -aG sudo ali        # Sudo grubuna ekle
sudo usermod -aG docker gulsenmurat  # Docker grubuna ekle

# Kullanıcı değiştirme
su - ali                        # ali kullanıcısına geç
sudo -i                         # Root shell aç
sudo komut                      # Tek komut root olarak çalıştır
```

---

## Slayt 9 — Paket Yönetimi

### Ubuntu / Debian (APT)

```bash
sudo apt update                     # Paket listesini güncelle
sudo apt upgrade -y                 # Tüm paketleri güncelle
sudo apt install nginx git curl     # Paket kur
sudo apt remove nginx               # Paket kaldır
sudo apt purge nginx                # Paket + ayarları kaldır
sudo apt autoremove                 # Gereksiz bağımlılıkları temizle
apt search "text editor"            # Paket ara
apt show nginx                      # Paket detayı
dpkg -l                             # Kurulu paketleri listele
```

### CentOS / RHEL / AlmaLinux (DNF)

```bash
sudo dnf check-update               # Güncellemeleri kontrol et
sudo dnf update -y                  # Tüm paketleri güncelle
sudo dnf install nginx              # Paket kur
sudo dnf remove nginx               # Paket kaldır
dnf search "web server"             # Paket ara
dnf info nginx                      # Paket detayı
rpm -qa                             # Kurulu paketleri listele
```

---

## Slayt 10 — Süreç (Process) Yönetimi

```bash
# Süreçleri listele
ps aux                          # Tüm süreçleri göster
ps aux | grep nginx             # Nginx süreçlerini filtrele
top                             # Canlı kaynak kullanımı
htop                            # Gelişmiş canlı görünüm (önce kur: apt install htop)

# Süreç öldürme
kill 1234                       # PID ile sinyal gönder (varsayılan: SIGTERM)
kill -9 1234                    # Zorla öldür (SIGKILL)
pkill nginx                     # İsme göre öldür
killall nginx                   # Aynı isimli tümünü öldür

# Arka planda çalıştırma
komut &                         # Arka planda başlat
jobs                            # Arka plan işlerini listele
fg %1                           # 1. işi ön plana getir
bg %1                           # 1. işi arka planda devam ettir
nohup komut &                   # Terminal kapansa da devam et
```

---

## Slayt 11 — Systemd Servis Yönetimi

Modern Linux dağıtımlarında servisler **systemd** ile yönetilir.

```bash
# Servis durumu
sudo systemctl status nginx         # Durum kontrol
sudo systemctl is-active nginx      # Çalışıyor mu? (active/inactive)
sudo systemctl is-enabled nginx     # Açılışta başlıyor mu?

# Servis kontrolü
sudo systemctl start nginx          # Başlat
sudo systemctl stop nginx           # Durdur
sudo systemctl restart nginx        # Yeniden başlat
sudo systemctl reload nginx         # Yapılandırmayı yeniden yükle

# Başlangıç ayarları
sudo systemctl enable nginx         # Açılışta başlamasını sağla
sudo systemctl disable nginx        # Açılışta başlamasını engelle

# Tüm servisleri listele
systemctl list-units --type=service
```

---

## Slayt 12 — Ağ Komutları

```bash
# IP ve arayüz bilgisi
ip addr show                    # Tüm ağ arayüzleri
ip addr show eth0               # Belirli arayüz
hostname -I                     # IP adreslerini göster

# Bağlantı testi
ping google.com                 # ICMP ping
ping -c 4 google.com            # 4 kez ping at
traceroute google.com           # Ağ yolunu izle
curl -I https://google.com      # HTTP başlıklarını getir
wget https://orneksite.com/dosya.zip   # Dosya indir

# Açık portlar ve bağlantılar
ss -tlnp                        # Dinleyen TCP portları
ss -tulnp                       # TCP + UDP portları
netstat -tlnp                   # Eski alternatif
lsof -i :80                     # 80 portunu kullanan süreç

# DNS sorgusu
nslookup google.com
dig google.com
dig google.com MX               # Mail kayıtları
```

---

## Slayt 13 — Metin İşleme Araçları

```bash
# grep — Arama
grep "hata" /var/log/syslog                # Satır ara
grep -i "ERROR" /var/log/nginx/error.log   # Büyük/küçük harf duyarsız
grep -v "200" /var/log/nginx/access.log    # 200 İÇERMEYEN satırlar
grep -c "404" /var/log/nginx/access.log    # Eşleşen satır sayısı

# awk — Sütun işleme
awk '{print $1}' access.log               # 1. sütunu al (IP)
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# sed — Metin dönüştürme
sed 's/eski/yeni/g' dosya.txt             # Değiştir (dosyayı değiştirme)
sed -i 's/eski/yeni/g' dosya.txt          # Dosyayı değiştir
sed -n '5,10p' dosya.txt                  # 5-10. satırları göster

# sort, uniq, wc
sort dosya.txt                            # Sırala
sort -n sayilar.txt                       # Sayısal sırala
uniq -c tekrarlar.txt                     # Tekrar sayısıyla
wc -l dosya.txt                           # Satır sayısı
wc -w dosya.txt                           # Kelime sayısı
```

---

## Slayt 14 — Yönlendirme ve Borular (Pipe)

```bash
# Çıktı yönlendirme
komut > dosya.txt          # Üzerine yaz
komut >> dosya.txt         # Sonuna ekle
komut 2> hata.txt          # Hata çıktısını kaydet
komut &> tumu.txt          # Her şeyi kaydet
komut > /dev/null 2>&1     # Tüm çıktıyı sil

# Pipe — Komutları zincirle
ls -la | grep ".conf"      # ls çıktısını grep ile filtrele
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr

# Pratik örnekler
# En büyük 10 dosya
du -sh /var/log/* | sort -rh | head -10

# 80 portunu kullanan process
ss -tlnp | grep :80

# Aktif nginx bağlantıları say
ss -tnp | grep nginx | wc -l

# En çok 404 veren URL'ler
grep "404" /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c | sort -nr | head
```

---

## Slayt 15 — Disk ve Depolama

```bash
# Disk kullanımı
df -h                       # Tüm dosya sistemleri (insan okunabilir)
df -h /var                  # Belirli dizin
du -sh /var/log/            # Dizin toplam boyutu
du -sh /var/log/*           # Alt dizinleri karşılaştır
du -ah /home/ | sort -rh | head -20  # En büyük 20 dosya

# Disk bölümleri
lsblk                       # Blok aygıtları listele
fdisk -l                    # Disk bölümlerini göster
mount                       # Bağlı dosya sistemleri

# Dosya arama
find /var/log -name "*.log" -mtime +30    # 30 günden eski loglar
find /home -name "*.sh" -perm /111        # Çalıştırılabilir .sh dosyaları
find /tmp -size +100M                     # 100MB'tan büyük dosyalar
find /etc -name "nginx.conf"              # Dosya bul
```

---

## Slayt 16 — Ortam Değişkenleri ve Bashrc

```bash
# Ortam değişkenleri
echo $HOME          # Ev dizini
echo $PATH          # Çalıştırılabilir dizinler
echo $USER          # Kullanıcı adı
echo $SHELL         # Aktif kabuk
env                 # Tüm ortam değişkenleri
printenv HOME       # Belirli değişken

# Değişken tanımlama
export PROJE_DIZINI="/home/gulsenmurat/projeler"
echo $PROJE_DIZINI

# ~/.bashrc — Kalıcı ayarlar
nano ~/.bashrc

# ~/.bashrc içine eklenecek örnekler:
export PATH="$PATH:/usr/local/myapp/bin"
alias ll="ls -la --color=auto"
alias guncelle="sudo apt update && sudo apt upgrade -y"
alias nginx-log="sudo tail -f /var/log/nginx/access.log"

# Değişiklikleri uygula
source ~/.bashrc
# veya
. ~/.bashrc
```

---

## Slayt 17 — SSH ile Uzak Sunucu Yönetimi

```bash
# SSH bağlantısı
ssh kullanici@sunucu-ip          # Şifre ile bağlan
ssh -p 2222 kullanici@sunucu-ip  # Özel port
ssh -i ~/.ssh/anahtar.pem ec2-user@aws-sunucu-ip  # Anahtar dosyası ile

# SSH anahtar çifti oluşturma (daha güvenli)
ssh-keygen -t ed25519 -C "gulsenmurat@email.com"
# → ~/.ssh/id_ed25519 (özel anahtar)
# → ~/.ssh/id_ed25519.pub (genel anahtar)

# Genel anahtarı sunucuya kopyala
ssh-copy-id kullanici@sunucu-ip
# veya manuel:
cat ~/.ssh/id_ed25519.pub | ssh kullanici@sunucu-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Dosya transfer
scp dosya.txt kullanici@sunucu:/home/kullanici/     # Gönder
scp kullanici@sunucu:/var/log/nginx/access.log ./   # Al
scp -r klasor/ kullanici@sunucu:/tmp/               # Dizin gönder

# rsync (daha verimli)
rsync -avz ./proje/ kullanici@sunucu:/var/www/proje/
```

---

## Slayt 18 — Cron ile Zamanlama

```bash
# Crontab düzenle
crontab -e      # Geçerli kullanıcı
sudo crontab -e # Root

# Crontab sözdizimi:
# ┌───── dakika    (0-59)
# │ ┌──── saat     (0-23)
# │ │ ┌─── gün     (1-31)
# │ │ │ ┌── ay      (1-12)
# │ │ │ │ ┌─ haftanın günü (0-7, 0=Pazar)
# │ │ │ │ │
# * * * * *  komut

# Örnekler:
0 * * * *    /usr/bin/python3 /scripts/saat-basi.py      # Her saat başı
0 2 * * *    /usr/bin/certbot renew --quiet              # Her gece 02:00
0 9 * * 1    /scripts/haftalik-rapor.sh                  # Her Pazartesi 09:00
*/5 * * * *  /scripts/kontrol.sh                         # Her 5 dakikada bir
0 0 1 * *    /scripts/aylik-temizlik.sh                  # Her ayın 1'i

# Cron işlerini listele
crontab -l

# Cron çıktısını kaydet
0 2 * * * /scripts/yedek.sh >> /var/log/yedek.log 2>&1
```

---

## Slayt 19 — Log Yönetimi

```bash
# Sistem logları
sudo journalctl                          # Tüm systemd logları
sudo journalctl -u nginx                 # Nginx logları
sudo journalctl -u nginx -n 50          # Son 50 satır
sudo journalctl -u nginx -f             # Canlı izle
sudo journalctl --since "2024-01-01"    # Tarihten itibaren
sudo journalctl --since "1 hour ago"    # Son 1 saat

# Geleneksel log dosyaları
/var/log/syslog          # Genel sistem logları (Debian/Ubuntu)
/var/log/messages        # Genel sistem logları (RHEL/CentOS)
/var/log/auth.log        # SSH giriş denemeleri, sudo kullanımı
/var/log/kern.log        # Kernel logları
/var/log/dmesg           # Donanım mesajları

# Başarısız SSH girişleri
sudo grep "Failed password" /var/log/auth.log | tail -20

# Son sisteme girişler
last
lastb                    # Başarısız girişler
```

---

## Slayt 20 — Özet: Temel Linux Komutları Referans Kartı

```
NAVIGASYON          │  DOSYA İŞLEMLERİ      │  SİSTEM
pwd, ls, cd         │  touch, mkdir, cp      │  top, htop, ps
ls -la, ls -lh      │  mv, rm, rm -rf        │  kill, systemctl
                    │  cat, less, head, tail  │  df -h, du -sh
                    │  chmod, chown           │  free -h, uptime

AĞ                  │  ARAMA                  │  KULLANICI
ip addr, ping       │  grep, find             │  whoami, id
ss -tlnp, curl      │  locate, which          │  useradd, passwd
ssh, scp, rsync     │  awk, sed               │  sudo, su

PAKETLEmanager      │  METİN DÜZENLEME        │  LOG
apt update/install  │  nano, vim              │  journalctl -u
dnf update/install  │  cat > dosya            │  tail -f
                    │  echo "..." >> dosya    │  grep "hata"
```

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Linux Journey (etkileşimli öğrenme) | https://linuxjourney.com |
| The Linux Command Line (kitap) | https://linuxcommand.org/tlcl.php |
| OverTheWire: Bandit (oyunlaştırılmış) | https://overthewire.org/wargames/bandit |
| Ubuntu Dokümantasyonu | https://ubuntu.com/tutorials |
| man sayfaları | `man komut` (terminalde) |

> 💡 **İpucu:** `man komut` veya `komut --help` ile herhangi bir komutun yardım sayfasını okuyabilirsiniz.

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
