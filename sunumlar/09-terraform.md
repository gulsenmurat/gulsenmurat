---
title: "Terraform: Infrastructure as Code"
author: "Gülşen Murat"
date: "2024"
---

# 🏗️ Terraform

**Infrastructure as Code: Altyapını Kodla Yönet**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Infrastructure as Code (IaC) Nedir?

### Geleneksel vs IaC

```
Geleneksel Yöntem                Infrastructure as Code
──────────────────               ───────────────────────
Konsoldan sunucu oluştur         Kod yaz → terraform apply
Manuel güvenlik grubu ekle       Her şey otomatik oluşur
Başka bölgede tekrar yap         Aynı kodu farklı bölgede çalıştır
"Neyi neden kurdum?" ─ Bilmem    Git geçmişinde her değişiklik
```

### IaC'nin Faydaları

| Fayda | Açıklama |
|-------|----------|
| **Tekrarlanabilirlik** | Aynı altyapıyı defalarca kur |
| **Versiyon kontrolü** | Git ile altyapı geçmişi |
| **İşbirliği** | Ekip aynı kod üzerinde çalışır |
| **Belgeleme** | Kod = canlı belge |
| **Hızlı silme/kurma** | Test ortamını 5 dakikada kur/sil |

---

## Slayt 2 — Terraform Nedir?

HashiCorp tarafından geliştirilen açık kaynaklı IaC aracı.

```
Terraform

├── AWS       (EC2, S3, RDS, VPC...)
├── GCP       (Compute Engine, GKE, Cloud SQL...)
├── Azure     (VM, AKS, Storage...)
├── Docker    (Container, Image, Network...)
├── Kubernetes (Deployment, Service, Ingress...)
└── 3000+     Provider (GitHub, Cloudflare, Datadog...)
```

### Nasıl Çalışır?

```
.tf dosyaları         terraform plan          terraform apply
─────────────    →    ─────────────────   →   ────────────────
Kaynak tanımla        Ne değişecek gör        Gerçek altyapıyı kur
```

### HCL (HashiCorp Configuration Language)

```hcl
# Basit bir kaynak tanımı
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = {
    Name = "web-sunucu"
  }
}
```

---

## Slayt 3 — Kurulum

### Linux (Ubuntu/Debian)

```bash
# HashiCorp GPG anahtarı ve repo ekle
wget -O- https://apt.releases.hashicorp.com/gpg \
    | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
    | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform -y

# Sürüm kontrolü
terraform version
```

### macOS

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### tfenv (Sürüm Yöneticisi — Önerilen)

```bash
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

tfenv install 1.6.0      # Belirli sürüm kur
tfenv use 1.6.0          # Kullanılacak sürümü seç
tfenv list               # Kurulu sürümler
```

---

## Slayt 4 — Temel Komutlar

```bash
# Proje başlat (provider'ları indir)
terraform init

# Değişiklikleri önizle
terraform plan

# Planı dosyaya kaydet
terraform plan -out=tfplan

# Kaydedilen planı uygula
terraform apply tfplan

# Değişiklikleri uygula (onay ister)
terraform apply

# Onaysız uygula (dikkatli!)
terraform apply -auto-approve

# Tüm altyapıyı sil
terraform destroy

# Mevcut altyapının durumunu göster
terraform show

# Durum dosyasını listele
terraform state list
terraform state show aws_instance.web

# Bozuk kaynağı güncelle (konsolla fark varsa)
terraform refresh

# Kod biçimlendir
terraform fmt

# Sözdizimi doğrula
terraform validate
```

---

## Slayt 5 — Provider Yapılandırması

```hcl
# main.tf
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"        # 5.x sürümlerini kullan
    }
  }

  # Uzak durum dosyası (ekip çalışması için)
  backend "s3" {
    bucket = "tfstate-gulsenmurat"
    key    = "uretim/terraform.tfstate"
    region = "eu-central-1"
  }
}

# Provider yapılandırması
provider "aws" {
  region = "eu-central-1"       # Frankfurt

  default_tags {
    tags = {
      Project   = "benim-projem"
      ManagedBy = "Terraform"
    }
  }
}
```

```bash
# AWS kimlik bilgileri (önerilen yol: env var)
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="xxxx"

# veya AWS CLI profili
aws configure --profile terraform-kullanicisi
export AWS_PROFILE=terraform-kullanicisi
```

---

## Slayt 6 — Kaynaklar (Resources)

```hcl
# VPC
resource "aws_vpc" "ana" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name = "ana-vpc"
  }
}

# Subnet
resource "aws_subnet" "genel" {
  vpc_id            = aws_vpc.ana.id    # Referans: başka kaynağa bağla
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-central-1a"

  tags = {
    Name = "genel-subnet"
  }
}

# Güvenlik Grubu
resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.ana.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"          # Tüm protokoller
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami                    = "ami-0faab6bdbac9486fb"  # Ubuntu 22.04 Frankfurt
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.genel.id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = "benim-ssh-anahtarim"

  user_data = <<-EOF
    #!/bin/bash
    apt update && apt install -y nginx
    systemctl enable --now nginx
  EOF

  tags = {
    Name = "web-sunucu"
  }
}
```

---

## Slayt 7 — Değişkenler (Variables)

```hcl
# variables.tf
variable "bolge" {
  description = "AWS bölgesi"
  type        = string
  default     = "eu-central-1"
}

variable "instance_type" {
  description = "EC2 instance türü"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "instance_type t3.micro, t3.small veya t3.medium olmalı."
  }
}

variable "etiketler" {
  description = "Kaynaklara uygulanacak etiketler"
  type        = map(string)
  default = {
    Environment = "staging"
    Team        = "devops"
  }
}

variable "db_parolasi" {
  description = "Veritabanı parolası"
  type        = string
  sensitive   = true    # Plan/apply çıktısında gizle
}
```

```bash
# Değişken geçirme yolları
terraform apply -var="instance_type=t3.small"
terraform apply -var-file="uretim.tfvars"

# TF_VAR_ ortam değişkeni
export TF_VAR_db_parolasi="gizli_parola"
terraform apply
```

```hcl
# uretim.tfvars
bolge         = "eu-west-1"
instance_type = "t3.medium"
etiketler = {
  Environment = "production"
  Team        = "platform"
}
```

---

## Slayt 8 — Çıktılar (Outputs)

```hcl
# outputs.tf
output "web_ip" {
  description = "Web sunucusunun genel IP adresi"
  value       = aws_instance.web.public_ip
}

output "web_dns" {
  description = "Web sunucusunun DNS adı"
  value       = aws_instance.web.public_dns
}

output "vpc_id" {
  description = "Oluşturulan VPC'nin ID'si"
  value       = aws_vpc.ana.id
}

output "db_endpoint" {
  description = "RDS bağlantı adresi"
  value       = aws_db_instance.postgres.endpoint
  sensitive   = true    # Çıktıda gizle
}
```

```bash
# Çıktıları göster
terraform output
terraform output web_ip
terraform output -json            # JSON formatında

# Başka script'te kullan
WEB_IP=$(terraform output -raw web_ip)
ssh ubuntu@$WEB_IP
```

---

## Slayt 9 — Modüller (Modules)

Kodu yeniden kullanılabilir parçalara böl:

```
proje/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    ├── web-sunucusu/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── veritabani/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

```hcl
# main.tf — Modülü çağır
module "web" {
  source = "./modules/web-sunucusu"

  instance_count = 3
  instance_type  = var.instance_type
  vpc_id         = aws_vpc.ana.id
  subnet_ids     = [aws_subnet.genel.id]
}

module "veritabani" {
  source = "./modules/veritabani"

  db_name    = "uygulama"
  db_user    = "admin"
  db_sifre   = var.db_parolasi
  vpc_id     = aws_vpc.ana.id
}

# Modül çıktısına eriş
output "web_ipleri" {
  value = module.web.public_ips
}
```

---

## Slayt 10 — Döngüler ve Koşullar

```hcl
# count — Birden fazla kaynak oluştur
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0faab6bdbac9486fb"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index + 1}"    # web-1, web-2, web-3
  }
}

# for_each — Map/set ile oluştur
variable "sunucular" {
  default = {
    web = "t3.micro"
    api = "t3.small"
    db  = "t3.medium"
  }
}

resource "aws_instance" "hizmetler" {
  for_each      = var.sunucular
  ami           = "ami-0faab6bdbac9486fb"
  instance_type = each.value

  tags = {
    Name = each.key          # web, api, db
  }
}

# Koşullu kaynak
resource "aws_eip" "web" {
  count    = var.ortam == "production" ? 1 : 0  # Sadece üretimde
  instance = aws_instance.web[0].id
}
```

---

## Slayt 11 — Durum Dosyası (State) Yönetimi

```bash
# Durum dosyasını listele
terraform state list

# Belirli kaynağı göster
terraform state show aws_instance.web

# Kaynağı state'ten çıkar (silmeden)
terraform state rm aws_instance.eski

# Mevcut kaynağı state'e al (import)
terraform import aws_instance.mevcut i-1234567890abcdef0

# Durum dosyasını taşı
terraform state mv aws_instance.web aws_instance.web_sunucu
```

### Uzak Durum Dosyası (Ekip Çalışması)

```hcl
# Terraform Cloud ile
terraform {
  cloud {
    organization = "gulsenmurat-org"
    workspaces {
      name = "uretim"
    }
  }
}

# S3 ile (state lock için DynamoDB)
terraform {
  backend "s3" {
    bucket         = "tfstate-bucket"
    key            = "uretim/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

---

## Slayt 12 — Workspace (Ortam Yönetimi)

```bash
# Mevcut workspace listesi
terraform workspace list

# Yeni workspace oluştur
terraform workspace new staging
terraform workspace new production

# Workspace seç
terraform workspace select staging

# Mevcut workspace
terraform workspace show    # staging
```

```hcl
# Workspace'e göre değişken
locals {
  ortam_ayarlari = {
    staging = {
      instance_type  = "t3.micro"
      instance_count = 1
    }
    production = {
      instance_type  = "t3.medium"
      instance_count = 3
    }
  }

  ayarlar = local.ortam_ayarlari[terraform.workspace]
}

resource "aws_instance" "web" {
  count         = local.ayarlar.instance_count
  instance_type = local.ayarlar.instance_type
  ami           = "ami-0faab6bdbac9486fb"
}
```

---

## Slayt 13 — CI/CD ile Terraform

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Terraform kur
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: 1.6.0

    - name: Terraform init
      run: terraform init
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

    - name: Terraform format kontrol
      run: terraform fmt -check

    - name: Terraform validate
      run: terraform validate

    - name: Terraform plan (PR'da)
      if: github.event_name == 'pull_request'
      run: terraform plan -no-color
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

    - name: Terraform apply (main'e merge sonrası)
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Slayt 14 — İyi Pratikler

```
proje/
├── environments/
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── staging.tfvars
│   └── production/
│       ├── main.tf
│       └── production.tfvars
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
└── README.md
```

### ✅ Yapılacaklar

- **State'i uzakta sakla** (S3/Terraform Cloud) — asla yerel bırakma
- **`.tfvars` dosyalarını git'e commit etme** (secrets içeriyorsa)
- **`terraform plan` çıktısını PR'da göster**
- **Modülleri versiyon etiketiyle pin'le**
- **`sensitive = true`** ile hassas çıktıları gizle
- **`terraform fmt && terraform validate`** CI'da çalıştır

### ❌ Kaçınılacaklar

- State dosyasını elle düzenleme
- `terraform destroy -auto-approve` üretimde çalıştırma
- Credential'ları `.tf` dosyasına yazma

---

## Slayt 15 — Özet

### Öğrendiklerimiz ✅

- IaC kavramı ve Terraform'un rolü
- Provider yapılandırması (AWS örneği)
- Temel komutlar: init, plan, apply, destroy
- Resources ile altyapı kaynakları tanımlama
- Variables ve tfvars dosyaları
- Outputs ile değerleri paylaşma
- Modules ile kodu yeniden kullanma
- count ve for_each ile döngüler
- State yönetimi ve uzak backend
- Workspaces ile çoklu ortam
- GitHub Actions ile CI/CD entegrasyonu

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Terraform Dokümantasyonu | https://developer.hashicorp.com/terraform/docs |
| Terraform Registry (Provider/Module) | https://registry.terraform.io |
| AWS Provider Dokümantasyonu | https://registry.terraform.io/providers/hashicorp/aws |
| Terraform Best Practices | https://www.terraform-best-practices.com |
| Learn Terraform (HashiCorp) | https://developer.hashicorp.com/terraform/tutorials |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
