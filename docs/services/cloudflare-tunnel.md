# Cloudflare Tunnel

## Tujuan Service

Cloudflare Tunnel merupakan layanan secure reverse proxy dari Cloudflare yang digunakan untuk menghubungkan service lokal homelab ke internet tanpa perlu membuka port publik pada router atau menggunakan public IP address.

Pada project homelab ini, Cloudflare Tunnel digunakan untuk mengakses berbagai service internal seperti Grafana, Prometheus, Portainer, Homepage, dan Uptime Kuma secara aman melalui domain `hxms.web.id`.

Dengan menggunakan Cloudflare Tunnel, seluruh service dapat diakses melalui HTTPS tanpa melakukan port forwarding pada router.

Fungsi utama Cloudflare Tunnel dalam homelab:

* Expose service lokal ke internet
* Mengamankan akses service internal
* Mengurangi risiko expose public IP
* Mendukung HTTPS otomatis dari Cloudflare
* Integrasi dengan domain custom
* Mendukung remote access homelab

---

# Arsitektur Tunnel

```text id="a4b0h7"
Internet
   │
   ▼
Cloudflare Edge
   │
   ▼
Cloudflare Tunnel
   │
   ▼
Server Debian Homelab
   │
   ├── Grafana
   ├── Prometheus
   ├── Portainer
   ├── Homepage
   └── Uptime Kuma
```

---

# Service yang Diexpose

| Service     | Domain                 | Local Port |
| ----------- | ---------------------- | ---------- |
| Grafana     | grafana.hxms.web.id    | 3100       |
| Prometheus  | prometheus.hxms.web.id | 9090       |
| Uptime Kuma | uptime.hxms.web.id     | 3001       |
| Portainer   | portainer.hxms.web.id  | 9443       |
| Homepage    | home.hxms.web.id       | 3000       |

---

# Instalasi Cloudflared

## 1. Download Package

```bash id="j6s8tb"
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```

---

## 2. Install Package

```bash id="n3v0dz"
sudo dpkg -i cloudflared-linux-amd64.deb
```

---

## 3. Verifikasi Instalasi

```bash id="w8j5oc"
cloudflared --version
```

---

# Login Cloudflare

Jalankan login:

```bash id="u0k3cf"
cloudflared tunnel login
```

Perintah ini akan membuka browser untuk autentikasi akun Cloudflare.

Setelah berhasil login, file sertifikat akan tersimpan pada:

```text id="c7h9n2"
~/.cloudflared/cert.pem
```

---

# Membuat Tunnel

## 1. Membuat Tunnel Baru

```bash id="x2m5rk"
cloudflared tunnel create homelab
```

---

## 2. Melihat Daftar Tunnel

```bash id="f5p2yh"
cloudflared tunnel list
```

---

# Konfigurasi Tunnel

## Lokasi File Konfigurasi

```text id="q3k9du"
/etc/cloudflared/config.yml
```

---

## Isi Konfigurasi

```yaml id="k8u1xn"
tunnel: TUNNEL-ID

ingress:
  - hostname: grafana.example.com
    service: http://localhost:3100

  - hostname: prometheus.example.com
    service: http://localhost:9090

  - hostname: uptime.example.com
    service: http://localhost:3001

  - hostname: portainer.example.com
    service: https://localhost:9443
    originRequest:
      noTLSVerify: true

  - hostname: home.example.com
    service: http://localhost:3000

  - service: http_status:404
```

---

# Routing DNS Tunnel

Menambahkan DNS route:

```bash id="n0m4vj"
cloudflared tunnel route dns homelab grafana.example.com

cloudflared tunnel route dns homelab prometheus.example.com

cloudflared tunnel route dns homelab uptime.example.com

cloudflared tunnel route dns homelab portainer.example.com

cloudflared tunnel route dns homelab home.example.com
```

---

# Install Service Systemd

Install Cloudflare Tunnel sebagai service Linux:

```bash id="u6z1fr"
sudo cloudflared service install <TOKEN-TUNNEL>
```

---

# Manajemen Service

## Menjalankan Service

```bash id="j4o9mp"
sudo systemctl start cloudflared
```

---

## Restart Service

```bash id="m5r2xa"
sudo systemctl restart cloudflared
```

---

## Enable Auto Start

```bash id="r2x7te"
sudo systemctl enable cloudflared
```

---

## Melihat Status Service

```bash id="b1v4oq"
systemctl status cloudflared
```

---

## Monitoring Log Tunnel

```bash id="y7n2kl"
journalctl -u cloudflared -f
```

---

# Keamanan

Beberapa hal penting terkait keamanan konfigurasi:

* Jangan menyimpan token tunnel pada repository GitHub
* Jangan upload file credentials JSON
* Jangan upload file `cert.pem`
* Gunakan `.gitignore` untuk file sensitif
* Gunakan HTTPS dari Cloudflare
* Hindari expose port langsung ke internet

Contoh `.gitignore`:

```gitignore id="s9c5vx"
*.json
cert.pem
config.yml
.env
```

---

# Troubleshooting

## 1. Tunnel Tidak Berjalan

Cek status service:

```bash id="n2k7xf"
systemctl status cloudflared
```

Melihat log error:

```bash id="o8w4qy"
journalctl -u cloudflared -f
```

---

## 2. Domain Tidak Bisa Diakses

Pastikan:

* DNS route sudah dibuat
* Tunnel berjalan
* Service lokal aktif
* Domain menggunakan Cloudflare proxy

---

## 3. Bad Gateway

Pastikan service tujuan berjalan:

```bash id="p0e4az"
docker ps
```

Cek port service:

```bash id="g3v9rq"
sudo ss -tulpn
```

---

## 4. Tunnel Credentials Error

Jika muncul error credentials:

```text id="x5f8ju"
tunnel credentials file not found
```

Install ulang service tunnel menggunakan token baru.

---

# Kesimpulan

Cloudflare Tunnel merupakan solusi secure remote access yang sangat cocok digunakan pada project homelab karena memungkinkan service lokal diakses melalui internet tanpa perlu public IP maupun port forwarding router. Dengan integrasi Cloudflare Tunnel, homelab menjadi lebih aman, fleksibel, dan mudah diakses dari mana saja menggunakan domain custom dan HTTPS otomatis.
