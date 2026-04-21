---
title: "Prometheus & Grafana: İzleme ve Gözlemlenebilirlik"
author: "Gülşen Murat"
date: "2024"
---

# 📊 Prometheus & Grafana

**Metrik Toplama, Uyarı ve Görselleştirme**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Gözlemlenebilirlik Nedir?

### Üç Temel Sütun

```
         GÖZLEMLENEBİLİRLİK
        ┌────────────────────┐
        │                    │
   METRİKLER           LOGLAR           İZLER
   (Metrics)           (Logs)          (Traces)
       │                  │                │
  Sayısal veri       Olaylar metin    İstek akışı
  CPU: %45           [ERROR] DB bağl  Req → API → DB
  RAM: 2.1GB         [INFO] Nginx 200 → Önbellek → ...
       │
  Prometheus
       │
   Grafana
```

### Neden İzleme?

| Soru | Araç |
|------|------|
| "Sunucu sağlıklı mı?" | Prometheus + Alertmanager |
| "CPU ne zaman patladı?" | Grafana grafikleri |
| "Hangi endpoint yavaş?" | Grafana + APM |
| "Disk dolmadan uyar!" | Alertmanager → Slack/PagerDuty |

---

## Slayt 2 — Prometheus Nedir?

2012'de SoundCloud tarafından geliştirildi, 2016'da CNCF'e bağışlandı.

### Nasıl Çalışır?

```
Hedef Uygulamalar          Prometheus              Grafana
──────────────────         ──────────              ───────
node-exporter:9100  ←────  /metrics        →────   Dashboard
nginx-exporter:9113  scrape eder  (pull model)     Alertmanager
app:8080/metrics ←────           depolama          → Slack
                    her N saniyede                  → E-posta
                    bir toplar
```

### Temel Özellikler

| Özellik | Detay |
|---------|-------|
| **Pull model** | Prometheus kendisi çeker (push değil) |
| **PromQL** | Güçlü sorgu dili |
| **Yerel depolama** | TSDB (Time Series DB) |
| **Alertmanager** | Uyarı yönlendirme ve susturma |
| **Exporter ekosistemi** | 700+ hazır exporter |

---

## Slayt 3 — Kurulum (Docker Compose)

```yaml
# docker-compose.yml
version: "3.9"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.retention.time=15d"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=güvenli_sifre
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    pid: host
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
```

```bash
docker compose up -d
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3000 (admin/güvenli_sifre)
```

---

## Slayt 4 — Prometheus Yapılandırması

```yaml
# prometheus.yml
global:
  scrape_interval:     15s    # Her 15 saniyede metrik topla
  evaluation_interval: 15s    # Her 15 saniyede kuralları değerlendir

# Uyarı kuralı dosyaları
rule_files:
  - "alerts/*.yml"

# Alertmanager bağlantısı
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

# Hedefler (scrape targets)
scrape_configs:
  # Prometheus kendini izle
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Sunucu sistem metrikleri
  - job_name: "node"
    static_configs:
      - targets:
          - "node-exporter:9100"
          - "web01.orneksite.com:9100"
          - "web02.orneksite.com:9100"

  # Nginx metrikleri
  - job_name: "nginx"
    static_configs:
      - targets: ["nginx-exporter:9113"]

  # Uygulama metrikleri
  - job_name: "uygulama"
    metrics_path: /metrics
    static_configs:
      - targets: ["uygulama:8080"]
```

---

## Slayt 5 — Exporter'lar

### Popüler Exporter'lar

| Exporter | Port | İzlediği |
|----------|------|----------|
| **node_exporter** | 9100 | CPU, RAM, Disk, Ağ |
| **nginx-prometheus-exporter** | 9113 | Nginx bağlantı istatistikleri |
| **postgres_exporter** | 9187 | PostgreSQL metrikleri |
| **redis_exporter** | 9121 | Redis bellek, komutlar |
| **mysqld_exporter** | 9104 | MySQL/MariaDB |
| **blackbox_exporter** | 9115 | HTTP/HTTPS/DNS probe |
| **cadvisor** | 9338 | Docker konteyner metrikleri |

### node_exporter Kurulumu (Bare Metal)

```bash
# İndir ve kur
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xzf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

# Systemd servisi oluştur
sudo tee /etc/systemd/system/node-exporter.service <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now node-exporter
```

---

## Slayt 6 — PromQL (Prometheus Sorgu Dili)

### Temel Sorgular

```promql
# Anlık değer
up                                     # Hedef ayakta mı? (1=evet, 0=hayır)
node_cpu_seconds_total                 # CPU sayacı

# Filtreleme (label)
up{job="nginx"}                        # Sadece nginx jobları
node_cpu_seconds_total{mode="idle"}    # Sadece idle CPU

# Rate — saniye başına artış hızı (son 5 dakika)
rate(node_cpu_seconds_total{mode="idle"}[5m])

# CPU kullanım yüzdesi
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# RAM kullanımı (GB)
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / 1024 / 1024 / 1024

# Disk kullanım yüzdesi
100 - ((node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100)

# HTTP istek hızı (saniyede)
rate(http_requests_total[1m])

# HTTP hata oranı (%)
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
```

---

## Slayt 7 — Uyarı Kuralları (Alerting Rules)

```yaml
# alerts/sunucu-uyarilari.yml
groups:
  - name: sunucu
    interval: 30s          # Kurala özgü değerlendirme aralığı

    rules:
      # Sunucu erişilemez
      - alert: SunucuErisilemez
        expr: up == 0
        for: 2m            # 2 dakika boyunca sürekli tetiklenirse uyar
        labels:
          severity: critical
        annotations:
          summary: "Sunucu erişilemez: {{ $labels.instance }}"
          description: "{{ $labels.instance }} 2 dakikadır erişilemez durumda."

      # Yüksek CPU
      - alert: YuksekCPU
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Yüksek CPU: {{ $labels.instance }}"
          description: "CPU kullanımı 5 dakikadır %{{ $value | humanize }} üzerinde."

      # Disk dolmak üzere
      - alert: DiskDoluyor
        expr: |
          100 - ((node_filesystem_avail_bytes{mountpoint="/"} /
                  node_filesystem_size_bytes{mountpoint="/"}) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Disk dolmak üzere: {{ $labels.instance }}"
          description: "Kök disk %{{ $value | humanize }} dolu."
```

---

## Slayt 8 — Alertmanager

```yaml
# alertmanager.yml
global:
  smtp_from: "uyari@orneksite.com"
  smtp_smarthost: "smtp.gmail.com:587"
  smtp_auth_username: "uyari@orneksite.com"
  smtp_auth_password: "uygulama_sifresi"

route:
  group_by: ["alertname", "instance"]
  group_wait:      30s     # İlk bildirimden önce bekle
  group_interval:  5m      # Gruba yeni uyarı eklenirse bekle
  repeat_interval: 4h      # Tekrar ne zaman bildir
  receiver: "ekip-slack"

  routes:
    - match:
        severity: critical
      receiver: "ekip-slack"
      continue: true        # Diğer rotaları da dene

    - match:
        severity: critical
      receiver: "pagerduty"

receivers:
  - name: "ekip-slack"
    slack_configs:
      - api_url: "https://hooks.slack.com/services/xxx/yyy/zzz"
        channel: "#uyarilar"
        title: "{{ .GroupLabels.alertname }}"
        text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"

  - name: "pagerduty"
    pagerduty_configs:
      - routing_key: "xxx"

  - name: "e-posta"
    email_configs:
      - to: "devops@orneksite.com"

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ["instance"]    # Kritik varsa warning'i sustur
```

---

## Slayt 9 — Grafana: İlk Dashboard

```bash
# Grafana'ya bağlan
# http://localhost:3000
# Kullanıcı: admin / Şifre: güvenli_sifre

# 1. Prometheus data source ekle
#    Configuration → Data Sources → Add data source
#    URL: http://prometheus:9090

# 2. Hazır dashboard import et
#    Dashboards → Import
#    Dashboard ID: 1860  (Node Exporter Full)
#    Dashboard ID: 9614  (NGINX Prometheus Exporter)
#    Dashboard ID: 9628  (PostgreSQL Database)
#    Dashboard ID: 893   (Docker and system monitoring)
```

### Panel Oluşturma

```
Dashboards → New Dashboard → Add panel

Panel query (PromQL):
  100 - (avg by(instance)
    (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

Visualization: Time series
Unit: Percent (0-100)
Thresholds: 80 (warning), 90 (critical)
```

---

## Slayt 10 — Grafana: Uyarılar

```yaml
# Grafana içinden uyarı oluşturma (UI):
# Panel → Alert → Create alert rule

# Örnek kural:
# Condition: avg() of query(A) IS ABOVE 90
# Evaluate every: 1m for 5m
# Notifications: Slack channel #uyarilar
```

### Grafana Alerting (YAML ile — Provisioning)

```yaml
# grafana/provisioning/alerting/cpu-uyarisi.yml
groups:
  - name: sunucu-uyarilari
    folder: Altyapı
    interval: 1m
    rules:
      - uid: cpu-uyarisi-001
        title: "Yüksek CPU Kullanımı"
        condition: C
        data:
          - refId: A
            datasourceUid: prometheus
            model:
              expr: |
                100 - (avg by(instance)
                  (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
          - refId: C
            datasourceUid: "__expr__"
            model:
              type: threshold
              conditions:
                - evaluator:
                    params: [90]
                    type: gt
        noDataState: NoData
        execErrState: Error
        for: 5m
        annotations:
          summary: "Yüksek CPU: {{ $labels.instance }}"
        labels:
          severity: warning
```

---

## Slayt 11 — Uygulama Metrikleri (Custom Metrics)

### Python (prometheus-client)

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Sayaçlar tanımla
http_requests = Counter(
    'http_requests_total',
    'Toplam HTTP istek sayısı',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP istek süresi',
    buckets=[0.1, 0.25, 0.5, 1.0, 2.5]
)

active_connections = Gauge(
    'active_connections',
    'Aktif bağlantı sayısı'
)

# Kullanım
def handle_request(method, endpoint):
    start = time.time()
    active_connections.inc()

    try:
        # ... isteği işle ...
        status = "200"
        http_requests.labels(method=method, endpoint=endpoint, status=status).inc()
    finally:
        duration = time.time() - start
        request_duration.observe(duration)
        active_connections.dec()

# /metrics endpoint'ini başlat (port 8080)
start_http_server(8080)
```

### Node.js (prom-client)

```javascript
const client = require('prom-client');

const httpRequests = new client.Counter({
  name: 'http_requests_total',
  help: 'Toplam HTTP istek sayısı',
  labelNames: ['method', 'route', 'status']
});

app.use((req, res, next) => {
  res.on('finish', () => {
    httpRequests.inc({ method: req.method, route: req.path, status: res.statusCode });
  });
  next();
});

// /metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

---

## Slayt 12 — Grafana Loki (Log İzleme)

```yaml
# docker-compose.yml'e ekle
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - loki-data:/loki
    restart: unless-stopped

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - ./promtail.yml:/etc/promtail/config.yml
    restart: unless-stopped
```

```yaml
# promtail.yml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log

  - job_name: system
    static_configs:
      - targets: [localhost]
        labels:
          job: system
          __path__: /var/log/syslog
```

---

## Slayt 13 — SLI / SLO / SLA

### Kavramlar

| Kavram | Açıklama | Örnek |
|--------|----------|-------|
| **SLI** (Service Level Indicator) | Ölçülen metrik | Başarılı istek oranı |
| **SLO** (Service Level Objective) | Hedef | %99.9 başarı |
| **SLA** (Service Level Agreement) | Sözleşme | %99.5 garanti |
| **Error Budget** | Kaybedebileceğin hata payı | Ayda 43.8 dk |

### PromQL ile SLO Hesaplama

```promql
# Son 30 günlük başarı oranı
sum(rate(http_requests_total{status!~"5.."}[30d]))
/
sum(rate(http_requests_total[30d]))

# Hata bütçesi tüketimi (%)
1 - (
  sum(rate(http_requests_total{status!~"5.."}[30d]))
  /
  sum(rate(http_requests_total[30d]))
)
/ (1 - 0.999)   # %99.9 SLO için
```

---

## Slayt 14 — Özet

### Öğrendiklerimiz ✅

- Gözlemlenebilirliğin 3 sütunu: Metrikler, Loglar, İzler
- Prometheus'un pull-based çalışma modeli
- Docker Compose ile Prometheus + Grafana + node_exporter kurulumu
- prometheus.yml ile scrape hedefleri
- Popüler exporter'lar (node, nginx, postgres, cadvisor)
- PromQL ile güçlü metrik sorguları
- Alerting rules ve Alertmanager yapılandırması
- Slack/PagerDuty/E-posta bildirim kanalları
- Grafana dashboard ve panel oluşturma
- Uygulama içinden custom metrik üretme
- Loki ile log izleme
- SLI/SLO/SLA ve hata bütçesi kavramı

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Prometheus Dokümantasyonu | https://prometheus.io/docs |
| Grafana Dokümantasyonu | https://grafana.com/docs |
| PromQL Cheat Sheet | https://promlabs.com/promql-cheat-sheet |
| Grafana Dashboard Hub | https://grafana.com/grafana/dashboards |
| SLO İyi Pratikler | https://sre.google/sre-book/service-level-objectives |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
