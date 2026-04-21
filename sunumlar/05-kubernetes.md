---
title: "Kubernetes: Konteyner Orkestrasyonu"
author: "Gülşen Murat"
date: "2024"
---

# ☸️ Kubernetes (K8s)

**Konteyner Orkestrasyonu: Dağıtım, Ölçeklendirme ve Yönetim**

> Hazırlayan: Gülşen Murat

---

## Slayt 1 — Kubernetes Nedir?

### Kubernetes (K8s)

Google tarafından 2014'te açık kaynak olarak yayınlanan **konteyner orkestrasyon** platformu.

```
Docker tek bir konteyneri yönetir
Kubernetes çok sayıda konteyneri otomatik yönetir
```

### Kubernetes Ne Sağlar?

| Özellik | Açıklama |
|---------|----------|
| **Otomatik dağıtım** | Konteynerleri sunuculara dağıtır |
| **Kendini iyileştirme** | Çöken konteyneri yeniden başlatır |
| **Yatay ölçeklendirme** | Yük arttığında yeni kopya açar |
| **Güncelleme yönetimi** | Sıfır kesinti ile güncelleme |
| **Servis keşfi** | Konteynerler birbirini isimle bulur |
| **Yük dengeleme** | Trafiği kopyalar arasında dağıtır |
| **Gizli bilgi yönetimi** | Şifre/API anahtar güvenli saklama |

---

## Slayt 2 — Mimari: Cluster Yapısı

```
┌─────────────────────────────────────────────────────┐
│                  Kubernetes Cluster                  │
│                                                      │
│  ┌─────────────────────┐                            │
│  │   Control Plane      │                            │
│  │  (Master Node)       │                            │
│  │                      │                            │
│  │  ┌────────────────┐ │   ┌──────────┐ ┌────────┐ │
│  │  │  API Server    │ │   │  Node 1  │ │ Node 2 │ │
│  │  │  etcd          │◄├───│  (Worker)│ │(Worker)│ │
│  │  │  Scheduler     │ │   │ Pod Pod  │ │ Pod    │ │
│  │  │  Controller    │ │   └──────────┘ └────────┘ │
│  │  └────────────────┘ │                            │
│  └─────────────────────┘                            │
└─────────────────────────────────────────────────────┘
```

| Bileşen | Görev |
|---------|-------|
| **API Server** | Tüm iletişimin merkezi |
| **etcd** | Cluster durumu veri tabanı |
| **Scheduler** | Pod'ları uygun node'a atar |
| **Controller** | Gerçek durumu → istenen durum |
| **kubelet** | Node üzerinde pod yönetimi |

---

## Slayt 3 — Temel Kavramlar

### Pod

En küçük dağıtım birimi. Bir veya birden fazla konteyner içerir.

```
Pod
├── nginx container (port 80)
└── log-agent container (yan araba / sidecar)
```

### Deployment

Pod kümesini yönetir: kaç kopya, nasıl güncellenir.

### Service

Pod'lara sabit IP/DNS verir. Pod'lar yeniden başlasa da adres değişmez.

### Namespace

Cluster içinde mantıksal izolasyon alanı.

```bash
kubectl get namespaces
# default, kube-system, kube-public, kube-node-lease
```

### ConfigMap / Secret

- **ConfigMap** → Yapılandırma (ortam değişkeni, dosya)
- **Secret** → Şifreli hassas bilgi (base64)

---

## Slayt 4 — Kurulum: kubectl

```bash
# kubectl kur (Linux)
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

kubectl version --client
```

### Yerel Küme: minikube

```bash
# minikube kur
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Küme başlat
minikube start
minikube status

# Dashboard aç
minikube dashboard
```

### Alternatif: kind (Docker içinde K8s)

```bash
# kind kur
go install sigs.k8s.io/kind@latest
# veya
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin/

kind create cluster --name benim-kuumem
```

---

## Slayt 5 — İlk Deployment

### YAML ile Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                   # 3 kopya çalıştır
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

---

## Slayt 6 — Service Türleri

### ClusterIP (Varsayılan — Cluster içi erişim)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### NodePort (Dış erişim — Geliştirme)

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080      # 30000-32767 arası
```

### LoadBalancer (Üretim — Bulut)

```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f service.yaml
kubectl get services
kubectl get svc nginx-service
```

---

## Slayt 7 — Temel kubectl Komutları

```bash
# Kaynakları listele
kubectl get pods                        # Pod listesi
kubectl get pods -o wide               # IP ve node bilgisi dahil
kubectl get pods -n kube-system        # Belirli namespace
kubectl get all                        # Tüm kaynaklar
kubectl get deployments,services,pods  # Birden fazla tür

# Detaylı bilgi
kubectl describe pod nginx-xxxx-yyyy
kubectl describe deployment nginx-deployment

# Log görme
kubectl logs nginx-xxxx-yyyy
kubectl logs -f nginx-xxxx-yyyy        # Canlı log
kubectl logs nginx-xxxx-yyyy -c nginx  # Belirli konteyner

# Pod içine gir
kubectl exec -it nginx-xxxx-yyyy -- bash

# Port yönlendirme (test için)
kubectl port-forward pod/nginx-xxxx-yyyy 8080:80
kubectl port-forward service/nginx-service 8080:80

# Sil
kubectl delete pod nginx-xxxx-yyyy
kubectl delete -f deployment.yaml
```

---

## Slayt 8 — Güncelleme ve Geri Alma

### Rolling Update (Kesintisiz Güncelleme)

```bash
# Image güncelle
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# Güncelleme durumunu izle
kubectl rollout status deployment/nginx-deployment

# Geçmiş
kubectl rollout history deployment/nginx-deployment
```

### Deployment Stratejisi (YAML)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Aynı anda fazladan kaç pod
      maxUnavailable: 0    # Aynı anda kaç pod devre dışı olabilir
```

### Geri Alma (Rollback)

```bash
# Bir önceki sürüme geri dön
kubectl rollout undo deployment/nginx-deployment

# Belirli sürüme geri dön
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

## Slayt 9 — Ölçeklendirme

```bash
# Manuel ölçeklendirme
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale deployment nginx-deployment --replicas=1

# Otomatik ölçeklendirme (HPA — Horizontal Pod Autoscaler)
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=10

kubectl get hpa
```

### HPA YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## Slayt 10 — ConfigMap ve Secret

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: uygulama-konfig
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  DB_HOST: "postgres-service"
```

### Secret

```bash
# Base64 kodla
echo -n "güvenli_sifre" | base64   # Z8O8dmVubGlfc2lmcmU=
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: Z8O8dmVubGlfc2lmcmU=
```

### Deployment'ta Kullan

```yaml
spec:
  containers:
  - name: uygulama
    image: benim-uygulama:1.0
    envFrom:
    - configMapRef:
        name: uygulama-konfig
    - secretRef:
        name: db-secret
```

---

## Slayt 11 — PersistentVolume ve PersistentVolumeClaim

```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

```yaml
# Deployment'ta kullan
spec:
  containers:
  - name: postgres
    image: postgres:15
    volumeMounts:
    - name: postgres-depolama
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: postgres-depolama
    persistentVolumeClaim:
      claimName: postgres-pvc
```

```bash
kubectl get pvc
kubectl get pv
```

---

## Slayt 12 — Ingress (HTTP Yönlendirme)

Tek IP üzerinden birden fazla servise yönlendirme:

```
orneksite.com/api  → api-service:3000
orneksite.com/     → frontend-service:80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: orneksite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: orneksite.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 3000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

```bash
# Nginx Ingress Controller kur
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

---

## Slayt 13 — Namespace ile İzolasyon

```bash
# Namespace oluştur
kubectl create namespace gelistirme
kubectl create namespace uretim

# Belirli namespace'te çalış
kubectl apply -f deployment.yaml -n gelistirme
kubectl get pods -n gelistirme
kubectl get pods -n uretim

# Tüm namespace'lerde
kubectl get pods --all-namespaces

# Varsayılan namespace ayarla
kubectl config set-context --current --namespace=gelistirme
```

```yaml
# YAML'da namespace belirt
metadata:
  name: nginx-deployment
  namespace: gelistirme
```

---

## Slayt 14 — Helm: Kubernetes Paket Yöneticisi

```bash
# Helm kur
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Repo ekle
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Chart ara
helm search repo nginx
helm search repo postgresql

# Kur
helm install benim-nginx bitnami/nginx
helm install benim-postgres bitnami/postgresql \
  --set auth.postgresPassword=sifre123 \
  --set primary.persistence.size=10Gi

# Yönetim
helm list
helm upgrade benim-nginx bitnami/nginx --set replicaCount=3
helm uninstall benim-nginx

# Kendi chart'ını oluştur
helm create benim-uygulama
```

---

## Slayt 15 — Özet

### Öğrendiklerimiz ✅

- Kubernetes'in rolü ve Docker'dan farkı
- Master/Worker mimari yapısı
- Pod, Deployment, Service, Namespace kavramları
- kubectl ile temel yönetim komutları
- Rolling Update ve Rollback
- Manuel ve otomatik ölçeklendirme (HPA)
- ConfigMap ve Secret yönetimi
- PersistentVolume ile veri kalıcılığı
- Ingress ile HTTP yönlendirme
- Helm ile paket yönetimi

---

## 📚 Kaynaklar

| Kaynak | Bağlantı |
|--------|---------|
| Resmi Kubernetes Dokümantasyonu | https://kubernetes.io/docs |
| kubectl Kopya Kağıdı | https://kubernetes.io/docs/reference/kubectl/cheatsheet/ |
| Play with Kubernetes | https://labs.play-with-k8s.com |
| Helm Hub | https://artifacthub.io |
| Kubernetes The Hard Way | https://github.com/kelseyhightower/kubernetes-the-hard-way |

---

*Bu sunum Gülşen Murat tarafından hazırlanmıştır. © 2024*
