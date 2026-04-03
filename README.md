# 📊 Monitoring Stack & Observability

Selamat datang di direktori konfigurasi monitoring! Direktori ini berisi seluruh komponen yang dibutuhkan untuk memantau kesehatan, *logs*, dan *metrics* dari cluster Kubernetes (K3d) maupun sistem lokal (Docker).

Berikut adalah penjelasan fungsional dari arsitektur monitoring ini yang disusun secara sederhana dan jelas.

---

## 🏗 Arsitektur Komponen

### 1. Grafana (Visualisasi & Dashboard)
> **"Layar TV utama Anda"**

Grafana adalah antarmuka web (UI) tempat Anda memonitor semua data. Grafana sendiri **tidak** menyimpan data apapun. Ia bekerja bagaikan layar proyektor yang hanya menarik (mengambil) data mentah dari "Gudang Data" (seperti Prometheus & Loki), lalu mengubahnya menjadi grafik, diagram (untuk metrics), atau layar konsol (*logs*) yang interaktif, bisa difilter, dan mudah dibaca oleh manusia.

### 2. Prometheus (Gudang Metrics / Angka)
> **"Pencatat Angka & Kesehatan Sistem"**

Prometheus adalah sistem **Database** spesialis *time-series data* (data angka berbasis timeline waktu).
Tugas utamanya adalah menyimpan data utilitas mesin/aplikasi Anda berupa angka-angka.
**Contoh data yang disimpan:** 
- Berapa persen CPU yang terpakai oleh *microservice login*?
- Berapa banyak limit RAM tersisa pada worker node B?
- Berapa kali user membuka halaman ini dalam 1 jam terakhir?

### 3. Grafana Loki (Gudang Logs / Teks)
> **"Pencatat Teks & Konsol Log"**

Sama seperti Prometheus, Loki adalah **Database**. Bedanya, Loki **tidak menyimpan angka**, melainkan menyimpan lautan **Teks/String**. Loki didesain secara spesifik untuk menangkap keluaran standard (*stdout/stderr*) dari terminal aplikasi Anda.
**Contoh data yang disimpan:**
- Pesan `Error: Database connection failed at 18:00`.
- Pesan log aplikasi Go `Info: user hegieswe logged in successfully`.

### 4. Grafana Alloy (Agen Kurir Data)
> **"Sang Pekerja Lapangan di Kubernetes"**

Karena sangat berisiko untuk meletakkan "Gudang" (Prometheus/Loki) di **dalam** Kubernetes (karena jika k8s down, monitoring ikut down dan kita akan buta arah), kita meletakkan Grafana, Prometheus, dan Loki di **Luar / Lokal**. Oleh karena itu, kita membutuhkan perwakilan "Agen Kurir" di dalam Kubernetes.
Itulah fungsi utama **Alloy**! Alloy di-deploy ke dalam node Kubernetes. Ia bekerja tanpa henti menangkap log otomatis dari setiap Pod yang menyala dan mencatat metrik Kubernetes, lalu langsung **mengirimkannya (push/forward)** menyeberang keluar menuju ke Prometheus & Loki lokal Anda (`host.docker.internal`).

### 5. Helm (Package Manager Kubernetes)
> **"App Store untuk Kubernetes"**

Helm mirip seperti perintah `apt-get`, `npm`, atau `brew` di OS biasa, tetapi khusus untuk *environment* Kubernetes.
Mendeploy aplikasi serumit monitoring (Alloy) membutuhkan pembuatan puluhan manifest YAML secara manual. Helm menyederhanakannya! Dengan Helm, kita tinggal menjalankan satu baris perintah `helm install`, lalu helm akan men-download semua pengaturan dasar dari internet, menggabungkannya dengan konfigurasi kustom kita (`alloy-values.yaml`), dan men-deploy semuanya ke Kubernetes secara instan.

---

## 🚀 Panduan Menjalankan Sistem

### A. Menjalankan "Layar & Gudang" di Lokal
Jalankan gudang penyimpanan data dan dashboard Grafana menggunakan Docker Compose (sudah diatur untuk membuat *network* isolasinya sendiri):
```bash
docker-compose up -d
```
1. Akses **Grafana UI**: `http://localhost:3000` (User: `admin` / Password: `admin`).
2. Masuk ke **Connections > Data Sources**.
3. Tambahkan Data source baru:
   - Prometheus dengan URL: `http://prometheus:9090`
   - Loki dengan URL: `http://loki:3100`

### B. Menerjunkan "Kurir" (Alloy) ke dalam Kubernetes dengan Helm
Setelah "gudang" menyala, tugaskan Alloy kita di dalam cluster:

1. Unduh katalog Grafana ke Helm Anda (Hanya dilakukan sekali):
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo update
   ```
2. Buatkan area kerja (Namespace) bernama `monitoring`:
   ```bash
   kubectl create namespace monitoring
   ```
3. Pasang Grafana Alloy menggunakan Helm, dengan membawa konfigurasi log & metric spesifik kita (`alloy-values.yaml`):
   ```bash
   helm install alloy grafana/alloy -n monitoring -f alloy-values.yaml
   ```

**Selesai!** 
Sekarang biarkan sistem berjalan, dan metrics beserta barisan Logs akan terus mengalir live ke Dashboard Grafana lokal Anda. Anda dapat kembali setiap saat jika Kubernetes mengalami kendala.
