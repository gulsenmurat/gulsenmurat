---
title: "Bash Scripting"
author: "Gülşen Murat"
date: "2024"
---

# 💻 Bash Scripting

**Kabuk Betik Programlama: Otomasyon ve Sistem Yönetimi**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Bash Script Nedir?

### Bash Script

Linux terminalinde çalıştırabileceğin komutları bir dosyaya yazarak **otomatikleştirme** aracı.

```bash
# Manuel yol (her gün elle yapılıyor):
cd /var/www/uygulama
git pull origin main
npm install --production
pm2 restart uygulama
echo "Deploy bitti"

# Script ile otomatik:
./deploy.sh    ← tek komut, her şeyi yapar
```

### Ne Zaman Bash Script Yazmalıyız?

- 🔄 Tekrarlayan sistem görevleri (yedekleme, temizlik)
- 🚀 Uygulama dağıtımı (deploy)
- 📊 Log analizi ve raporlama
- 📦 Kurulum betikleri
- 🕐 Cron ile zamanlı görevler

---

## Slayt 2 — İlk Script

```bash
#!/bin/bash
# ^^^^ Shebang: hangi yorumlayıcı kullanılacak

# Bu bir yorum satırıdır

echo "Merhaba, Dünya!"
echo "Tarih: $(date)"
echo "Kullanıcı: $USER"
echo "Dizin: $(pwd)"
```

### Çalıştırılabilir Yap ve Çalıştır

```bash
# Çalıştırma izni ver
chmod +x merhaba.sh

# Çalıştır
./merhaba.sh

# veya
bash merhaba.sh

# Sözdizimi kontrolü (çalıştırmadan)
bash -n merhaba.sh

# Adım adım çıktı (debug)
bash -x merhaba.sh
```

---

## Slayt 3 — Değişkenler

```bash
#!/bin/bash

# Değişken tanımla (= etrafında boşluk OLMAZ)
ISIM="Gülşen"
YAS=28
DIZIN="/var/www/uygulama"

# Kullan
echo "Merhaba, $ISIM!"
echo "Yaşım: ${YAS}"           # Süslü parantez: değişken adı belirsizse
echo "Dizin: $DIZIN"

# Komut çıktısını değişkene al
TARIH=$(date +%Y-%m-%d)
DOSYA_SAYISI=$(ls /etc | wc -l)
echo "Bugün: $TARIH"
echo "/etc dizininde $DOSYA_SAYISI dosya var"

# Salt okunur değişken
readonly VERSIYON="1.0.0"

# Değişkeni temizle
unset ISIM

# Ortam değişkeni (alt süreçlere geçer)
export APP_ENV="production"
```

---

## Slayt 4 — Özel Değişkenler

```bash
#!/bin/bash

# Komut satırı argümanları
echo "Script adı: $0"
echo "1. argüman: $1"
echo "2. argüman: $2"
echo "Tüm argümanlar: $@"
echo "Argüman sayısı: $#"

# Son komutun çıkış kodu
ls /etc/nginx
echo "Çıkış kodu: $?"       # 0 = başarılı, sıfırdan farklı = hata

# Geçerli süreç ID
echo "PID: $$"

# ./script.sh web01 80 ile çağrılırsa:
SUNUCU=$1
PORT=$2
echo "Sunucu: $SUNUCU, Port: $PORT"

# Argüman kontrolü
if [ $# -lt 2 ]; then
    echo "Kullanım: $0 <sunucu> <port>"
    exit 1
fi
```

---

## Slayt 5 — Kullanıcı Girişi

```bash
#!/bin/bash

# Basit giriş
echo -n "Adınız: "
read ISIM
echo "Merhaba, $ISIM!"

# Gizli giriş (şifre)
echo -n "Şifre: "
read -s SIFRE
echo ""        # Satır sonu (read -s satır sonu basmaz)

# Varsayılan değerli giriş
read -p "Ortam [staging]: " ORTAM
ORTAM=${ORTAM:-staging}     # Boşsa "staging" kullan
echo "Seçilen ortam: $ORTAM"

# Zaman aşımı ile giriş
read -t 10 -p "10 saniye içinde girin: " CEVAP
if [ $? -ne 0 ]; then
    echo "Zaman aşımı!"
fi

# Onay alma
read -p "Devam etmek istiyor musunuz? (e/H) " ONAY
if [[ $ONAY =~ ^[Ee]$ ]]; then
    echo "Devam ediliyor..."
else
    echo "İptal edildi."
    exit 0
fi
```

---

## Slayt 6 — Koşullar (if / case)

```bash
#!/bin/bash

SAYI=15

# if / elif / else
if [ $SAYI -gt 10 ]; then
    echo "10'dan büyük"
elif [ $SAYI -eq 10 ]; then
    echo "10'a eşit"
else
    echo "10'dan küçük"
fi

# Karşılaştırma operatörleri
# Sayısal: -eq -ne -lt -le -gt -ge
# String:  =  !=  <  >  -z (boş)  -n (boş değil)
# Dosya:   -f (dosya)  -d (dizin)  -e (var)  -r -w -x (izinler)

# Dosya kontrolleri
if [ -f /etc/nginx/nginx.conf ]; then
    echo "Nginx yapılandırması mevcut"
fi

if [ ! -d /var/www/uygulama ]; then
    mkdir -p /var/www/uygulama
    echo "Dizin oluşturuldu"
fi

# case (çoklu seçenek)
read -p "İşletim sistemi: " OS
case $OS in
    ubuntu|debian)
        apt install nginx -y ;;
    centos|almalinux)
        dnf install nginx -y ;;
    *)
        echo "Desteklenmeyen işletim sistemi"
        exit 1 ;;
esac
```

---

## Slayt 7 — Döngüler

```bash
#!/bin/bash

# for — liste üzerinde dön
for SUNUCU in web01 web02 web03; do
    echo "Kontrol: $SUNUCU"
    ping -c 1 $SUNUCU > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "  ✅ Erişilebilir"
    else
        echo "  ❌ Erişilemiyor"
    fi
done

# for — sayı aralığı
for i in {1..5}; do
    echo "Adım $i"
done

# for — C tarzı sözdizimi
for ((i=0; i<10; i++)); do
    echo "İndeks: $i"
done

# while döngüsü
SAYAC=1
while [ $SAYAC -le 5 ]; do
    echo "Sayaç: $SAYAC"
    ((SAYAC++))
done

# Dosya satırlarını oku
while IFS= read -r SATIR; do
    echo "İşleniyor: $SATIR"
done < sunucu_listesi.txt

# break ve continue
for i in {1..10}; do
    [ $i -eq 3 ] && continue    # 3'ü atla
    [ $i -eq 7 ] && break       # 7'de dur
    echo $i
done
```

---

## Slayt 8 — Fonksiyonlar

```bash
#!/bin/bash

# Fonksiyon tanımla
log() {
    local SEVIYE=$1           # local: sadece bu fonksiyonda geçerli
    local MESAJ=$2
    echo "[$(date +%H:%M:%S)] [$SEVIYE] $MESAJ"
}

kontrol_et() {
    local SERVIS=$1
    if systemctl is-active --quiet "$SERVIS"; then
        log "OK" "$SERVIS çalışıyor"
        return 0              # Başarı
    else
        log "HATA" "$SERVIS çalışmıyor!"
        return 1              # Başarısız
    fi
}

dizin_olustur() {
    local DIZIN=$1
    if [ ! -d "$DIZIN" ]; then
        mkdir -p "$DIZIN"
        log "BİLGİ" "Oluşturuldu: $DIZIN"
    fi
}

# Fonksiyonu çağır
log "BİLGİ" "Script başladı"
kontrol_et nginx
kontrol_et postgresql || log "UYARI" "PostgreSQL kapalı, yedek çalıştırılamaz"
dizin_olustur /var/backups/uygulama

# Fonksiyon çıkış kodunu yakala
if kontrol_et nginx; then
    echo "Nginx sağlıklı"
fi
```

---

## Slayt 9 — Hata Yönetimi

```bash
#!/bin/bash

# Hata durumunda script'i durdur
set -e            # Hata olunca dur (exit on error)
set -u            # Tanımsız değişken kullanımını hata say
set -o pipefail   # Pipe zincirinde hata olunca yakala
set -euo pipefail # Hepsini birden

# Trap: beklenmedik çıkışta temizlik yap
temizle() {
    local CIKIS_KODU=$?
    echo "Script sonlandı (kod: $CIKIS_KODU)"
    rm -f /tmp/kilit.dosyasi
    if [ $CIKIS_KODU -ne 0 ]; then
        echo "HATA OLUŞTU! Log dosyasını kontrol edin: /var/log/script.log"
    fi
}
trap temizle EXIT

# Belirli sinyaller için trap
trap 'echo "Ctrl+C yakalandı!"; exit 1' INT TERM

# Hata mesajı fonksiyonu
hata() {
    echo "HATA: $1" >&2    # stderr'e yaz
    exit 1
}

# Komut başarısızlık kontrolü
apt install nginx -y || hata "Nginx kurulumu başarısız"

# Manuel hata fırlat
[ -f /etc/nginx/nginx.conf ] || hata "nginx.conf bulunamadı"
```

---

## Slayt 10 — Metin İşleme

```bash
#!/bin/bash

# String uzunluğu
METIN="Merhaba Dünya"
echo ${#METIN}            # 14

# Alt string (dilimleme)
echo ${METIN:0:7}         # Merhaba
echo ${METIN:8}           # Dünya

# Değiştirme
DOSYA="nginx.conf.backup"
echo ${DOSYA/backup/yedek}            # nginx.conf.yedek
echo ${DOSYA//a/X}                    # XXT değiştirme (tüm a'lar)

# Uzantı kaldırma
DOSYA="rapor.2024-01-15.tar.gz"
echo ${DOSYA%.tar.gz}     # rapor.2024-01-15
echo ${DOSYA%.*}          # rapor.2024-01-15.tar
echo ${DOSYA##*.}         # gz

# Büyük/küçük harf
AD="gülşen"
echo ${AD^}               # Gülşen
echo ${AD^^}              # GÜLŞEN
echo ${AD,,}              # gülşen

# Boşsa varsayılan değer ver
DIZIN=${1:-/var/www/html}
echo "Dizin: $DIZIN"
```

---

## Slayt 11 — Dosya ve Dizin İşlemleri

```bash
#!/bin/bash

# Log dosyası
LOG="/var/log/script.log"

# Fonksiyon: log yaz
yaz() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"
}

# Geçici dosya (otomatik silinir)
GECICI=$(mktemp /tmp/isleme.XXXXXX)
trap "rm -f $GECICI" EXIT

# Dosya kilitleme (eş zamanlı çalışmayı önle)
KILIT="/var/run/benim-script.pid"
if [ -f "$KILIT" ]; then
    yaz "Script zaten çalışıyor (PID: $(cat $KILIT))"
    exit 1
fi
echo $$ > "$KILIT"
trap "rm -f $KILIT" EXIT

# Dizin işlemleri
yaz "Eski log dosyaları temizleniyor..."
find /var/log -name "*.log" -mtime +30 -delete
yaz "Temizlik tamamlandı"

# Dosya boyutu kontrol
BOYUT=$(du -sm /var/log | awk '{print $1}')
if [ $BOYUT -gt 1000 ]; then
    yaz "UYARI: /var/log $BOYUT MB kullanıyor"
fi
```

---

## Slayt 12 — Pratik Script: Yedekleme

```bash
#!/bin/bash
# yedekle.sh — Veritabanı ve dosya yedekleme scripti

set -euo pipefail

# ── Yapılandırma ─────────────────────────────────
DB_ADI="uygulama_db"
DB_KULLANICI="admin"
YEDEK_DIZINI="/var/backups/uygulama"
SAKLA_GUN=7                         # Kaç gün sakla
LOG="$YEDEK_DIZINI/yedek.log"

# ── Fonksiyonlar ─────────────────────────────────
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"; }
hata() { log "HATA: $1"; exit 1; }

# ── Ana script ───────────────────────────────────
mkdir -p "$YEDEK_DIZINI"
log "══════════════════════════════════"
log "Yedekleme başladı"

TARIH=$(date +%Y%m%d_%H%M%S)

# PostgreSQL yedeği
YEDEK_DOSYASI="$YEDEK_DIZINI/db_${DB_ADI}_${TARIH}.sql.gz"
log "Veritabanı yedekleniyor: $DB_ADI"
pg_dump -U "$DB_KULLANICI" "$DB_ADI" | gzip > "$YEDEK_DOSYASI" \
    || hata "Veritabanı yedeklemesi başarısız"
log "Veritabanı yedeği: $YEDEK_DOSYASI ($(du -sh $YEDEK_DOSYASI | cut -f1))"

# Dosya yedeği
log "Dosyalar yedekleniyor: /var/www/uygulama"
tar czf "$YEDEK_DIZINI/dosyalar_${TARIH}.tar.gz" \
    --exclude='/var/www/uygulama/node_modules' \
    /var/www/uygulama \
    || hata "Dosya yedeklemesi başarısız"

# Eski yedekleri temizle
log "$SAKLA_GUN günden eski yedekler siliniyor..."
find "$YEDEK_DIZINI" -name "*.gz" -mtime +$SAKLA_GUN -delete

log "Yedekleme başarıyla tamamlandı ✅"
```

---

## Slayt 13 — Pratik Script: Sistem Sağlık Kontrolü

```bash
#!/bin/bash
# saglik-kontrol.sh

ESIK_CPU=80      # % CPU kullanımı uyarı eşiği
ESIK_RAM=85      # % RAM kullanımı uyarı eşiği
ESIK_DISK=90     # % Disk kullanımı uyarı eşiği

UYARILAR=()

kontrol_et() {
    local BIRIM=$1 DEGER=$2 ESIK=$3
    if [ "$DEGER" -gt "$ESIK" ]; then
        UYARILAR+=("⚠️  $BIRIM: %$DEGER (eşik: %$ESIK)")
    else
        echo "✅ $BIRIM: %$DEGER"
    fi
}

# CPU kullanımı
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2+$4)}')
kontrol_et "CPU" $CPU $ESIK_CPU

# RAM kullanımı
RAM=$(free | awk '/^Mem:/ {printf "%d", $3/$2*100}')
kontrol_et "RAM" $RAM $ESIK_RAM

# Disk kullanımı
while IFS= read -r SATIR; do
    YUZDE=$(echo "$SATIR" | awk '{print int($5)}')
    BAGLAMA=$(echo "$SATIR" | awk '{print $6}')
    kontrol_et "Disk ($BAGLAMA)" $YUZDE $ESIK_DISK
done < <(df -h | grep "^/dev" | grep -v tmpfs)

# Servisleri kontrol et
for SERVIS in nginx postgresql docker; do
    if systemctl is-active --quiet "$SERVIS" 2>/dev/null; then
        echo "✅ Servis: $SERVIS çalışıyor"
    else
        UYARILAR+=("❌ Servis: $SERVIS ÇALIŞMIYOR")
    fi
done

# Sonuç
if [ ${#UYARILAR[@]} -gt 0 ]; then
    echo ""
    echo "═══ UYARILAR ═══"
    for UYARI in "${UYARILAR[@]}"; do
        echo "$UYARI"
    done
    exit 1
fi

echo ""
echo "✅ Sistem sağlıklı!"
```

---

## Slayt 14 — Script Yazım Kuralları

### ✅ Yapılacaklar

```bash
#!/bin/bash
set -euo pipefail

# Sabitler büyük harf
readonly LOG_DOSYASI="/var/log/script.log"

# Yerel değişkenler fonksiyon içinde
fonksiyon() {
    local degisken="değer"
}

# Değişkenleri çift tırnak içine al
rm -rf "$DIZIN"           # ✅ Boşluk içeriyorsa güvenli
# rm -rf $DIZIN           # ❌ Boşluk varsa ayrı argüman olur

# Hata kontrolü
komut || { echo "Hata!"; exit 1; }

# Geçici dosyaları temizle
GECICI=$(mktemp)
trap "rm -f $GECICI" EXIT
```

### ❌ Kaçınılacaklar

```bash
# Tanımsız değişken kullanma → set -u ile önle
echo $TANIMSIZ

# ls çıktısını parse etme → glob veya find kullan
for f in $(ls *.txt); do ...   # ❌
for f in *.txt; do ...         # ✅

# Gereksiz cat
cat dosya | grep "aranan"      # ❌ UUOC (Useless Use of Cat)
grep "aranan" dosya            # ✅
```

---

## Slayt 15 — Özet

### Öğrendiklerimiz ✅

- Shebang ve script çalıştırma
- Değişkenler: tanımlama, özel değişkenler ($1, $?, $$)
- Kullanıcı girişi alma (read)
- Koşullar: if/elif/else, case
- Döngüler: for, while, break, continue
- Fonksiyonlar ve yerel değişkenler
- Hata yönetimi: set -euo pipefail, trap
- Metin işleme: dilimleme, değiştirme, büyük/küçük harf
- Dosya kilitleme ve geçici dosya yönetimi
- Gerçek dünya örneği: Yedekleme ve sistem sağlık kontrolü

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Bash Referans Kılavuzu | https://www.gnu.org/software/bash/manual/ |
| ShellCheck (hata kontrolü) | https://www.shellcheck.net |
| Advanced Bash Scripting Guide | https://tldp.org/LDP/abs/html/ |
| Bash Kopya Kağıdı | https://devhints.io/bash |
| Güvenli Bash (best practices) | https://github.com/nickel-lang/nickel |

> 💡 **İpucu:** Script'lerini her zaman ShellCheck ile kontrol et!
> `shellcheck myscript.sh` (apt install shellcheck)

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
