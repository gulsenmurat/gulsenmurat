---
title: "GitHub Actions: CI/CD"
author: "Gülşen Murat"
date: "2024"
---

# ⚙️ GitHub Actions

**Sürekli Entegrasyon ve Sürekli Dağıtım (CI/CD)**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — CI/CD Nedir?

### Sürekli Entegrasyon (CI)

Her kod değişikliğinde otomatik olarak:
- Testleri çalıştır
- Kodu derle / build et
- Kod kalitesini kontrol et (lint)

### Sürekli Dağıtım (CD)

Test geçen kodu otomatik olarak:
- Test ortamına dağıt
- Üretim ortamına dağıt

```
Geliştirici     GitHub          CI (Build+Test)    CD (Deploy)
    │               │                 │                 │
    │──git push────►│                 │                 │
    │               │──tetikle───────►│                 │
    │               │                 │──test geçti────►│
    │               │                 │                 │──dağıt──►sunucu
    │◄──bildirim──────────────────────────────────────── │
```

---

## Slayt 2 — GitHub Actions Nedir?

GitHub'ın **yerleşik CI/CD platformu**.

### Temel Kavramlar

| Kavram | Açıklama |
|--------|----------|
| **Workflow** | Otomasyonun tamamı (`.github/workflows/*.yml`) |
| **Event** | Workflow'u tetikleyen olay (`push`, `PR`, `schedule`) |
| **Job** | Bir veya fazla adımdan oluşan iş birimi |
| **Step** | Tek bir komut veya Action |
| **Action** | Hazır, yeniden kullanılabilir adım |
| **Runner** | Workflow'un çalıştığı sanal makine |

### Neden GitHub Actions?

- ✅ GitHub reposuna entegre — ek kurulum yok
- ✅ Ücretsiz (public repo: sınırsız, private: 2000 dk/ay)
- ✅ 10.000+ hazır Action (GitHub Marketplace)
- ✅ Linux, Windows, macOS runner desteği
- ✅ Kendi runner'ını ekleyebilirsin (self-hosted)

---

## Slayt 3 — İlk Workflow

```
repo/
└── .github/
    └── workflows/
        └── ci.yml       ← Workflow dosyaları buraya
```

### .github/workflows/ci.yml

```yaml
name: CI Pipeline

on:                         # Tetikleyici olaylar
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:                     # Job adı
    runs-on: ubuntu-latest  # Runner seçimi

    steps:
    - name: Kodu al
      uses: actions/checkout@v4

    - name: Node.js kur
      uses: actions/setup-node@v4
      with:
        node-version: '18'

    - name: Bağımlılıkları yükle
      run: npm ci

    - name: Testleri çalıştır
      run: npm test

    - name: Lint kontrol
      run: npm run lint
```

---

## Slayt 4 — Tetikleyici Olaylar (on)

```yaml
on:
  # Push olayları
  push:
    branches: [ main, 'release/*' ]
    paths:                          # Sadece bu dizin değişince
      - 'src/**'
      - 'tests/**'

  # Pull Request olayları
  pull_request:
    branches: [ main ]
    types: [ opened, synchronize, reopened ]

  # Zamanlanmış çalıştırma
  schedule:
    - cron: '0 8 * * 1-5'          # Hafta içi her sabah 08:00

  # Manuel tetikleme
  workflow_dispatch:
    inputs:
      ortam:
        description: 'Hangi ortama dağıt?'
        required: true
        default: 'staging'
        type: choice
        options: [ staging, production ]

  # Başka workflow tamamlandığında
  workflow_run:
    workflows: [ "CI Pipeline" ]
    types: [ completed ]
```

---

## Slayt 5 — Ortam Değişkenleri ve Gizli Bilgiler

### Secrets Ayarlama

GitHub → Repo → Settings → Secrets and variables → Actions

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Deploy
      env:
        API_KEY: ${{ secrets.API_KEY }}         # Repo secret
        DB_URL: ${{ secrets.DATABASE_URL }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Otomatik oluşturulur

      run: |
        echo "API anahtarı: $API_KEY"
        ./deploy.sh
```

### Ortam Değişkenleri

```yaml
env:                                    # Tüm workflow için
  NODE_ENV: production
  APP_VERSION: 1.0.0

jobs:
  build:
    env:                                # Bu job için
      BUILD_DIR: ./dist
    steps:
    - run: echo "Sürüm: $APP_VERSION"
```

---

## Slayt 6 — Matrix Build (Çoklu Ortam)

Aynı kodu farklı OS / Node sürümlerinde test et:

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ ubuntu-latest, macos-latest, windows-latest ]
        node: [ 16, 18, 20 ]
        exclude:
          - os: windows-latest
            node: 16

    steps:
    - uses: actions/checkout@v4

    - name: Node.js ${{ matrix.node }} kur
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node }}

    - run: npm ci && npm test
```

```
# Yukarıdaki matrix: 3 OS × 3 Node - 1 exclude = 8 paralel job
ubuntu-latest + Node 16 | ubuntu-latest + Node 18 | ubuntu-latest + Node 20
macos-latest  + Node 16 | macos-latest  + Node 18 | macos-latest  + Node 20
                           windows-latest + Node 18 | windows-latest + Node 20
```

---

## Slayt 7 — Docker Build ve Registry

```yaml
name: Docker Build & Push

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Docker meta (tag hesapla)
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: gulsenmurat/benim-uygulama
        tags: |
          type=semver,pattern={{version}}
          type=sha,prefix=sha-

    - name: Docker Hub'a giriş yap
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build ve Push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        cache-from: type=gha          # GitHub Cache kullan
        cache-to: type=gha,mode=max
```

---

## Slayt 8 — Deployment (SSH ile Sunucuya)

```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: test              # Önce test job'u geçmeli

    steps:
    - uses: actions/checkout@v4

    - name: Sunucuya deploy et
      uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/benim-uygulama
          git pull origin main
          npm ci --production
          pm2 restart uygulama
          echo "Deploy tamamlandı ✅"

    - name: Slack bildirimi gönder
      if: always()
      uses: slackapi/slack-github-action@v1
      with:
        payload: |
          {
            "text": "Deploy durumu: ${{ job.status }}"
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Slayt 9 — Cache (Önbellekleme)

```yaml
steps:
- uses: actions/checkout@v4

# Node.js bağımlılıklarını önbellekle
- name: npm cache
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- run: npm ci

# ─────────────────────────────────────────────

# Python bağımlılıklarını önbellekle
- name: pip cache
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

- run: pip install -r requirements.txt

# ─────────────────────────────────────────────

# Maven önbellek
- name: Maven cache
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

---

## Slayt 10 — Artifact ve Rapor Saklama

```yaml
steps:
- run: npm run build

# Build çıktısını sakla
- name: Build artifact'ı yükle
  uses: actions/upload-artifact@v4
  with:
    name: dist-dosyalari
    path: dist/
    retention-days: 7             # 7 gün sakla

# Başka job'da indir
- name: Artifact'ı indir
  uses: actions/download-artifact@v4
  with:
    name: dist-dosyalari
    path: dist/

# Test raporları
- name: Jest raporu
  uses: actions/upload-artifact@v4
  if: always()                    # Test başarısız olsa da yükle
  with:
    name: test-sonuclari
    path: coverage/

# GitHub PR'a test özeti ekle
- name: Test özeti
  uses: dorny/test-reporter@v1
  if: always()
  with:
    name: Jest Tests
    path: 'test-results/*.xml'
    reporter: jest-junit
```

---

## Slayt 11 — Tam CI/CD Pipeline Örneği

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # ── 1. Kod kalitesi ──────────────────────────────
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with: { node-version: '18', cache: 'npm' }
    - run: npm ci
    - run: npm run lint

  # ── 2. Test ──────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with: { node-version: '18', cache: 'npm' }
    - run: npm ci && npm test -- --coverage

  # ── 3. Docker build ──────────────────────────────
  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
    - uses: actions/checkout@v4
    - uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - uses: docker/build-push-action@v5
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
        tags: gulsenmurat/app:${{ github.sha }}

  # ── 4. Staging deploy ─────────────────────────────
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
    - uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: deploy
        key: ${{ secrets.SSH_KEY }}
        script: |
          docker pull gulsenmurat/app:${{ github.sha }}
          docker-compose up -d
```

---

## Slayt 12 — Reusable Workflows (Yeniden Kullanılabilir)

```yaml
# .github/workflows/deploy-template.yml
name: Deploy Template

on:
  workflow_call:                  # Başka workflow tarafından çağrılabilir
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      SSH_KEY:
        required: true
      SERVER_HOST:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
    - uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.SERVER_HOST }}
        key: ${{ secrets.SSH_KEY }}
        username: deploy
        script: |
          docker pull gulsenmurat/app:${{ inputs.image-tag }}
          docker-compose up -d
```

```yaml
# .github/workflows/main.yml — Çağıran workflow
jobs:
  deploy:
    uses: ./.github/workflows/deploy-template.yml
    with:
      environment: production
      image-tag: ${{ github.sha }}
    secrets:
      SSH_KEY: ${{ secrets.PROD_SSH_KEY }}
      SERVER_HOST: ${{ secrets.PROD_HOST }}
```

---

## Slayt 13 — Güvenlik İpuçları

```yaml
# 1. En az yetki prensibi
permissions:
  contents: read          # Sadece okuma
  packages: write         # Sadece gerekli

# 2. Action'ların commit SHA'sını sabitle (güvenlik)
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2

# 3. Kod güvenlik taraması
- name: CodeQL analizi
  uses: github/codeql-action/analyze@v3

# 4. Bağımlılık güvenlik taraması
- name: Dependency Review
  uses: actions/dependency-review-action@v4

# 5. Secret'ları loglamaktan kaçın
- run: |
    # ❌ Yapma
    echo "Key: ${{ secrets.API_KEY }}"
    # ✅ Yap
    ./script.sh    # Script içinde $API_KEY kullan
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

---

## Slayt 14 — Hata Ayıklama

```yaml
# İş tamamlanınca (başarılı veya başarısız)
- name: Temizlik yap
  if: always()

# Sadece başarısızlıkta
- name: Slack'e hata bildir
  if: failure()

# Sadece başarıda
- name: Başarı bildirimi
  if: success()

# Adımları atla ama job başarılı say
- name: Opsiyonel adım
  continue-on-error: true
  run: npm run optional-check

# Workflow debug modu
# Repo → Settings → Secrets → ACTIONS_STEP_DEBUG = true
# veya manuel tetiklemede:
- name: Debug bilgisi
  run: |
    echo "Branch: ${{ github.ref }}"
    echo "SHA: ${{ github.sha }}"
    echo "Actor: ${{ github.actor }}"
    echo "Event: ${{ github.event_name }}"
```

---

## Slayt 15 — Özet

### Öğrendiklerimiz ✅

- CI/CD kavramları ve faydaları
- GitHub Actions yapısı: Workflow, Job, Step, Action
- Tetikleyici olaylar: push, PR, schedule, workflow_dispatch
- Secret ve ortam değişkeni yönetimi
- Matrix build ile çoklu ortam testi
- Docker build ve Registry'e push
- SSH ile uzak sunucuya deployment
- Cache ile build hızlandırma
- Artifact ve test raporu saklama
- Reusable workflow ile tekrar kullanım
- Güvenlik pratikleri

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| GitHub Actions Dokümantasyonu | https://docs.github.com/en/actions |
| GitHub Marketplace (Actions) | https://github.com/marketplace?type=actions |
| Workflow Sözdizimi Referansı | https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions |
| Actions Güvenlik İpuçları | https://docs.github.com/en/actions/security-guides |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
