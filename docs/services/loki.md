# Loki

## Tujuan Service

Loki merupakan log aggregation system yang dikembangkan oleh Grafana Labs dan digunakan untuk mengumpulkan, menyimpan, serta melakukan pencarian log dari berbagai service dan container.

Pada project homelab ini, Loki digunakan sebagai centralized logging server untuk seluruh container Docker yang berjalan pada server Debian. Loki bekerja bersama Promtail sebagai log shipper dan Grafana sebagai visualisasi log monitoring.

Dengan adanya Loki, administrator dapat memonitor log seluruh service dalam satu dashboard sehingga proses troubleshooting dan monitoring menjadi lebih mudah.

Fungsi utama Loki dalam homelab:

* Centralized logging Docker container
* Monitoring log realtime
* Integrasi dengan Grafana
* Mempermudah troubleshooting service
* Menyimpan histori log container

---

# Arsitektur Loki Stack

```text id="lk1"
Docker Container
       │
       ▼
Promtail ───► Loki ───► Grafana
```

Keterangan:

* Promtail bertugas membaca log container
* Loki menyimpan log
* Grafana menampilkan log melalui dashboard

---

# Port yang Digunakan

| Port | Fungsi        |
| ---- | ------------- |
| 3200 | HTTP API Loki |

---

# Struktur Directory

Lokasi project Loki:

```text id="lk2"
/opt/homelab/compose/monitoring/loki
```

Lokasi data Loki:

```text id="lk3"
/opt/homelab/data/loki
```

---

# Cara Deploy

## 1. Membuat Folder Project

```bash id="lk4"
mkdir -p /opt/homelab/compose/monitoring/loki
```

---

## 2. Membuat Folder Data Loki

```bash id="lk5"
sudo mkdir -p /opt/homelab/data/loki/chunks
sudo mkdir -p /opt/homelab/data/loki/rules
```

---

## 3. Mengubah Permission Folder

```bash id="lk6"
sudo chown -R 10001:10001 /opt/homelab/data/loki
```

Keterangan:

* User ID `10001` digunakan oleh container Loki
* Permission diperlukan agar Loki dapat menulis data log

---

## 4. Membuat File Docker Compose

Lokasi file:

```text id="lk7"
/opt/homelab/compose/monitoring/loki/compose.yml
```

Isi konfigurasi:

```yaml id="lk8"
services:
  loki:
    image: grafana/loki:latest
    container_name: loki

    restart: unless-stopped

    ports:
      - "3200:3100"

    command: -config.file=/etc/loki/local-config.yaml

    volumes:
      - ./config:/etc/loki
      - /opt/homelab/data/loki:/loki

    networks:
      - monitoring

networks:
  monitoring:
    external: true
```

---

## 5. Membuat Folder Config

```bash id="lk9"
mkdir -p /opt/homelab/compose/monitoring/loki/config
```

---

## 6. Membuat File Konfigurasi Loki

Lokasi file:

```text id="lk10"
/opt/homelab/compose/monitoring/loki/config/local-config.yaml
```

Isi konfigurasi:

```yaml id="lk11"
auth_enabled: false

server:
  http_listen_port: 3100

common:
  instance_addr: 127.0.0.1
  path_prefix: /loki

  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules

  replication_factor: 1

  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2024-01-01

      store: tsdb
      object_store: filesystem
      schema: v13

      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```

---

## 7. Menjalankan Container

```bash id="lk12"
cd /opt/homelab/compose/monitoring/loki

docker compose up -d
```

---

## 8. Memastikan Container Berjalan

```bash id="lk13"
docker ps
```

Container yang berhasil berjalan:

```text id="lk14"
loki
```

---

# Verifikasi Loki

Cek API Loki melalui browser:

```text id="lk15"
http://IP-SERVER:3200/ready
```

Contoh:

```text id="lk16"
http://192.168.100.22:3200/ready
```

Jika berhasil akan muncul:

```text id="lk17"
ready
```

---

# Integrasi Grafana

## Menambahkan Data Source Loki

1. Login ke Grafana
2. Buka menu:

```text id="lk18"
Connections → Data Sources
```

3. Klik:

```text id="lk19"
Add data source
```

4. Pilih:

```text id="lk20"
Loki
```

5. Isi URL:

```text id="lk21"
http://loki:3100
```

6. Klik:

```text id="lk22"
Save & Test
```

Jika berhasil akan muncul:

```text id="lk23"
Data source connected successfully
```

---

# Testing Log

Buka menu:

```text id="lk24"
Explore
```

Pilih datasource:

```text id="lk25"
Loki
```

Contoh query:

```logql id="lk26"
{container=~".+"}
```

Query tersebut akan menampilkan seluruh log container Docker.

---

# Troubleshooting

## 1. Container Restart Terus

Cek log container:

```bash id="lk27"
docker logs loki --tail 50
```

---

## 2. Permission Denied

Jika muncul error:

```text id="lk28"
mkdir /loki/rules: permission denied
```

Jalankan:

```bash id="lk29"
sudo chown -R 10001:10001 /opt/homelab/data/loki
```

---

## 3. Loki Tidak Bisa Connect ke Grafana

Pastikan:

* Loki dan Grafana berada dalam network Docker yang sama
* URL datasource benar

Cek network:

```bash id="lk30"
docker network ls
```

---

## 4. Data Source Error

Jika muncul:

```text id="lk31"
no such host
```

Pastikan service menggunakan network yang sama:

```yaml id="lk32"
networks:
  - monitoring
```

---

## 5. Tidak Ada Log Muncul

Pastikan:

* Promtail berjalan
* Loki berjalan
* Query LogQL benar

Cek container:

```bash id="lk33"
docker ps
```

Cek log promtail:

```bash id="lk34"
docker logs promtail
```

---

# Kesimpulan

Loki merupakan service penting dalam observability stack homelab karena memungkinkan administrator melakukan centralized logging terhadap seluruh container Docker secara realtime. Dengan integrasi Loki, Promtail, dan Grafana, proses monitoring dan troubleshooting service menjadi lebih efisien karena seluruh log dapat dipantau dalam satu dashboard terpusat.
