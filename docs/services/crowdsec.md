# CrowdSec Integration on Homelab Infrastructure

## Overview

Pada tahap ini aku mulai menambahkan layer security tambahan ke homelab menggunakan CrowdSec.  
Tujuan utamanya bukan cuma sekadar install tool keamanan, tapi juga belajar bagaimana konsep collaborative threat intelligence dan behavioral detection bekerja di environment nyata.

CrowdSec digunakan berdampingan dengan:

- Wazuh (SIEM & monitoring)
- Suricata (IDS/IPS)
- Docker services
- Debian 12 server
- Tailscale remote access

---

# Tujuan Implementasi

Beberapa hal yang ingin dipelajari dari implementasi ini:

- Cara kerja log parsing
- Behavioral analysis
- Threat detection berbasis scenario
- Automatic ban decision
- Integrasi IDS (Suricata) dengan CrowdSec
- Integrasi security stack modern

---

# Arsitektur

```text
[ Network Traffic ]
        ↓
    Suricata IDS
        ↓
  eve.json logs
        ↓
     CrowdSec
        ↓
Behavior Analysis
        ↓
Ban / Decision
```

---

# Environment

| Component | Value |
|---|---|
| OS | Debian 12 |
| CrowdSec | v1.7 |
| IDS | Suricata 7 |
| SIEM | Wazuh |
| VPN | Tailscale |
| Reverse Proxy | Nginx Proxy Manager |

---

# Instalasi CrowdSec

## Add Repository

```bash
curl -s https://install.crowdsec.net | sudo sh
```

## Install CrowdSec

```bash
sudo apt install crowdsec crowdsec-firewall-bouncer-iptables -y
```

---

# Problem yang Ditemukan

Saat pertama kali menjalankan CrowdSec muncul error:

```text
bind: address already in use
```

Penyebabnya karena port `8080` sudah digunakan container Docker lain.

Cek menggunakan:

```bash
sudo ss -tulpn | grep 8080
```

Output menunjukkan port digunakan oleh docker-proxy.

---

# Solusi

Port Local API CrowdSec dipindahkan dari:

```yaml
127.0.0.1:8080
```

menjadi:

```yaml
127.0.0.1:8081
```

File konfigurasi:

```bash
sudo nano /etc/crowdsec/config.yaml
```

Bagian yang diubah:

```yaml
api:
  server:
    listen_uri: 127.0.0.1:8081
```

---

# Restart Service

```bash
sudo systemctl restart crowdsec
```

Cek status:

```bash
sudo systemctl status crowdsec
```

---

# Integrasi dengan Suricata

CrowdSec dikonfigurasi membaca log Suricata:

```yaml
filenames:
  - /var/log/suricata/eve.json
labels:
  type: suricata
```

Collection Suricata:

```bash
sudo cscli collections install crowdsecurity/suricata
```

---

# Verifikasi Metrics

```bash
sudo cscli metrics
```

Hasil menunjukkan:

- Suricata logs berhasil diparse
- Parser aktif
- Event masuk ke CrowdSec
- Firewall bouncer aktif

---

# Firewall Bouncer

Bouncer digunakan untuk melakukan block otomatis terhadap IP berbahaya.

Installed:

```bash
crowdsec-firewall-bouncer-iptables
```

Verifikasi:

```bash
sudo cscli bouncers list
```

---

# Monitoring & Testing

Beberapa pengujian yang dilakukan:

## Nmap Scan

Melakukan scanning dari device lain:

```bash
nmap -sV 192.168.100.22
```

## Monitoring Logs

```bash
sudo journalctl -u crowdsec -f
```

## Melihat Alerts

```bash
sudo cscli alerts list
```

## Melihat Decisions

```bash
sudo cscli decisions list
```

---

# Integrasi dengan Wazuh

Stack keamanan pada homelab sekarang terdiri dari:

| Tool | Function |
|---|---|
| Wazuh | SIEM & Monitoring |
| Suricata | IDS |
| CrowdSec | Behavioral Security |
| Firewall Bouncer | Automatic Blocking |

Konsep ini dibuat supaya bisa belajar alur modern security monitoring secara end-to-end.

---

# Lessons Learned

Beberapa hal penting yang dipelajari:

- Port conflict pada environment Docker sangat umum terjadi
- CrowdSec bekerja berdasarkan behavior/scenario, bukan sekadar signature
- Integrasi IDS + SIEM + Threat Intelligence cukup powerful untuk homelab
- Parsing log adalah bagian paling penting dalam detection pipeline
- Tidak semua alert otomatis menghasilkan ban

---

# Next Plan

Rencana pengembangan berikutnya:

- Integrasi CrowdSec dengan Nginx Proxy Manager
- Monitoring SSH brute-force
- Integrasi Grafana dashboard
- Alerting ke Discord / Telegram
- Testing attack simulation
- Threat hunting sederhana

---

# Notes

Dokumentasi ini sengaja dibuat dengan pendekatan learning-oriented supaya proses implementasi dan troubleshooting tetap terdokumentasi dengan jelas.

Semua credential, token, dan data sensitif sudah disamarkan atau tidak dicantumkan demi keamanan.
