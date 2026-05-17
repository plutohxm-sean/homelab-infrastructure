# Nginx Proxy Manager (NPM)

## Tujuan Service

Nginx Proxy Manager digunakan sebagai reverse proxy untuk menghubungkan domain dengan service yang berjalan di dalam server homelab. Service ini mempermudah pengelolaan website atau aplikasi berbasis web melalui tampilan GUI tanpa perlu melakukan konfigurasi Nginx secara manual.

Selain itu, NPM juga mendukung:

* SSL/TLS otomatis menggunakan Let's Encrypt
* Reverse proxy berbasis domain/subdomain
* Access list dan basic security
* Manajemen host dengan lebih mudah

Pada project homelab ini, NPM digunakan sebagai pusat manajemen akses seluruh service seperti:

* Portainer
* Grafana
* n8n
* Dashboard internal
* Service monitoring lainnya

---

## Port yang Digunakan

| Port | Fungsi              |
| ---- | ------------------- |
| 80   | HTTP                |
| 81   | Dashboard Admin NPM |
| 443  | HTTPS               |

---

## Cara Deploy

### 1. Membuat Folder Data

```bash
mkdir -p /opt/homelab/data/npm/{data,letsencrypt}
```

### 2. Membuat Docker Compose

Lokasi file:

```bash
/opt/homelab/compose/npm/compose.yml
```

Isi konfigurasi:

```yaml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped

    ports:
      - "80:80"
      - "81:81"
      - "443:443"

    volumes:
      - /opt/homelab/data/npm/data:/data
      - /opt/homelab/data/npm/letsencrypt:/etc/letsencrypt

    networks:
      - proxy

networks:
  proxy:
    external: true
```

### 3. Menjalankan Container

```bash
cd /opt/homelab/compose/npm
sudo docker compose up -d
```

### 4. Memastikan Container Berjalan

```bash
sudo docker ps
```

---

## Login Default

Akses dashboard:

```text
http://IP-SERVER:81
```

Contoh:

```text
http://192.168.100.22:81
```

Login default:

```text
Email    : admin@example.com
Password : changeme
```

Setelah login pertama, pengguna akan diminta mengganti email dan password admin.

---

## Troubleshooting

### 1. Container Tidak Berjalan

Cek status container:

```bash
sudo docker ps -a
```

Cek log:

```bash
sudo docker logs nginx-proxy-manager
```

---

### 2. Port Sudah Digunakan

Jika muncul error seperti:

```text
port is already allocated
```

Maka kemungkinan port 80, 81, atau 443 sedang digunakan service lain.

Cek port:

```bash
sudo ss -tulpn
```

---

### 3. Tidak Bisa Akses Dashboard

Pastikan:

* Container berjalan
* Firewall tidak memblokir port
* IP server benar
* Service Docker aktif

Cek service Docker:

```bash
sudo systemctl status docker
```

---

### 4. SSL Let's Encrypt Gagal

Penyebab umum:

* Domain belum mengarah ke IP server
* Port 80/443 belum terbuka
* Cloudflare proxy aktif saat request SSL awal

Solusi:

* Pastikan DNS sudah benar
* Gunakan mode DNS Only saat proses awal SSL

---

## Kesimpulan

Nginx Proxy Manager sangat membantu dalam pengelolaan reverse proxy pada lingkungan homelab karena menyediakan antarmuka berbasis web yang mudah digunakan. Dengan adanya NPM, proses konfigurasi domain, SSL, dan routing service menjadi lebih sederhana dibanding konfigurasi manual menggunakan Nginx biasa.

