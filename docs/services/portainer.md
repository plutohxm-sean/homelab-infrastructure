# Portainer

## Tujuan Service

Portainer merupakan platform manajemen Docker berbasis web yang digunakan untuk mempermudah pengelolaan container, image, volume, network, dan stack Docker melalui antarmuka GUI.

Pada project homelab ini, Portainer digunakan sebagai pusat monitoring dan manajemen seluruh container yang berjalan di server Debian. Dengan adanya Portainer, proses deployment dan pengelolaan service menjadi lebih mudah dibandingkan menggunakan command line secara terus-menerus.

Fungsi utama Portainer dalam homelab:

* Monitoring container Docker
* Deploy stack Docker Compose
* Manajemen volume dan network
* Monitoring resource container
* Mempermudah administrasi service homelab

---

## Port yang Digunakan

| Port | Fungsi                    |
| ---- | ------------------------- |
| 9443 | Dashboard HTTPS Portainer |
| 8000 | Edge Agent Port           |

---

## Cara Deploy

### 1. Membuat Volume

```bash id="8f7n0x"
sudo docker volume create portainer_data
```

---

### 2. Membuat Folder Project

```bash id="u4g4lm"
mkdir -p /opt/homelab/compose/portainer
```

---

### 3. Membuat File Docker Compose

Lokasi file:

```text id="9k7j2j"
/opt/homelab/compose/portainer/compose.yml
```

Isi konfigurasi:

```yaml id="g6y4qx"
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped

    ports:
      - "9443:9443"

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

---

### 4. Menjalankan Container

```bash id="k4e2yu"
cd /opt/homelab/compose/portainer

sudo docker compose up -d
```

---

### 5. Memastikan Container Berjalan

```bash id="h6m0tv"
sudo docker ps
```

---

## Login Default

Akses dashboard melalui browser:

```text id="7n5a6m"
https://IP-SERVER:9443
```

Contoh:

```text id="2q8m9s"
https://192.168.100.22:9443
```

Pada login pertama:

* User diminta membuat username admin
* User diminta membuat password baru

Portainer tidak memiliki default password permanen demi alasan keamanan.

---

## Troubleshooting

### 1. Container Tidak Berjalan

Cek status container:

```bash id="6r0n1a"
sudo docker ps -a
```

Melihat log container:

```bash id="p1z9hf"
sudo docker logs portainer
```

---

### 2. Permission Denied Docker Socket

Jika muncul error:

```text id="f3d8v1"
permission denied while trying to connect to docker.sock
```

Tambahkan user ke grup docker:

```bash id="8t2x4m"
sudo usermod -aG docker $USER
```

Lalu logout dan login kembali.

---

### 3. Port 9443 Tidak Bisa Diakses

Pastikan:

* Container berjalan
* Firewall tidak memblokir port
* Docker service aktif

Cek service Docker:

```bash id="n1s7q0"
sudo systemctl status docker
```

---

### 4. Dashboard Tidak Bisa Dibuka

Cek apakah port sudah listen:

```bash id="e9w4jk"
sudo ss -tulpn | grep 9443
```

Pastikan browser mengakses:

* HTTPS
* IP server yang benar

---

## Kesimpulan

Portainer merupakan service penting dalam homelab berbasis Docker karena mempermudah pengelolaan container melalui tampilan GUI yang lebih user-friendly. Dengan adanya Portainer, proses deployment, monitoring, dan troubleshooting container menjadi lebih cepat dan efisien dibandingkan hanya menggunakan terminal Linux.
