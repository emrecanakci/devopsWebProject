# DevOps Web Project - Docker & Kubernetes Dokümantasyonu

Bu dokümanda, projenin Docker ve Kubernetes yapılandırmaları detaylı şekilde açıklanmaktadır.

---

## 📁 Proje Yapısı

```
devopsProject/
├── Dockerfile              # Docker imaj tanımı
├── Jenkinsfile             # CI/CD pipeline tanımı
├── docker-compose.yml      # Jenkins container yapılandırması
├── index.html              # Web sayfası içeriği
├── DOCUMENTATION.md        # Bu dosya
└── k8s/
    ├── deployment.yaml     # Kubernetes Deployment
    └── service.yaml        # Kubernetes Service
```

---

## 🐳 Docker

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

#### Satır Satır Açıklama:

| Satır | Komut | Açıklama |
|-------|-------|----------|
| 1 | `FROM nginx:alpine` | Base imaj olarak nginx:alpine kullanır. Alpine Linux çok hafiftir (~5MB) |
| 2 | `COPY index.html ...` | Yerel `index.html` dosyasını nginx'in varsayılan web dizinine kopyalar |

#### Neden nginx:alpine?
- **Hafiflik**: Alpine tabanlı imajlar çok küçüktür (~7MB vs ~140MB)
- **Güvenlik**: Daha az paket = daha az güvenlik açığı
- **Hız**: Küçük boyut = hızlı indirme ve başlatma

### Docker Komutları

```bash
# İmaj oluşturma
docker build -t myrepo/webapp .

# -t: İmaja tag (isim) verir
# . : Dockerfile'ın bulunduğu dizin (mevcut dizin)
```

```bash
# Container çalıştırma
docker run -d -p 8080:80 myrepo/webapp

# -d: Detached mode (arka planda çalışır)
# -p 8080:80: Host'un 8080 portunu container'ın 80 portuna bağlar
```

```bash
# Minikube için Docker ortamını kullanma
eval $(minikube docker-env)

# Bu komut, Docker client'ı Minikube'ün Docker daemon'ına yönlendirir
# Böylece build edilen imajlar direkt Minikube içinde kullanılabilir
```

---

## ☸️ Kubernetes

### Temel Kavramlar

| Kavram | Açıklama |
|--------|----------|
| **Pod** | En küçük deploy edilebilir birim. Bir veya daha fazla container içerir |
| **Deployment** | Pod'ların nasıl oluşturulacağını ve yönetileceğini tanımlar |
| **Service** | Pod'lara ağ erişimi sağlar, yük dengeleme yapar |
| **ReplicaSet** | Belirli sayıda pod'un çalışmasını garantiler |

---

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: myrepo/webapp:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"
            requests:
              memory: "64Mi"
              cpu: "250m"
```

#### Bölüm Bölüm Açıklama:

##### 1. Metadata
```yaml
apiVersion: apps/v1    # Kubernetes API versiyonu
kind: Deployment       # Kaynak tipi
metadata:
  name: webapp         # Deployment'ın benzersiz adı
  labels:
    app: webapp        # Etiketleme (gruplandırma için)
```

##### 2. Spec (Specification)
```yaml
spec:
  replicas: 2          # Kaç pod çalışacak (yüksek erişilebilirlik için 2+)
  selector:
    matchLabels:
      app: webapp      # Hangi pod'ları yöneteceğini belirler
```

##### 3. Pod Template
```yaml
template:
  metadata:
    labels:
      app: webapp      # Pod'lara verilecek etiket (selector ile eşleşmeli)
  spec:
    containers:
      - name: webapp                    # Container adı
        image: myrepo/webapp:latest     # Kullanılacak Docker imajı
        imagePullPolicy: Never          # Lokalde ara, registry'den çekme
        ports:
          - containerPort: 80           # Container'ın dinlediği port
```

##### 4. Resource Limits
```yaml
resources:
  limits:              # Maksimum kullanılabilir kaynak
    memory: "128Mi"    # 128 Megabyte RAM
    cpu: "500m"        # 0.5 CPU core (500 millicore)
  requests:            # Garantili minimum kaynak
    memory: "64Mi"
    cpu: "250m"
```

#### imagePullPolicy Değerleri:
| Değer | Açıklama |
|-------|----------|
| `Always` | Her zaman registry'den çek |
| `IfNotPresent` | Lokalde yoksa çek |
| `Never` | Sadece lokal imajı kullan (Minikube için ideal) |

---

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30090
```

#### Bölüm Bölüm Açıklama:

```yaml
apiVersion: v1         # Core API (Service için v1 kullanılır)
kind: Service          # Kaynak tipi
metadata:
  name: webapp-service # Service'in benzersiz adı
```

```yaml
spec:
  type: NodePort       # Service tipi (dışarıdan erişim için)
  selector:
    app: webapp        # Hangi pod'lara trafik yönlendireceği
```

```yaml
ports:
  - protocol: TCP      # Protokol (TCP veya UDP)
    port: 80           # Service port (cluster içi erişim)
    targetPort: 80     # Container'ın dinlediği port
    nodePort: 30090    # Dış dünyadan erişim portu (30000-32767 arası)
```

#### Service Tipleri:

| Tip | Açıklama | Kullanım |
|-----|----------|----------|
| `ClusterIP` | Sadece cluster içinden erişim | Varsayılan, internal servisler |
| `NodePort` | Node IP + port ile dış erişim | Development, test ortamları |
| `LoadBalancer` | Cloud provider load balancer | Production (AWS, GCP, Azure) |
| `ExternalName` | DNS CNAME kaydı | Dış servislere yönlendirme |

#### Trafik Akışı:
```
İnternet → NodePort (30090) → Service (80) → Pod (80)
```

---

## 🔧 Kubernetes Komutları

### Temel Komutlar

```bash
# Cluster durumunu kontrol et
kubectl get nodes

# Tüm kaynakları listele
kubectl get all

# Pod'ları listele
kubectl get pods

# Detaylı pod bilgisi
kubectl get pods -o wide

# Service'leri listele
kubectl get svc
```

### Apply ve Delete

```bash
# Manifest dosyasını uygula
kubectl apply -f k8s/

# -f: Dosya veya dizin belirt
# k8s/: Tüm YAML dosyalarını uygula
```

```bash
# Kaynağı sil
kubectl delete -f k8s/

# Belirli pod'u sil
kubectl delete pod <pod-name>

# Label'a göre sil
kubectl delete pods -l app=webapp
```

### Debug Komutları

```bash
# Pod loglarını görüntüle
kubectl logs <pod-name>

# Canlı log takibi
kubectl logs -f <pod-name>

# Pod'a bağlan (shell)
kubectl exec -it <pod-name> -- /bin/sh

# Pod detayları (events dahil)
kubectl describe pod <pod-name>
```

### Scaling

```bash
# Replica sayısını değiştir
kubectl scale deployment webapp --replicas=3

# Otomatik ölçekleme (HPA)
kubectl autoscale deployment webapp --min=2 --max=10 --cpu-percent=80
```

---

## 🚀 Minikube Komutları

```bash
# Minikube başlat
minikube start

# Durumu kontrol et
minikube status

# Durdur
minikube stop

# Dashboard aç
minikube dashboard

# Service URL'ini al
minikube service webapp-service --url

# Service'i tarayıcıda aç
minikube service webapp-service
```

### Docker Ortamı

```bash
# Minikube'ün Docker daemon'ını kullan
eval $(minikube docker-env)

# Normal Docker'a geri dön
eval $(minikube docker-env -u)
```

---

## 🔄 CI/CD Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myrepo/webapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Docker imaji olusturuluyor...'
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                sh 'docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Push') {
            steps {
                echo 'Docker imaji push ediliyor...'
                sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                sh 'docker push ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Kubernetes deploy ediliyor...'
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline basariyla tamamlandi!'
        }
        failure {
            echo 'Pipeline basarisiz oldu!'
        }
    }
}
```

### Pipeline Aşamaları:

| Aşama | İşlem | Komut |
|-------|-------|-------|
| **Build** | Docker imajı oluştur | `docker build -t myrepo/webapp:1 .` |
| **Push** | İmajı registry'e gönder | `docker push myrepo/webapp:1` |
| **Deploy** | Kubernetes'e deploy et | `kubectl apply -f k8s/` |

---

## 📊 Faydalı Kubectl Komutları

### Hızlı Referans

| Komut | Açıklama |
|-------|----------|
| `kubectl get pods -w` | Pod'ları canlı izle |
| `kubectl top pods` | Pod kaynak kullanımı |
| `kubectl rollout status deployment/webapp` | Deployment durumu |
| `kubectl rollout undo deployment/webapp` | Son değişikliği geri al |
| `kubectl port-forward svc/webapp-service 8080:80` | Local port forwarding |

---

## 🔗 Yararlı Linkler

- [Kubernetes Resmi Dokümantasyon](https://kubernetes.io/docs/)
- [Docker Resmi Dokümantasyon](https://docs.docker.com/)
- [Minikube Dokümantasyon](https://minikube.sigs.k8s.io/docs/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)

---

## 📝 Notlar

1. **imagePullPolicy: Never** - Minikube'de lokal imaj kullanmak için gerekli
2. **NodePort aralığı** - 30000-32767 arası olmalı
3. **Resource limits** - Production'da mutlaka tanımlanmalı
4. **Replicas** - Yüksek erişilebilirlik için minimum 2 olmalı

---

*Bu dokümantasyon, DevOps Web Project için oluşturulmuştur.*
*Son güncelleme: 2026-01-11*
