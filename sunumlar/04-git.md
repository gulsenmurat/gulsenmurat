---
title: "Git: Versiyon Kontrol Sistemi"
author: "Gülşen Murat"
date: "2024"
---

# 🌿 Git

**Versiyon Kontrol: Kurulum, Kullanım ve Takım Çalışması**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Git Nedir?

### Git

2005'te **Linus Torvalds** tarafından Linux kernel geliştirmesi için yaratıldı.

```
Versiyon 1.0   Versiyon 1.1   Versiyon 1.2   (şu an)
    │               │               │            │
────●───────────────●───────────────●────────────●──→
    │                                            │
  ilk commit                              son commit
```

### Neden Git Kullanmalıyız?

| Sorun | Git'in Çözümü |
|-------|--------------|
| Dosyaları elle yedeklemek | Her commit otomatik anlık görüntü |
| "Eski haline döndüremiyorum" | İstediğin commit'e geri dön |
| Takımla çakışma | Branch ile paralel geliştirme |
| "Kim bu kodu yazdı?" | `git blame` ile geçmiş |
| Yedek yok, disk bozuldu | Uzak repo (GitHub/GitLab) |

---

## Slayt 2 — Kurulum

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install git -y
git --version    # git version 2.43.0
```

### CentOS / AlmaLinux

```bash
sudo dnf install git -y
git --version
```

### Windows

Git for Windows: https://git-scm.com/download/win  
(Git Bash ile birlikte gelir)

### macOS

```bash
brew install git
# veya
xcode-select --install   # Xcode araçlarıyla birlikte gelir
```

---

## Slayt 3 — İlk Yapılandırma

Git'e kim olduğunu söyle:

```bash
# Kimlik bilgileri (commit'lere eklenir)
git config --global user.name  "Gülşen Murat"
git config --global user.email "gulsenmurat@email.com"

# Varsayılan metin editörü
git config --global core.editor nano     # nano
git config --global core.editor "code --wait"  # VS Code

# Varsayılan dal adı (main)
git config --global init.defaultBranch main

# Yapılandırmayı görüntüle
git config --list
cat ~/.gitconfig
```

---

## Slayt 4 — Git'in Çalışma Alanları

```
Çalışma          Hazırlık           Yerel             Uzak
 Dizini           Alanı              Repo              Repo
(Working)        (Staging)          (Local)           (Remote)
   │                 │                 │                 │
   │──git add──────►│                 │                 │
   │                 │──git commit───►│                 │
   │                 │                │──git push──────►│
   │◄──git checkout──────────────────│                 │
   │                 │                │◄──git fetch─────│
   │◄──git pull──────────────────────────────────────── │
```

| Alan | Açıklama |
|------|----------|
| **Working Directory** | Düzenlediğin dosyalar |
| **Staging Area** | Commit'e hazırlanan değişiklikler (`git add`) |
| **Local Repo** | Kendi bilgisayarındaki commit geçmişi |
| **Remote Repo** | GitHub/GitLab'daki paylaşılan repo |

---

## Slayt 5 — Repo Oluşturma ve İlk Commit

```bash
# Mevcut dizini Git repo'su yap
mkdir benim-projem && cd benim-projem
git init
# → .git/ dizini oluşturulur

# Uzak repoyu klonla
git clone https://github.com/gulsenmurat/proje.git
cd proje

# ── İlk commit akışı ────────────────────────────────────

# 1. Dosya oluştur
echo "# Benim Projem" > README.md

# 2. Durumu kontrol et
git status
# → Untracked files: README.md

# 3. Hazırlık alanına ekle
git add README.md        # Belirli dosya
git add .                # Tüm değişiklikler

# 4. Commit et
git commit -m "İlk commit: README eklendi"

# 5. Uzak repoya gönder
git push origin main
```

---

## Slayt 6 — Temel Komutlar

```bash
# Durum ve geçmiş
git status                  # Değişikliklerin durumu
git log                     # Commit geçmişi
git log --oneline           # Özet geçmiş
git log --oneline --graph   # Dal grafiği
git diff                    # Sahne öncesi değişiklikler
git diff --staged           # Sahnelenen değişiklikler

# Değişiklik alma
git fetch origin            # Uzak değişiklikleri getir (birleştirmez)
git pull origin main        # Getir + birleştir
git pull --rebase origin main  # Getir + rebase

# Gönderme
git push origin main        # main dalına gönder
git push -u origin main     # Takip ayarla (ilk seferlik)
git push --force-with-lease # Güvenli force push
```

---

## Slayt 7 — Dal (Branch) Yönetimi

```bash
# Dalları listele
git branch              # Yerel dallar
git branch -r           # Uzak dallar
git branch -a           # Hepsi

# Dal oluştur ve geç
git branch yeni-ozellik             # Sadece oluştur
git checkout yeni-ozellik           # Geç
git checkout -b yeni-ozellik        # Oluştur + geç (kısayol)
git switch -c yeni-ozellik          # Modern sözdizimi

# Dal değiştir
git checkout main
git switch main

# Dal birleştir
git checkout main
git merge yeni-ozellik              # Fast-forward veya merge commit

# Dal sil
git branch -d yeni-ozellik         # Birleştirilmişse sil
git branch -D yeni-ozellik         # Zorla sil
git push origin --delete yeni-ozellik  # Uzak dalı sil
```

---

## Slayt 8 — Bir Feature'ın Yaşam Döngüsü

```bash
# 1. main'den güncel branch aç
git checkout main
git pull origin main
git checkout -b feature/kullanici-girisi

# 2. Değişiklikleri yap
# ... kod yaz ...
git add src/login.js
git commit -m "feat: kullanıcı giriş formu eklendi"

# ... daha fazla kod yaz ...
git add src/auth.js tests/auth.test.js
git commit -m "feat: JWT doğrulama eklendi"

# 3. main'i takip et (çakışmaları erkenden çöz)
git fetch origin
git rebase origin/main

# 4. Uzağa gönder
git push origin feature/kullanici-girisi

# 5. GitHub/GitLab'da Pull Request aç
# 6. Code review sonrası main'e merge et
# 7. Feature branch'i sil
git branch -d feature/kullanici-girisi
```

---

## Slayt 9 — Geri Alma İşlemleri

```bash
# Henüz commit edilmemiş değişiklikleri geri al
git restore dosya.js            # Çalışma dizinini geri al
git restore --staged dosya.js   # Staging'den geri al
git restore .                   # Tüm dosyaları geri al

# Son commit'i düzenle (henüz push edilmemişse)
git commit --amend -m "Düzeltilmiş commit mesajı"
git commit --amend --no-edit    # Sadece dosya ekle

# Commit geri alma (geçmişi korur — GÜVENLİ)
git revert HEAD                 # Son commit'i geri al
git revert abc1234              # Belirli commit'i geri al

# Commit geri alma (geçmişi değiştirir — PAYLAŞILMAMIŞSA)
git reset --soft HEAD~1         # Son commit'i geri al, değişiklikler staging'de
git reset --mixed HEAD~1        # Son commit'i geri al, değişiklikler working'de
git reset --hard HEAD~1         # Son commit'i tamamen sil ⚠️
```

---

## Slayt 10 — Stash (Geçici Saklama)

Yarım kalan çalışmayı kaydet, başka bir şey üzerinde çalış, geri dön.

```bash
# Değişiklikleri sakla
git stash                       # Sakla (isim otomatik)
git stash push -m "login formu yarım kaldı"  # İsimle sakla

# Stash listesi
git stash list
# stash@{0}: On main: login formu yarım kaldı
# stash@{1}: On feature/api: WIP

# Geri al
git stash pop                   # En son stash'i uygula + listeden sil
git stash apply stash@{1}       # Belirli stash'i uygula (silme)

# Stash sil
git stash drop stash@{0}
git stash clear                 # Hepsini sil

# Stash içeriğini gör
git stash show -p stash@{0}
```

---

## Slayt 11 — .gitignore

```bash
# .gitignore dosyası oluştur
nano .gitignore
```

```gitignore
# Bağımlılıklar
node_modules/
vendor/
.venv/

# Build çıktıları
dist/
build/
*.o
*.class

# Ortam değişkenleri (ASLA commit etme!)
.env
.env.local
.env.*.local
*.pem
*.key

# IDE dosyaları
.idea/
.vscode/
*.swp
*.swo

# İşletim sistemi dosyaları
.DS_Store
Thumbs.db

# Log dosyaları
*.log
logs/
```

```bash
# Zaten takip edilen dosyayı ignore et
git rm --cached .env
git commit -m "chore: .env dosyasını takipten çıkar"
```

---

## Slayt 12 — Çakışma (Conflict) Çözümü

İki kişi aynı satırı değiştirince çakışma olur:

```bash
git merge feature-dal
# CONFLICT (content): Merge conflict in src/app.js
```

### Çakışma İşaretleri

```
<<<<<<< HEAD (senin değişikliğin)
console.log("Merhaba Dünya");
=======
console.log("Hello World");
>>>>>>> feature-dal (diğer kişinin değişikliği)
```

### Çözüm Adımları

```bash
# 1. Çakışan dosyaları düzenle (işaretleri sil, doğru kodu tut)
nano src/app.js

# 2. Düzeltilen dosyayı ekle
git add src/app.js

# 3. Merge'ü tamamla
git commit -m "merge: feature-dal birleştirildi, çakışmalar çözüldü"

# Grafiksel araç kullan (opsiyonel)
git mergetool
```

---

## Slayt 13 — GitHub SSH Bağlantısı

```bash
# 1. SSH anahtar çifti oluştur
ssh-keygen -t ed25519 -C "gulsenmurat@email.com"
# → ~/.ssh/id_ed25519 (özel)
# → ~/.ssh/id_ed25519.pub (genel — GitHub'a eklenecek)

# 2. Genel anahtarı kopyala
cat ~/.ssh/id_ed25519.pub

# 3. GitHub'a ekle:
# Profile → Settings → SSH and GPG keys → New SSH key
# Kopyaladığın metni yapıştır

# 4. Test et
ssh -T git@github.com
# Hi gulsenmurat! You've successfully authenticated...

# 5. Repoyu SSH ile klonla
git clone git@github.com:gulsenmurat/proje.git

# Mevcut HTTPS URL'yi SSH'a çevir
git remote set-url origin git@github.com:gulsenmurat/proje.git
git remote -v
```

---

## Slayt 14 — Commit Mesajı Yazma Kuralları

İyi commit mesajları, projenin geçmişini anlamayı kolaylaştırır.

### Conventional Commits Formatı

```
<tür>(<kapsam>): <özet>

<isteğe bağlı gövde>

<isteğe bağlı alt bilgi>
```

### Tür Örnekleri

| Tür | Kullanım |
|-----|----------|
| `feat` | Yeni özellik |
| `fix` | Hata düzeltmesi |
| `docs` | Dokümantasyon |
| `style` | Biçim (kod mantığı değişmez) |
| `refactor` | Yeniden yapılandırma |
| `test` | Test ekleme/düzenleme |
| `chore` | Araç/yapılandırma değişikliği |

### Örnekler

```bash
git commit -m "feat(auth): JWT ile oturum yönetimi eklendi"
git commit -m "fix(nginx): SSL sertifikası yenileme hatası düzeltildi"
git commit -m "docs: Docker kurulum adımları güncellendi"
git commit -m "refactor(api): veritabanı bağlantısı havuza alındı"
```

---

## Slayt 15 — Faydalı Git Komutları

```bash
# Belirli commit'e git
git checkout abc1234             # Detached HEAD moduna geç
git checkout main                # Geri dön

# Belirli bir dosyayı eski haline getir
git checkout abc1234 -- src/app.js

# Commit'i başka dala uygula
git cherry-pick abc1234

# Geçmişi yeniden yaz (interactive rebase)
git rebase -i HEAD~3             # Son 3 commit'i düzenle
# → pick / squash / reword / drop seçenekleri

# Kim yazdı?
git blame src/app.js             # Her satır için son commit
git log -p src/app.js            # Dosyanın değişim geçmişi

# Bir değişikliğin hangi commit'te geldiğini bul
git bisect start
git bisect bad                   # Şu an hatalı
git bisect good v1.0             # Bu noktada iyiydi
# Git ikili arama yapar...

# Takma adlar (alias) tanımla
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.lg "log --oneline --graph --all"
```

---

## Slayt 16 — Git Workflow Stratejileri

### GitHub Flow (Basit, önerilen)

```
main ●────────────────────────────────────●────►
      \                                  /
       ● feature/x ────────────────────●
         (geliştir → PR aç → review → merge)
```

### Git Flow (Karmaşık projeler)

```
main      ●──────────────────────────────●──────►
           \                            /
develop     ●──●──────────────────●──●──►
               \                 /
feature          ●──────────────●
```

### Trunk-Based Development (CI/CD uyumlu)

```
main  ●──●──●──●──●──●──●──►   (çok sık, küçük commitler)
         (feature flags ile incomplete özellikler saklanır)
```

---

## Slayt 17 — Özet

### Öğrendiklerimiz ✅

- Git'in çalışma alanları: Working, Staging, Local, Remote
- İlk repo kurulumu ve kimlik yapılandırması
- Temel komutlar: add, commit, push, pull, fetch
- Branch ile paralel geliştirme
- Değişiklikleri geri alma: restore, revert, reset
- Stash ile geçici saklama
- .gitignore ile hassas dosyaları dışlama
- Çakışma çözümü
- SSH ile GitHub bağlantısı
- Conventional Commits ile anlamlı mesajlar

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Git Dokümantasyonu | https://git-scm.com/doc |
| Pro Git (Türkçe dahil ücretsiz kitap) | https://git-scm.com/book/tr/v2 |
| GitHub Skills (etkileşimli) | https://skills.github.com |
| Conventional Commits | https://www.conventionalcommits.org/tr |
| Learn Git Branching (görsel) | https://learngitbranching.js.org/?locale=tr_TR |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
