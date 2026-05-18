# Monitoring Stack

> **Stack:** Prometheus + Grafana + Node Exporter + cAdvisor  
> **Status:** ✅ Production  
> **Last Updated:** 2025-05-18

---

## Overview

Observability pipeline untuk homelab berbasis time-series metrics. Stack ini mengumpulkan system metrics dari host dan container, menyimpannya di Prometheus, lalu divisualisasikan di Grafana.

```
Host OS / Docker Containers
        │
        ▼
┌───────────────────┐     ┌─────────────────┐
│   Node Exporter   │────▶│                 │
│   (system metrics)│     │   Prometheus    │
├───────────────────┤     │  (TSDB :9090)   │
│     cAdvisor      │────▶│                 │
│ (container metrics│     └────────┬────────┘
└───────────────────┘              │
                                   ▼
                          ┌─────────────────┐
                          │     Grafana      │
                          │  (dashboard :3100│
                          └─────────────────┘
```

---

## Directory Structure

```
services/monitoring/
├── compose.yml
├── .env                   ← credentials (tidak di-commit)
├── .env.example           ← template
└── config/
    ├── prometheus.yml     ← scrape config
    └── grafana/
        └── provisioning/
            ├── datasources/
            │   └── prometheus.yml   ← auto-connect datasource
            └── dashboards/
                └── dashboards.yml  ← dashboard loader config
```

---

## Services

| Service | Image | Port | Fungsi |
|---|---|---|---|
| prometheus | `prom/prometheus:latest` | `9090` | Time-series database & scraper |
| grafana | `grafana/grafana:latest` | `3100` | Visualization & alerting |
| node-exporter | `prom/node-exporter:latest` | `9100` | Host system metrics |
| cadvisor | `gcr.io/cadvisor/cadvisor:latest` | `8080` | Docker container metrics |

---

## Deployment

### Prerequisites

```bash
# Pastikan networks sudah ada
docker network create proxy 2>/dev/null || true
docker network create monitoring 2>/dev/null || true

# Buat data directories
mkdir -p /opt/homelab/data/{prometheus,grafana}
```

### Setup

```bash
cd /opt/homelab/services/monitoring

# Copy dan edit credentials
cp .env.example .env
nano .env

# Deploy
docker compose up -d

# Verify
docker compose ps
```

### Environment Variables

```env
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=your_secure_password
GRAFANA_ROOT_URL=http://192.168.100.22:3100
```

---

## Configuration

### Prometheus Scrape Config

File: `config/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: "debian-homelab"   # ← custom instance label

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

**Retention:** 15 hari (`--storage.tsdb.retention.time=15d`)

### Reload Config Tanpa Restart

```bash
curl -X POST http://localhost:9090/-/reload
```

### Grafana Datasource Provisioning

Datasource Prometheus di-provision otomatis via file `config/grafana/provisioning/datasources/prometheus.yml` — tidak perlu setup manual di UI saat pertama deploy.

---

## Dashboards

### Node Exporter Full (ID: 1860)

Dashboard utama untuk host monitoring.

**Import:**
1. Grafana → Dashboards → New → Import
2. Masukkan ID: `1860`
3. Load → pilih datasource `Prometheus` → Import

**Panels yang tersedia:**
- Quick CPU / Mem / Disk (gauge)
- CPU Basic (time series, breakdown per mode)
- Memory Basic (Used, Cache, Free, Swap)
- Network Traffic
- Disk Space & I/O
- System Load
- File Descriptors
- Uptime

**Filter variables:**
| Variable | Value |
|---|---|
| `datasource` | Prometheus |
| `job` | node-exporter |
| `instance` | debian-homelab |

---

## Metrics Reference

### Key Queries

**CPU Usage (%)**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**RAM Usage (%)**
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**Disk Usage (%)**
```promql
(1 - (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"})) * 100
```

**Uptime**
```promql
node_time_seconds - node_boot_time_seconds
```

**Container CPU Usage**
```promql
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
```

**Container Memory Usage**
```promql
container_memory_usage_bytes{name!=""} / 1024 / 1024
```

---

## Troubleshooting

### Container conflict saat deploy

```bash
docker rm -f prometheus grafana node-exporter cadvisor
docker compose up -d
```

### Network label mismatch warning

Terjadi jika network `monitoring` dibuat manual sebelumnya. Pastikan di `compose.yml`:

```yaml
networks:
  monitoring:
    external: true
```

### Instance label tidak muncul di Grafana

1. Cek `config/prometheus.yml` — pastikan `relabel_configs` ada
2. Reload config: `curl -X POST http://localhost:9090/-/reload`
3. Tunggu 1 scrape interval (15 detik)

### Grafana datasource duplikat

Terjadi jika sebelumnya ada datasource yang dibuat manual + provisioning baru. Hapus yang manual (lowercase `prometheus`) via UI → Connections → Data sources.

---

## Network Topology

```
Docker Networks:
  monitoring  ← internal, prometheus ↔ exporters
  proxy       ← external, grafana & prometheus expose ke NPM
```

Container yang join `monitoring`: prometheus, node-exporter, cadvisor, grafana  
Container yang join `proxy`: prometheus, grafana

---

## Related Docs

- [Architecture Overview](./architecture.md)
- [Nginx Proxy Manager](./nginx-proxy-manager.md)
- [Alerting Setup](./alerting.md) ← coming soon
