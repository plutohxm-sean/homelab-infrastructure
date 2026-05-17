# Uptime Kuma

## Tujuan
Digunakan untuk monitoring service homelab secara real-time seperti:
- status online/offline
- latency
- SSL monitoring
- notifikasi alert

## Port
- 3001

## Cara Deploy

```bash
cd /opt/homelab/compose/kuma
sudo docker compose up -d
```

## Akses Dashboard

```text
http://SERVER-IP:3001
```

## Initial Setup
Membuat akun administrator saat pertama kali membuka dashboard.

## Troubleshooting

### Container tidak berjalan

```bash
sudo docker ps
```

### Cek log container

```bash
sudo docker logs uptime-kuma
```

### Port bentrok

```bash
sudo ss -tulpn | grep 3001
```
