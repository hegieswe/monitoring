# Menjalankan Monitoring Stack

## 1. Jalankan Local Monitoring (Grafana, Prometheus, Loki)
Masuk ke terminal di direktori ini, jalankan:
```bash
docker-compose up -d
```
Akses Grafana di: `http://localhost:3000` (User: `admin` / Pass: `admin`).

> **Penting**: Masuk ke Grafana -> Data Sources, tambahkan **Prometheus** (`http://prometheus:9090`) dan **Loki** (`http://loki:3100`).

## 2. Deploy Grafana Alloy di Kubernetes (K3d)
Setelah monitoring suite jalan di lokal, saatnya deploy agennya di kuberntes menggunakan Helm.

Tambahkan repo Grafana ke sistem helm anda:
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Install Grafana Alloy di namespace `monitoring` menggunakan konfigurasi values.yaml yang telah kita buat:
```bash
kubectl create namespace monitoring
helm install alloy grafana/alloy -n monitoring -f alloy-values.yaml
```

**Selesai!** 
Sekarang Alloy akan menangkap semua logs setiap container dan metric dasar di cluster kubernetes, dan mengirimkannya ke `host.docker.internal` (Komputer lokal Anda tempat Grafana/Prometheus/Loki menyala).
