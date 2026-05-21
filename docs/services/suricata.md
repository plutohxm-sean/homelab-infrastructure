# Suricata IDS - Homelab Security Monitoring

## Overview

Suricata digunakan sebagai Intrusion Detection System (IDS) pada homelab untuk melakukan monitoring traffic jaringan dan mendeteksi aktivitas mencurigakan secara real-time.

Pada implementasi ini, Suricata diintegrasikan dengan Wazuh sehingga seluruh alert dan event keamanan dapat dimonitor langsung melalui dashboard SIEM.

---

# Tujuan Implementasi

Beberapa tujuan dari implementasi Suricata pada homelab ini:

- Belajar konsep Intrusion Detection System (IDS)
- Monitoring traffic jaringan secara real-time
- Mendeteksi aktivitas reconnaissance seperti port scanning
- Integrasi network security monitoring dengan Wazuh SIEM
- Memahami alur log security event menggunakan eve.json
- Simulasi basic threat detection pada lingkungan homelab

---

# Arsitektur Monitoring

```text
Internet
   ↓
Cloudflare Tunnel
   ↓
Nginx Proxy Manager
   ↓
Docker Services

         ↓
     Suricata IDS
         ↓
      eve.json
         ↓
     Wazuh Agent
         ↓
    Wazuh Manager
         ↓
   Wazuh Dashboard
```

---

# Installasi Suricata

Update repository:

```bash
sudo apt update
```

Install Suricata:

```bash
sudo apt install suricata -y
```

Verifikasi instalasi:

```bash
suricata --build-info
```

---

# Konfigurasi Interface Monitoring

Cek interface jaringan:

```bash
ip a
```

Edit konfigurasi:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Cari bagian berikut:

```yaml
af-packet:
```

Lalu sesuaikan interface yang digunakan:

```yaml
af-packet:
  - interface: eth0
```

Catatan:
- Interface dapat berbeda tergantung environment
- Pada VM Debian biasanya menggunakan:
  - `eth0`
  - `ens18`
  - `enp0s3`

---

# Aktivasi Eve JSON Logging

Masih pada file:

```bash
/etc/suricata/suricata.yaml
```

Pastikan konfigurasi berikut aktif:

```yaml
- eve-log:
    enabled: yes
    filetype: regular
    filename: /var/log/suricata/eve.json

    types:
      - alert
      - http
      - dns
      - tls
      - ssh
```

---

# Menjalankan Service Suricata

Enable service:

```bash
sudo systemctl enable suricata
```

Restart service:

```bash
sudo systemctl restart suricata
```

Cek status service:

```bash
sudo systemctl status suricata
```

---

# Verifikasi Log Eve JSON

Cek apakah log berhasil dibuat:

```bash
sudo tail -f /var/log/suricata/eve.json
```

Jika berhasil, akan muncul log format JSON secara realtime.

---

# Integrasi Dengan Wazuh

Edit konfigurasi agent:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Tambahkan konfigurasi berikut sebelum tag:

```xml
</ossec_config>
```

Tambahkan:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Restart Wazuh agent:

```bash
sudo systemctl restart wazuh-agent
```

---

# Verifikasi Pada Dashboard Wazuh

Masuk ke dashboard Wazuh:

```text
Security Events
```

Gunakan filter:

```text
suricata
```

atau:

```text
rule.groups: suricata
```

Jika berhasil, alert Suricata akan muncul pada dashboard SIEM.

---

# Simulasi Threat Detection

## Port Scanning Detection

Dari device lain:

```bash
nmap -A SERVER_IP
```

atau:

```bash
nmap -sV SERVER_IP
```

Expected result:
- Suricata menghasilkan alert
- Alert dikirim ke Wazuh
- Event muncul pada dashboard security monitoring

---

# Konsep Yang Dipelajari

## Intrusion Detection System (IDS)

Suricata digunakan untuk:
- monitoring traffic
- packet inspection
- signature detection
- network threat monitoring

---

## Eve JSON

Format log JSON yang digunakan Suricata untuk:
- SIEM integration
- centralized logging
- threat analysis
- security monitoring

---

## SIEM Workflow

Alur monitoring:

```text
Suricata
  ↓
eve.json
  ↓
Wazuh Agent
  ↓
Wazuh Manager
  ↓
Wazuh Dashboard
```

---

# Threat Detection Yang Dapat Dipelajari

Beberapa aktivitas yang dapat dideteksi:

- Port scanning
- Reconnaissance activity
- Suspicious HTTP requests
- Brute force attempts
- Malware indicators
- Network anomalies

---

# Future Improvement

Pengembangan berikutnya yang direncanakan:

- Custom Suricata rules
- Grafana security visualization
- CrowdSec integration
- Active response automation
- GeoIP monitoring
- Threat intelligence feed integration

---

# Kesimpulan

Implementasi Suricata pada homelab memberikan pengalaman belajar mengenai network security monitoring dan IDS workflow secara langsung.

Dengan integrasi ke Wazuh, seluruh security event dapat dianalisis secara terpusat sehingga membantu memahami konsep SIEM, threat detection, dan observability pada lingkungan infrastruktur modern.
