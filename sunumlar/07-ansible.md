---
title: "Ansible: Yapılandırma Yönetimi"
author: "Gülşen Murat"
date: "2024"
---

# 🔧 Ansible

**Agentsız Yapılandırma Yönetimi ve Otomasyon**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Ansible Nedir?

### Ansible

Red Hat tarafından geliştirilen açık kaynaklı **yapılandırma yönetimi**, **uygulama dağıtımı** ve **görev otomasyonu** aracı.

### Neden Ansible?

```
Geleneksel Yöntem            Ansible ile
──────────────────           ───────────
Her sunucuya SSH gir →       Tek komutla 100 sunucuyu yapılandır
Manuel komut çalıştır →      Tekrarlanabilir, idempotent görevler
Hata yapmaya açık →          YAML ile okunabilir kod
Belgelenmemiş →              Playbook = hem belge hem betik
```

### Temel Özellikler

| Özellik | Açıklama |
|---------|----------|
| **Agentsız** | Hedef sunuculara yazılım kurmaya gerek yok |
| **SSH tabanlı** | Sadece SSH bağlantısı yeterli |
| **İdempotent** | Birden çok çalıştırma aynı sonucu verir |
| **YAML** | Kolay okunabilir yapılandırma dili |
| **5000+ modül** | nginx, docker, apt, yum, user... |

---

## Slayt 2 — Mimari

```
┌────────────────────────────────────────────────────┐
│              Kontrol Düğümü (Sen)                  │
│         ansible komutları buradan çalışır          │
│                                                    │
│  Inventory    Playbook     ansible.cfg             │
│  (sunucu      (görev       (yapılandırma)          │
│   listesi)    listesi)                             │
└────────────────────┬───────────────────────────────┘
                     │ SSH
         ┌───────────┼───────────┐
         ▼           ▼           ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ web01    │ │ web02    │ │ db01     │
   │ (Ubuntu) │ │ (Ubuntu) │ │ (CentOS) │
   └──────────┘ └──────────┘ └──────────┘
   Yönetilen Düğümler (Managed Nodes)
   — ajan kurulmaz, sadece SSH + Python gerekir
```

---

## Slayt 3 — Kurulum

### Kontrol Düğümüne Ansible Kur

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install ansible -y

# CentOS / AlmaLinux
sudo dnf install epel-release -y
sudo dnf install ansible -y

# pip ile (en güncel sürüm)
pip3 install ansible

# Sürümü kontrol et
ansible --version
```

### Hedef Sunuculara SSH Erişimi Ayarla

```bash
# Kontrol düğümünde SSH anahtarı oluştur
ssh-keygen -t ed25519 -C "ansible@kontrol"

# Hedef sunuculara genel anahtarı kopyala
ssh-copy-id kullanici@web01
ssh-copy-id kullanici@web02
ssh-copy-id kullanici@db01

# Test et
ssh kullanici@web01 "echo bağlantı başarılı"
```

---

## Slayt 4 — Inventory (Envanter)

Hangi sunucuları yönetmek istediğini tanımla.

### /etc/ansible/hosts (veya inventory.ini)

```ini
# Tek sunucu
web01.orneksite.com

# Gruplama
[web]
web01.orneksite.com
web02.orneksite.com ansible_port=2222

[db]
db01.orneksite.com

[uretim:children]   # Grupların grubu
web
db

# Değişkenler
[web:vars]
http_port=80
nginx_version=1.25

# Bağlantı seçenekleri
[db:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/db_key
```

### YAML Inventory (Alternatif)

```yaml
# inventory.yml
all:
  children:
    web:
      hosts:
        web01.orneksite.com:
          http_port: 80
        web02.orneksite.com:
    db:
      hosts:
        db01.orneksite.com:
          ansible_user: ubuntu
```

---

## Slayt 5 — Ad Hoc Komutlar

Playbook yazmadan tek komutla çalıştır:

```bash
# Bağlantıyı test et
ansible all -m ping
ansible web -m ping

# Komut çalıştır
ansible web -m command -a "uptime"
ansible all -m shell -a "df -h"

# Paket kur
ansible web -m apt -a "name=nginx state=present" --become

# Servis yönetimi
ansible web -m service -a "name=nginx state=started enabled=yes" --become

# Dosya kopyala
ansible web -m copy -a "src=nginx.conf dest=/etc/nginx/nginx.conf" --become

# Dosya oluştur / sil
ansible web -m file -a "path=/tmp/test.txt state=touch"
ansible web -m file -a "path=/tmp/test.txt state=absent"

# Komut çıktısını göster (-v: verbose)
ansible web -m setup -a "filter=ansible_os_family"
```

---

## Slayt 6 — İlk Playbook

```yaml
# nginx-kur.yml
---
- name: Nginx Kurulumu ve Yapılandırması
  hosts: web                        # Hangi gruba uygula
  become: true                      # sudo kullan

  tasks:
    - name: Paket listesini güncelle
      apt:
        update_cache: yes
        cache_valid_time: 3600      # 1 saat geçerliyse güncelleme

    - name: Nginx kur
      apt:
        name: nginx
        state: present              # present: kur, absent: kaldır, latest: güncelle

    - name: Nginx'i başlat ve etkinleştir
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Güvenlik duvarında 80 portunu aç
      ufw:
        rule: allow
        port: '80'
        proto: tcp
```

```bash
# Çalıştır
ansible-playbook nginx-kur.yml -i inventory.ini

# Dry run (gerçek değişiklik yapma)
ansible-playbook nginx-kur.yml --check

# Verbose mod
ansible-playbook nginx-kur.yml -v
```

---

## Slayt 7 — Değişkenler (Variables)

```yaml
---
- name: Web Sunucusu Kurulumu
  hosts: web
  become: true

  vars:                             # Playbook içinde değişken
    http_port: 80
    max_keepalive: 15
    uygulama_dizini: /var/www/uygulama

  vars_files:
    - vars/uretim.yml               # Harici dosyadan yükle

  tasks:
    - name: Nginx yapılandırması
      template:
        src: nginx.conf.j2          # Jinja2 şablonu
        dest: /etc/nginx/nginx.conf
      notify: nginx yeniden başlat  # Handler tetikle

  handlers:
    - name: nginx yeniden başlat
      service:
        name: nginx
        state: reloaded
```

```yaml
# vars/uretim.yml
http_port: 443
ssl_enabled: true
domain: orneksite.com
```

---

## Slayt 8 — Jinja2 Şablonları (Templates)

```
# templates/nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ domain }};

    root {{ uygulama_dizini }};
    index index.html;

    {% if ssl_enabled %}
    ssl_certificate /etc/ssl/{{ domain }}.crt;
    ssl_certificate_key /etc/ssl/{{ domain }}.key;
    {% endif %}

    # Nginx sürümü gizle
    server_tokens off;

    # Maksimum keepalive
    keepalive_timeout {{ max_keepalive }};
}
```

```yaml
# Playbook'ta kullan
- name: Nginx yapılandırması oluştur
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ domain }}
    owner: root
    group: root
    mode: '0644'
  notify: nginx yeniden yükle
```

---

## Slayt 9 — Döngüler ve Koşullar

### Döngüler (Loops)

```yaml
tasks:
  # Paket listesi kur
  - name: Gerekli paketleri kur
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - git
      - curl
      - python3

  # Kullanıcı listesi oluştur
  - name: Kullanıcılar oluştur
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      state: present
    loop:
      - { name: 'ali', groups: 'sudo' }
      - { name: 'ayse', groups: 'developers' }
```

### Koşullar (When)

```yaml
tasks:
  - name: Ubuntu'da nginx kur
    apt:
      name: nginx
      state: present
    when: ansible_os_family == "Debian"

  - name: CentOS'ta nginx kur
    dnf:
      name: nginx
      state: present
    when: ansible_os_family == "RedHat"

  - name: Sadece üretimde çalıştır
    debug:
      msg: "Üretim sunucusu!"
    when: inventory_hostname in groups['uretim']
```

---

## Slayt 10 — Roller (Roles)

Ansible kodunu modüler organize et.

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml          ← Görevler
    ├── handlers/
    │   └── main.yml          ← Handler'lar
    ├── templates/
    │   └── nginx.conf.j2     ← Jinja2 şablonlar
    ├── files/
    │   └── index.html        ← Statik dosyalar
    ├── vars/
    │   └── main.yml          ← Değişkenler
    ├── defaults/
    │   └── main.yml          ← Varsayılan değerler
    └── meta/
        └── main.yml          ← Rol meta bilgisi
```

```bash
# Boş rol iskelet oluştur
ansible-galaxy role init nginx
ansible-galaxy role init docker
ansible-galaxy role init postgresql
```

```yaml
# site.yml — Rolleri kullan
---
- name: Web Sunucuları
  hosts: web
  roles:
    - nginx
    - { role: ssl, when: ssl_enabled }
```

---

## Slayt 11 — Ansible Galaxy (Hazır Roller)

```bash
# Rol ara
ansible-galaxy role search nginx
ansible-galaxy role search docker

# Rol kur
ansible-galaxy role install geerlingguy.nginx
ansible-galaxy role install geerlingguy.docker
ansible-galaxy role install geerlingguy.postgresql

# Kurulu rolleri listele
ansible-galaxy role list

# requirements.yml ile toplu kur
```

```yaml
# requirements.yml
roles:
  - name: geerlingguy.nginx
    version: "3.2.0"
  - name: geerlingguy.docker
    version: "6.1.0"

collections:
  - name: community.docker
  - name: community.postgresql
```

```bash
ansible-galaxy install -r requirements.yml
```

---

## Slayt 12 — Vault (Gizli Bilgi Şifreleme)

```bash
# Şifreli dosya oluştur
ansible-vault create vars/gizli.yml

# Mevcut dosyayı şifrele
ansible-vault encrypt vars/gizli.yml

# Dosyayı şifreli düzenle
ansible-vault edit vars/gizli.yml

# Şifrele çöz
ansible-vault decrypt vars/gizli.yml

# Şifresini değiştir
ansible-vault rekey vars/gizli.yml

# Şifreli değişken oluştur
ansible-vault encrypt_string 'gizli_sifre' --name 'db_password'
```

```yaml
# vars/gizli.yml (şifreli)
$ANSIBLE_VAULT;1.1;AES256
61636...

# Playbook'ta kullan
- name: Gizli değişkenleri yükle
  include_vars: vars/gizli.yml
```

```bash
# Şifre ile çalıştır
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

---

## Slayt 13 — Tam Web Sunucusu Örneği

```yaml
# site.yml
---
- name: Temel sunucu yapılandırması
  hosts: all
  become: true
  tasks:
    - name: Sistemi güncelle
      apt:
        upgrade: dist
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Gerekli araçları kur
      apt:
        name: [ curl, vim, htop, git, ufw ]
        state: present

    - name: Güvenlik duvarını yapılandır
      ufw:
        rule: "{{ item.rule }}"
        port: "{{ item.port }}"
        proto: tcp
      loop:
        - { rule: allow, port: '22' }
        - { rule: allow, port: '80' }
        - { rule: allow, port: '443' }

    - name: Güvenlik duvarını etkinleştir
      ufw:
        state: enabled

- name: Web sunucusu kurulumu
  hosts: web
  become: true
  roles:
    - nginx

- name: Veritabanı sunucusu
  hosts: db
  become: true
  vars_files:
    - vars/db-gizli.yml
  roles:
    - postgresql
```

---

## Slayt 14 — Özet

### Öğrendiklerimiz ✅

- Ansible'ın agentsız çalışma mantığı
- Inventory ile sunucu gruplarını yönetme
- Ad hoc komutlarla hızlı işlemler
- Playbook yazımı ve task yapısı
- Değişkenler, vars_files ve şablonlar (Jinja2)
- Döngüler ve koşullar (loop, when)
- Handler ile servis yeniden başlatma
- Roller (Roles) ile modüler yapı
- Ansible Galaxy'den hazır rolleri kullanma
- Vault ile gizli bilgileri şifreleme

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Ansible Dokümantasyonu | https://docs.ansible.com |
| Ansible Galaxy | https://galaxy.ansible.com |
| Jeff Geerling'in rolleri | https://github.com/geerlingguy |
| Ansible Best Practices | https://docs.ansible.com/ansible/latest/tips_tricks/ |
| Playground (bağlantısız test) | https://www.katacoda.com/courses/ansible |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
