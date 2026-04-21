# 📚 Sunumlar

Teknik konularda hazırlanmış sunum materyalleri.

---

## İçindekiler

| # | Konu | Dosya | Slayt Sayısı |
|---|------|-------|-------------|
| 01 | 🐧 Linux Temelleri | [01-linux-temelleri.md](./01-linux-temelleri.md) | 20 |
| 02 | 🚀 Nginx (Slayt) | [02-nginx-slaytlar.md](./02-nginx-slaytlar.md) | 14 |
| 03 | 🐳 Docker | [03-docker.md](./03-docker.md) | 16 |
| 04 | 🌿 Git | [04-git.md](./04-git.md) | 17 |
| 05 | ☸️ Kubernetes | [05-kubernetes.md](./05-kubernetes.md) | 15 |
| 06 | ⚙️ GitHub Actions (CI/CD) | [06-github-actions.md](./06-github-actions.md) | 15 |
| 07 | 🔧 Ansible | [07-ansible.md](./07-ansible.md) | 14 |
| 08 | 💻 Bash Scripting | [08-bash-scripting.md](./08-bash-scripting.md) | 15 |
| 09 | 🏗️ Terraform | [09-terraform.md](./09-terraform.md) | 15 |
| 10 | 📊 Prometheus & Grafana | [10-prometheus-grafana.md](./10-prometheus-grafana.md) | 14 |
| 11 | 🐘 PostgreSQL | [11-postgresql.md](./11-postgresql.md) | 14 |
| 12 | 🔐 Linux Sunucu Güvenliği | [12-guvenlik.md](./12-guvenlik.md) | 14 |

> **Not:** Nginx için kapsamlı teknik referans dökümanı ana dizinde [`nginx-sunum.md`](../nginx-sunum.md) olarak da mevcuttur.

---

## Konu Özeti

### 🐧 01 — Linux Temelleri
Sıfırdan Linux sistem yönetimine giriş. Terminal kullanımı, dosya sistemi, kullanıcı yönetimi, paket yönetimi, süreç kontrolü, ağ komutları, metin işleme araçları ve SSH konularını kapsar.

### 🚀 02 — Nginx
Nginx'in olay tabanlı mimarisi, Ubuntu/CentOS kurulumu, Virtual Host yapılandırması, SSL/TLS, Reverse Proxy, Load Balancer, Rate Limiting ve yaygın hata çözümleri.

### 🐳 03 — Docker
Konteyner teknolojisinin temelleri, Docker kurulumu, temel komutlar, Dockerfile yazımı, Volume yönetimi, Docker Compose ile çok servisli uygulama yönetimi.

### 🌿 04 — Git
Versiyon kontrolünün temelleri, ilk yapılandırma, commit akışı, dal yönetimi, çakışma çözümü, GitHub SSH bağlantısı, Conventional Commits ve Git workflow stratejileri.

### ☸️ 05 — Kubernetes
Konteyner orkestrasyonu, Master/Worker mimarisi, Pod/Deployment/Service/Namespace kavramları, kubectl komutları, Rolling Update, Rollback, HPA ile otomatik ölçeklendirme, ConfigMap/Secret, PersistentVolume, Ingress ve Helm.

### ⚙️ 06 — GitHub Actions (CI/CD)
CI/CD kavramları, Workflow/Job/Step/Action yapısı, tetikleyici olaylar, Secret yönetimi, Matrix build, Docker build & push, SSH ile deployment, Cache, Artifact saklama, Reusable Workflows ve güvenlik pratikleri.

### 🔧 07 — Ansible
Agentsız yapılandırma yönetimi, Inventory tanımları, Ad hoc komutlar, Playbook yazımı, Jinja2 şablonları, döngüler/koşullar, Roller (Roles), Ansible Galaxy ve Vault ile gizli bilgi şifreleme.

### 💻 08 — Bash Scripting
Shebang ve çalıştırma, değişkenler, kullanıcı girişi, koşullar (if/case), döngüler (for/while), fonksiyonlar, hata yönetimi (set -euo pipefail, trap), metin işleme, dosya kilitleme, yedekleme ve sağlık kontrolü scriptleri.

### 🏗️ 09 — Terraform
Infrastructure as Code kavramı, Provider yapılandırması (AWS), temel komutlar (init/plan/apply/destroy), Resources ile altyapı tanımlama, Variables/Outputs, Modules, count/for_each döngüleri, State yönetimi, Workspaces ve GitHub Actions CI/CD entegrasyonu.

### 📊 10 — Prometheus & Grafana
Gözlemlenebilirliğin 3 sütunu, Prometheus pull-based çalışma modeli, Docker Compose kurulum, prometheus.yml yapılandırması, Exporter'lar (node/nginx/postgres/cadvisor), PromQL sorgu dili, Alerting Rules, Alertmanager (Slack/PagerDuty), Grafana dashboard oluşturma, Custom metrikler, Loki ile log izleme, SLI/SLO/SLA.

### 🐘 11 — PostgreSQL
Kurulum (Ubuntu/CentOS/Docker), kullanıcı ve yetki yönetimi, tablo tasarımı (veri tipleri, kısıtlar, indeksler), CRUD işlemleri ve Upsert, JOIN türleri, CTE ve pencere fonksiyonları, EXPLAIN ile sorgu analizi, pg_dump/pg_restore ile yedekleme, postgresql.conf ayarları, replikasyon temelleri.

### 🔐 12 — Linux Sunucu Güvenliği
Defense in Depth prensibi, SSH hardening (anahtar tabanlı giriş, root engel), UFW/firewalld güvenlik duvarı, Fail2ban brute-force koruması, otomatik güvenlik güncellemeleri, sudo kısıtlama ve denetim logları, auditd, Lynis ve rkhunter güvenlik taraması, Systemd güvenlik kısıtlamaları, güvenlik kontrol listesi.

---

*Hazırlayan: Gülşen Murat — 2024*
