# Homepage Dashboard

## Tujuan
Digunakan sebagai dashboard utama homelab untuk mengakses seluruh service dalam satu halaman.

## Port
- 3000

## Cara Deploy

```bash
cd /opt/homelab/compose/homepage
sudo docker compose up -d
```

## Akses Dashboard

```text
http://SERVER-IP:3000
```

## Konfigurasi
Service disimpan di:

```text
/opt/homelab/data/homepage/services.yaml
```

## Troubleshooting

### Dashboard kosong

Restart container:

```bash
sudo docker restart homepage
```

### Cek log

```bash
sudo docker logs homepage
```
