---
title: "PostgreSQL: Veritabanı Yönetimi"
author: "Gülşen Murat"
date: "2024"
---

# 🐘 PostgreSQL

**İlişkisel Veritabanı Yönetimi: Kurulum, Kullanım ve Performans**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — PostgreSQL Nedir?

### PostgreSQL

30 yılı aşkın geliştirme geçmişine sahip, açık kaynaklı **nesne-ilişkisel** veritabanı yönetim sistemi.

```
PostgreSQL

├── ACID Uyumlu        (Atomicity, Consistency, Isolation, Durability)
├── MVCC              (Multi-Version Concurrency Control)
├── JSON / JSONB      (NoSQL yetenekleri)
├── Tam Metin Arama   (Full Text Search)
├── Coğrafi Veri      (PostGIS eklentisi ile)
├── Replikasyon       (Streaming, Logical)
└── Eklenti Sistemi   (pg_stat_statements, pgvector...)
```

### Neden PostgreSQL?

| Özellik | MySQL | PostgreSQL |
|---------|-------|------------|
| JSON desteği | Sınırlı | Tam (JSONB) |
| Full-text search | Basit | Gelişmiş |
| Karmaşık sorgular | Orta | Çok güçlü |
| Eklenti sistemi | Sınırlı | Zengin |
| Lisans | GPL | PostgreSQL (özgür) |

---

## Slayt 2 — Kurulum

### Ubuntu / Debian

```bash
# Resmi PostgreSQL deposunu ekle (daha güncel sürüm için)
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

# Kur
sudo apt install postgresql-16 -y

# Servisi başlat
sudo systemctl enable --now postgresql
sudo systemctl status postgresql
```

### CentOS / AlmaLinux

```bash
# Repo ekle
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql    # Sistem modülünü devre dışı bırak

# Kur
sudo dnf install postgresql16-server -y

# İlk yapılandırma
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb
sudo systemctl enable --now postgresql-16
```

### Docker ile (Geliştirme Ortamı)

```bash
docker run -d \
  --name postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=güvenli_sifre \
  -e POSTGRES_DB=uygulamam \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16

# Bağlan
docker exec -it postgres psql -U admin -d uygulamam
```

---

## Slayt 3 — İlk Bağlantı ve Temel Yapı

```bash
# postgres kullanıcısına geç (sistem kullanıcısı)
sudo -u postgres psql

# veya
sudo -u postgres psql -d veritabani_adi
```

```sql
-- Bağlantı bilgisi
\conninfo

-- Listeler
\l         -- Veritabanları
\du        -- Kullanıcılar/Roller
\dt        -- Tablolar (mevcut db)
\dt *.*    -- Tüm şemalardaki tablolar
\d tablo   -- Tablo yapısı
\dn        -- Şemalar

-- Veritabanı değiştir
\c veritabani_adi

-- Çıkış
\q

-- Yardım
\help SELECT    -- SQL yardımı
\?              -- psql komut yardımı
```

---

## Slayt 4 — Kullanıcı ve Veritabanı Yönetimi

```sql
-- Yeni kullanıcı oluştur
CREATE USER uygulama_kullanicisi WITH PASSWORD 'güvenli_sifre';

-- Şifre değiştir
ALTER USER uygulama_kullanicisi WITH PASSWORD 'yeni_sifre';

-- Süper kullanıcı yap
ALTER USER ali WITH SUPERUSER;

-- Yeni veritabanı oluştur
CREATE DATABASE uygulamam
    OWNER uygulama_kullanicisi
    ENCODING 'UTF8'
    LC_COLLATE 'tr_TR.UTF-8'
    LC_CTYPE 'tr_TR.UTF-8';

-- Yetkileri ver
GRANT ALL PRIVILEGES ON DATABASE uygulamam TO uygulama_kullanicisi;

-- Şemadaki tüm tablolara okuma yetkisi ver
GRANT USAGE ON SCHEMA public TO sadece_oku_kullanici;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO sadece_oku_kullanici;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO sadece_oku_kullanici;

-- Kullanıcıyı sil
DROP USER eski_kullanici;

-- Veritabanını sil
DROP DATABASE eski_veritabani;
```

---

## Slayt 5 — Tablo Oluşturma ve Veri Tipleri

```sql
-- Kullanıcılar tablosu
CREATE TABLE kullanicilar (
    id          SERIAL PRIMARY KEY,           -- Otomatik artan tam sayı
    uuid        UUID DEFAULT gen_random_uuid(), -- Benzersiz ID
    ad          VARCHAR(100) NOT NULL,
    soyad       VARCHAR(100) NOT NULL,
    email       VARCHAR(255) UNIQUE NOT NULL,
    yas         SMALLINT CHECK (yas >= 0 AND yas <= 150),
    aktif       BOOLEAN DEFAULT TRUE,
    puan        NUMERIC(5, 2) DEFAULT 0.00,   -- 5 basamak, 2 ondalık
    meta        JSONB,                         -- JSON veri
    olusturuldu TIMESTAMPTZ DEFAULT NOW(),     -- Saat dilimi ile
    guncellendi TIMESTAMPTZ DEFAULT NOW()
);

-- İndeks ekle
CREATE INDEX idx_kullanicilar_email ON kullanicilar(email);
CREATE INDEX idx_kullanicilar_aktif ON kullanicilar(aktif) WHERE aktif = TRUE;

-- Tablo değiştir
ALTER TABLE kullanicilar ADD COLUMN telefon VARCHAR(20);
ALTER TABLE kullanicilar DROP COLUMN telefon;
ALTER TABLE kullanicilar ALTER COLUMN ad SET NOT NULL;
ALTER TABLE kullanicilar RENAME COLUMN ad TO isim;

-- Tablo sil
DROP TABLE kullanicilar;
DROP TABLE IF EXISTS kullanicilar CASCADE;
```

---

## Slayt 6 — CRUD İşlemleri

```sql
-- INSERT — Veri ekle
INSERT INTO kullanicilar (ad, soyad, email, yas)
VALUES ('Gülşen', 'Murat', 'gulsenmurat@email.com', 28);

-- Toplu ekleme
INSERT INTO kullanicilar (ad, soyad, email) VALUES
    ('Ali', 'Kaya', 'ali@email.com'),
    ('Ayşe', 'Demir', 'ayse@email.com'),
    ('Mehmet', 'Yılmaz', 'mehmet@email.com');

-- Çakışmada güncelle (Upsert)
INSERT INTO kullanicilar (email, ad, soyad)
VALUES ('ali@email.com', 'Ali', 'Yeni Soyad')
ON CONFLICT (email) DO UPDATE
    SET soyad = EXCLUDED.soyad,
        guncellendi = NOW();

-- SELECT — Veri oku
SELECT * FROM kullanicilar;
SELECT id, ad, email FROM kullanicilar WHERE aktif = TRUE;
SELECT * FROM kullanicilar ORDER BY olusturuldu DESC LIMIT 10;
SELECT * FROM kullanicilar WHERE ad ILIKE '%gül%';  -- Büyük/küçük duyarsız arama

-- UPDATE — Veri güncelle
UPDATE kullanicilar SET aktif = FALSE WHERE email = 'eski@email.com';
UPDATE kullanicilar
SET puan = puan + 10, guncellendi = NOW()
WHERE id = 1
RETURNING *;    -- Güncellenen satırı döndür

-- DELETE — Veri sil
DELETE FROM kullanicilar WHERE id = 5;
DELETE FROM kullanicilar WHERE aktif = FALSE RETURNING id;
```

---

## Slayt 7 — JOIN ve İlişkiler

```sql
-- Tablolar
CREATE TABLE siparisler (
    id          SERIAL PRIMARY KEY,
    kullanici_id INT REFERENCES kullanicilar(id) ON DELETE CASCADE,
    toplam      NUMERIC(10, 2),
    tarih       DATE DEFAULT CURRENT_DATE,
    durum       TEXT DEFAULT 'bekliyor'
);

CREATE TABLE urunler (
    id    SERIAL PRIMARY KEY,
    ad    VARCHAR(200) NOT NULL,
    fiyat NUMERIC(10, 2)
);

CREATE TABLE siparis_kalemleri (
    siparis_id INT REFERENCES siparisler(id),
    urun_id    INT REFERENCES urunler(id),
    miktar     INT DEFAULT 1,
    PRIMARY KEY (siparis_id, urun_id)
);

-- INNER JOIN — Her iki tarafta da olmalı
SELECT k.ad, k.email, COUNT(s.id) AS siparis_sayisi
FROM kullanicilar k
INNER JOIN siparisler s ON k.id = s.kullanici_id
GROUP BY k.id, k.ad, k.email
ORDER BY siparis_sayisi DESC;

-- LEFT JOIN — Sol taraf her zaman gelir
SELECT k.ad, k.email, COALESCE(COUNT(s.id), 0) AS siparis_sayisi
FROM kullanicilar k
LEFT JOIN siparisler s ON k.id = s.kullanici_id
GROUP BY k.id, k.ad, k.email;

-- Çok tablolu birleştirme
SELECT k.ad, u.ad AS urun, sk.miktar, u.fiyat * sk.miktar AS tutar
FROM kullanicilar k
JOIN siparisler s ON k.id = s.kullanici_id
JOIN siparis_kalemleri sk ON s.id = sk.siparis_id
JOIN urunler u ON sk.urun_id = u.id
WHERE s.durum = 'tamamlandi';
```

---

## Slayt 8 — Gelişmiş Sorgular

```sql
-- WITH (CTE — Common Table Expression)
WITH aylik_satis AS (
    SELECT
        DATE_TRUNC('month', tarih) AS ay,
        SUM(toplam) AS ciro
    FROM siparisler
    WHERE durum = 'tamamlandi'
    GROUP BY ay
),
onceki_ay AS (
    SELECT *, LAG(ciro) OVER (ORDER BY ay) AS onceki_ciro
    FROM aylik_satis
)
SELECT
    ay,
    ciro,
    ROUND((ciro - onceki_ciro) / onceki_ciro * 100, 2) AS buyume_yuzde
FROM onceki_ay;

-- Pencere Fonksiyonları (Window Functions)
SELECT
    ad,
    puan,
    RANK() OVER (ORDER BY puan DESC) AS sira,
    DENSE_RANK() OVER (ORDER BY puan DESC) AS sira2,
    NTILE(4) OVER (ORDER BY puan DESC) AS ceyrek,
    AVG(puan) OVER () AS genel_ortalama
FROM kullanicilar
WHERE aktif = TRUE;

-- JSONB sorguları
SELECT * FROM kullanicilar
WHERE meta->>'sehir' = 'İstanbul';

SELECT ad, meta->'tercihler'->>'dil' AS dil
FROM kullanicilar
WHERE meta @> '{"premium": true}';
```

---

## Slayt 9 — İndeksler ve Performans

```sql
-- İndeks türleri
CREATE INDEX idx_email ON kullanicilar(email);                  -- B-tree (varsayılan)
CREATE INDEX idx_ad_gin ON kullanicilar USING GIN(meta);        -- JSONB için
CREATE INDEX idx_metin ON belgeler USING GIN(to_tsvector('turkish', icerik));  -- Full-text

-- Kısmi indeks (partial index)
CREATE INDEX idx_aktif_kullanicilar ON kullanicilar(email)
WHERE aktif = TRUE;

-- Bileşik indeks
CREATE INDEX idx_durum_tarih ON siparisler(durum, tarih DESC);

-- İndeks listele
\di+ kullanicilar_*

-- Yavaş sorguları bul (pg_stat_statements gerekir)
SELECT query, calls, total_exec_time/calls AS ort_sure_ms, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Sorgu planı (EXPLAIN)
EXPLAIN SELECT * FROM kullanicilar WHERE email = 'test@email.com';
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM kullanicilar WHERE email = 'test@email.com';
-- → Seq Scan (kötü) vs Index Scan (iyi)

-- Tablo istatistiklerini güncelle
ANALYZE kullanicilar;
VACUUM ANALYZE kullanicilar;    -- Ölü satırları temizle + istatistik güncelle
```

---

## Slayt 10 — Yedekleme ve Geri Yükleme

```bash
# ── pg_dump ile yedekleme ─────────────────────────────────

# Tek veritabanı (SQL formatı)
pg_dump -U postgres -d uygulamam -f yedek.sql

# Sıkıştırılmış format (önerilen)
pg_dump -U postgres -d uygulamam -F c -f yedek.dump

# Sadece şema (veri yok)
pg_dump -U postgres -d uygulamam --schema-only -f sema.sql

# Sadece veri (şema yok)
pg_dump -U postgres -d uygulamam --data-only -f veri.sql

# Uzak sunucudan al
pg_dump -h sunucu-ip -U postgres -d uygulamam -F c -f yedek.dump

# ── Geri yükleme ──────────────────────────────────────────

# SQL dosyasından
psql -U postgres -d uygulamam < yedek.sql

# Özel formattan (pg_restore)
pg_restore -U postgres -d uygulamam -F c yedek.dump

# Paralel geri yükleme (daha hızlı)
pg_restore -U postgres -d uygulamam -j 4 yedek.dump

# ── Otomatik yedekleme scripti ───────────────────────────
# /scripts/postgres-yedek.sh
TARIH=$(date +%Y%m%d_%H%M%S)
pg_dump -U postgres uygulamam -F c -f /var/backups/pg_${TARIH}.dump
find /var/backups -name "pg_*.dump" -mtime +7 -delete
```

---

## Slayt 11 — postgresql.conf Temel Ayarlar

```ini
# /etc/postgresql/16/main/postgresql.conf

# Bağlantı
listen_addresses = '*'          # Tüm IP'lerden kabul (güvenlik duvarı koru!)
max_connections = 100           # Maksimum eş zamanlı bağlantı
port = 5432

# Bellek (RAM'in ~%25'i)
shared_buffers = 256MB          # Paylaşılan önbellek
work_mem = 4MB                  # Sorgu başına bellek
maintenance_work_mem = 64MB     # VACUUM, CREATE INDEX için

# Disk I/O
effective_cache_size = 1GB      # OS disk önbelleği tahmini (RAM'in ~%75)
wal_buffers = 16MB              # WAL yazmak için tampon

# Bağlantı havuzu için önerilen araç
# PgBouncer (transaction pooler) kullan

# Loglama
log_destination = 'stderr'
logging_collector = on
log_directory = '/var/log/postgresql'
log_filename = 'postgresql-%Y-%m-%d.log'
log_min_duration_statement = 1000  # 1 saniyeden uzun sorguları logla (ms)
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d '
```

```ini
# /etc/postgresql/16/main/pg_hba.conf (İstemci kimlik doğrulama)
# TYPE  DATABASE    USER              ADDRESS         METHOD
local   all         postgres                          peer
local   all         all                               md5
host    uygulamam   uygulama_kull    10.0.0.0/8      scram-sha-256
host    all         all              0.0.0.0/0        reject  # Varsayılan: hepsini reddet
```

---

## Slayt 12 — Replikasyon (Temel)

```bash
# Birincil (Primary) sunucu — postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64MB

# pg_hba.conf — replikasyon bağlantısı izni
host replication replika_kullanici 10.0.0.2/32 scram-sha-256
```

```sql
-- Replikasyon kullanıcısı oluştur
CREATE USER replika_kullanici
    WITH REPLICATION
    PASSWORD 'güvenli_sifre';
```

```bash
# İkincil (Standby) sunucu — ilk senkronizasyon
pg_basebackup -h birincil-ip -U replika_kullanici \
    -D /var/lib/postgresql/16/main \
    -P -R -X stream

# -R: recovery.conf otomatik oluştur
# Standby otomatik olarak birincili takip eder

# Replikasyon durumunu kontrol et (birincilde)
SELECT * FROM pg_stat_replication;
```

---

## Slayt 13 — Faydalı Sorgular

```sql
-- Büyük tablolar
SELECT
    schemaname || '.' || tablename AS tablo,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS boyut
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- Aktif bağlantılar
SELECT pid, usename, application_name, client_addr, state, query_start, query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

-- Uzun süren sorgular
SELECT pid, now() - pg_stat_activity.query_start AS sure, query, state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';

-- Kilitler (Locks)
SELECT bl.pid AS engellenen_pid, a.query AS engellenen_sorgu,
       kl.pid AS kilitleyen_pid, ka.query AS kilitleyen_sorgu
FROM pg_catalog.pg_locks bl
JOIN pg_catalog.pg_stat_activity a ON a.pid = bl.pid
JOIN pg_catalog.pg_locks kl ON kl.locktype = bl.locktype AND kl.granted AND NOT bl.granted
JOIN pg_catalog.pg_stat_activity ka ON ka.pid = kl.pid;

-- Kilidi sonlandır
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;
```

---

## Slayt 14 — Özet

### Öğrendiklerimiz ✅

- PostgreSQL'in güçlü yönleri (ACID, MVCC, JSONB)
- Ubuntu/CentOS kurulumu ve Docker ile çalıştırma
- Kullanıcı, veritabanı ve yetki yönetimi
- Tablo tasarımı: veri tipleri, kısıtlar, indeksler
- CRUD işlemleri ve Upsert
- JOIN türleri ve ilişkisel veri tasarımı
- CTE ve pencere fonksiyonları
- EXPLAIN ile sorgu analizi ve optimizasyon
- pg_dump / pg_restore ile yedekleme
- postgresql.conf temel ayarlar
- Replikasyon temelleri

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi PostgreSQL Dokümantasyonu | https://www.postgresql.org/docs |
| PostgreSQL Cheat Sheet | https://www.postgresqltutorial.com/postgresql-cheat-sheet |
| Explain Görselleştirici | https://explain.depesz.com |
| pgAdmin (GUI arayüz) | https://www.pgadmin.org |
| Türkçe PostgreSQL Kaynakları | https://wiki.postgresql.org/wiki/Translated_PostgreSQL_Documentation |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
