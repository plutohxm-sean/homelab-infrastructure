# Wazuh SIEM Implementation on Homelab

## Latar Belakang

Sebagai bagian dari pengembangan homelab pribadi yang berfokus pada bidang network engineering, cloud, observability, dan security, saya mulai mengimplementasikan platform SIEM menggunakan Wazuh.

Tujuan utama dari implementasi ini bukan hanya sekadar “menjalankan service”, tetapi juga memahami bagaimana monitoring keamanan bekerja di lingkungan nyata, mulai dari pengumpulan log, monitoring file penting, deteksi aktivitas mencurigakan, hingga visualisasi alert secara terpusat.

Project ini dijalankan pada server Debian 12 menggunakan Docker dan terintegrasi dengan beberapa service lain di homelab seperti:

* Grafana
* Prometheus
* Loki
* Cloudflare Tunnel
* Nginx Proxy Manager
* Docker Monitoring Stack

---

# Tujuan Implementasi

Beberapa tujuan dari implementasi Wazuh di homelab ini:

* Belajar konsep SIEM (Security Information and Event Management)
* Monitoring aktivitas host Linux
* Monitoring perubahan file penting
* Mengumpulkan dan menganalisa log keamanan
* Belajar incident detection dan alerting
* Membuat security monitoring environment sederhana
* Integrasi keamanan dengan infrastructure observability stack

---

# Arsitektur

```text
+----------------------+
| Debian 12 Host       |
|----------------------|
| Docker Engine        |
| Wazuh Manager        |
| Wazuh Dashboard      |
| Wazuh Indexer        |
+----------------------+
           |
           |
           v
+----------------------+
| Wazuh Agent          |
| Debian Host Agent    |
+----------------------+
```

---

# Struktur Directory

```text
/opt/homelab/
├── compose/
│   └── security/
│       └── wazuh/
│           └── single-node/
│
├── docs/
│   └── services/
│       └── wazuh.md
│
└── data/
```

---

# Deployment Wazuh

## Clone Repository Wazuh Docker

```bash
git clone https://github.com/wazuh/wazuh-docker.git
```

Masuk ke directory single-node deployment:

```bash
cd wazuh-docker/single-node
```

---

# Menjalankan Wazuh Stack

Karena port `443` sudah digunakan oleh Nginx Proxy Manager, port dashboard Wazuh diubah menjadi:

```yaml
4443:443
```

Kemudian service dijalankan menggunakan Docker Compose:

```bash
docker compose up -d
```

---

# Verifikasi Container

Cek status container:

```bash
docker ps
```

Container yang berhasil berjalan:

```text
wazuh.manager
wazuh.dashboard
wazuh.indexer
```

---

# Akses Dashboard

Dashboard dapat diakses melalui:

```text
https://IP_SERVER:4443
```

Contoh:

```text
https://192.168.100.22:4443
```

---

# Default Credential

```text
Username: admin
Password: SecretPassword
```

Password default dapat berubah tergantung versi image Wazuh yang digunakan.

---

# Instalasi Wazuh Agent

Agent diinstall langsung pada host Debian.

## Install package agent

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
```

Tambahkan repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list
```

Install agent:

```bash
sudo apt update
sudo apt install wazuh-agent
```

---

# Konfigurasi Agent

Edit file:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Ubah bagian server:

```xml
<client>
  <server>
    <address>192.168.100.22</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

---

# Menjalankan Agent

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Verifikasi status:

```bash
sudo systemctl status wazuh-agent
```

---

# Verifikasi Koneksi Agent

Monitoring log agent:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

Jika berhasil connect ke manager biasanya muncul log seperti:

```text
Connected to manager
```

---

# Fitur yang Dipelajari

Selama implementasi ini, beberapa fitur security yang mulai dipelajari:

## 1. Security Event Monitoring

Monitoring aktivitas login, SSH, sudo, dan event keamanan lainnya.

---

## 2. File Integrity Monitoring (FIM)

Mendeteksi perubahan file penting pada sistem Linux.

Contoh:

* `/etc/passwd`
* `/etc/shadow`
* `/etc/ssh/sshd_config`

---

## 3. Log Analysis

Analisa log dari:

* Syslog
* Auth log
* Docker logs
* System events

---

## 4. Threat Detection

Deteksi aktivitas mencurigakan seperti:

* Brute force SSH
* Login gagal berulang
* Perubahan konfigurasi sistem
* Escalation privilege

---

## 5. Agent Management

Monitoring beberapa endpoint menggunakan Wazuh Agent.

---

# Integrasi dengan Homelab

Wazuh direncanakan akan diintegrasikan dengan:

* Grafana
* Loki
* Prometheus
* CrowdSec
* Cloudflare Tunnel

Tujuannya agar monitoring dan security dapat berjalan dalam satu ecosystem observability stack.

---

# Kendala yang Ditemui

## Konflik Port 443

Karena port 443 sudah digunakan Nginx Proxy Manager, dashboard Wazuh tidak bisa berjalan pada port default.

Solusi:

```yaml
4443:443
```

---

## Dashboard Belum Ready

Saat pertama kali startup, dashboard membutuhkan beberapa menit untuk melakukan initialization.

Pesan berikut masih normal:

```text
Wazuh dashboard server is not ready yet
```

---

## Agent Belum Muncul

Awalnya agent tidak muncul pada dashboard karena konfigurasi IP manager belum sesuai.

Solusi dilakukan dengan mengubah:

```xml
<address>192.168.100.22</address>
```

---

# Pembelajaran yang Didapat

Dari implementasi ini saya mulai memahami:

* Konsep dasar SIEM
* Cara kerja centralized logging
* Monitoring keamanan host Linux
* Security observability
* Cara membaca alert dan event keamanan
* Integrasi security dengan infrastructure monitoring

Selain itu project ini juga membantu memahami bagaimana security monitoring diterapkan pada environment nyata meskipun masih dalam skala homelab.

---

# Next Plan

Beberapa pengembangan berikutnya:

* Integrasi CrowdSec
* Alert ke Discord / Telegram
* Monitoring Docker container
* Integrasi auditd
* Reverse proxy security monitoring
* GeoIP attack visualization
* Integrasi dengan Grafana dashboard

---

# Repository

Homelab repository:

```text
https://github.com/plutohxm-sean/homelab-infrastructure
```

---

# Notes

Dokumentasi ini dibuat sebagai bagian dari proses pembelajaran pribadi di bidang:

* Network Engineering
* Linux System Administration
* Observability
* Cloud Infrastructure
* Cyber Security
* SIEM Implementation
