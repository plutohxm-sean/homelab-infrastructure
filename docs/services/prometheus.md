---
# Prometheus + Node Exporter (Monitoring Stack)

## Tujuan Service

Prometheus merupakan sistem monitoring dan alerting berbasis metrics yang digunakan untuk mengumpulkan dan menganalisis data performa dari server dan service dalam homelab.

Pada project ini, Prometheus digunakan sebagai pusat observability untuk memantau kondisi server Debian dan service Docker yang berjalan.

Selain itu, digunakan juga Node Exporter untuk mengambil data metrics dari sistem seperti CPU, RAM, disk, dan network.

### Fungsi utama dalam homelab:

* Monitoring performa server (CPU, RAM, Disk)
* Monitoring resource Docker host
* Pengumpulan metrics secara real-time
* Dasar sistem observability (Grafana, Loki, Alerting)
* Membantu analisis performa infrastruktur

---

## Port yang Digunakan

| Service       | Port | Fungsi                |
| ------------- | ---- | --------------------- |
| Prometheus    | 9090 | Web UI & Metrics      |
| Node Exporter | 9100 | System metrics export |

---

## Cara Deploy

### 1. Membuat Folder Project

```bash
mkdir -p /opt/homelab/compose/monitoring/prometheus
```

---

### 2. Membuat Data Directory

```bash
mkdir -p /opt/homelab/data/prometheus
```

---

### 3. Membuat Konfigurasi Prometheus

Lokasi file:

```text
/opt/homelab/compose/monitoring/prometheus/prometheus.yml
```

Isi konfigurasi:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```

---

### 4. Membuat Docker Compose

Lokasi file:

```text
/opt/homelab/compose/monitoring/prometheus/compose.yml
```

Isi konfigurasi:

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped

    ports:
      - "9090:9090"

    volumes:
      - /opt/homelab/compose/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - /opt/homelab/data/prometheus:/prometheus

    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped

    ports:
      - "9100:9100"

    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge
```

---

### 5. Menjalankan Container

```bash
cd /opt/homelab/compose/monitoring/prometheus

sudo docker compose up -d
```

---

### 6. Memastikan Container Berjalan

```bash
sudo docker ps
```

---

## Akses Service

Buka browser:

```text
http://IP-SERVER:9090
```

Contoh:

```text
http://192.168.100.22:9090
```

Untuk Node Exporter metrics:

```text
http://IP-SERVER:9100/metrics
```

---

## Cara Cek Monitoring

Di dashboard Prometheus:

* Masuk ke menu **Status → Targets**
* Pastikan semua status **UP**

Jika status DOWN berarti ada masalah koneksi antar container atau konfigurasi target.

---

## Troubleshooting

### 1. Container Tidak Jalan

Cek container:

```bash
sudo docker ps -a
```

Cek log Prometheus:

```bash
sudo docker logs prometheus
```

---

### 2. Node Exporter Tidak Terdeteksi

Cek apakah container berjalan:

```bash
sudo docker ps | grep node-exporter
```

Pastikan target di Prometheus:

```
node-exporter:9100
```

---

### 3. Port Tidak Bisa Diakses

Cek port:

```bash
sudo ss -tulpn | grep 9090
```

Pastikan firewall tidak memblokir port tersebut.

---

### 4. Error Konfigurasi YAML

Jika Prometheus tidak jalan, cek file config:

```bash
sudo nano /opt/homelab/compose/monitoring/prometheus/prometheus.yml
```

Pastikan format YAML tidak salah (indentasi harus benar).

---

## Kesimpulan

Prometheus dan Node Exporter merupakan fondasi utama dalam sistem observability homelab. Dengan adanya service ini, kita bisa memantau kondisi server secara real-time dan memahami performa infrastruktur secara lebih dalam.

Stack ini juga menjadi langkah awal menuju sistem monitoring tingkat lanjut seperti:

* Grafana Dashboard Visualization
* Loki Logging System
* Alerting System (Telegram / Discord)
* Monitoring multi-node dan cloud environment
