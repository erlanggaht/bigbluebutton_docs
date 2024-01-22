
# Install BigBlueButton V2.7 di Ubuntu 2.0

Untuk menginstal bigbluebutton, Anda harus menggunakan Ubuntu versi 2.0
karena versi terbarunya belum mendukung dan harus menunggu update dari BBB.
dan untuk mode pengembang kita akan menggunakan Oracle VirtualBox dan install & konfigurasi menggunakan iso ubuntu 2.0.

setelah install ubuntu selesai. buka terminal:  
## 1. Cek Server Lokal

```
cat /etc/default/locale
```

jika ada, hasilnya: 

**LANG=”en_US.UTF-8″**

Jika tidak maka jalankan perintah ini:

```
sudo apt install -y language-pack-en
sudo update-locale LANG=en_US.UTF-8
```

## 2. Izinkan Port Dibuka

```
sudo ufw allow 443/tcp
sudo ufw allow 80/tcp
sudo ufw allow 22/tcp
sudo ufw allow 16384:32768/udp
```

## 3. Konfigurasi Certificate SSL 

- copy **fullchain.pem**, **privkey.pem**, **cert.pem** dan **chain.pem** yg sudah ada  ke 

```
/etc/letsencrypt/live/vmeet.falahtech.com
```

- buat ssl_dhparam 

```
sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096

```
untuk 4096 bebas.

- masukan code nginx ini untuk mengarahkan ke file kunci di atas di directory **/etc/nginx/sites-available/bigbluebutton**: 
dan buat di baris baru nya

```
sudo nano /etc/nginx/sites-available/bigbluebutton 
```

script nginx:
```
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name vmeet.falahtech.com;

  ssl_certificate /etc/letsencrypt/live/vmeet.falahtech.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/vmeet.falahtech.com/privkey.pem; # managed by Certbot
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 10m;
  ssl_protocols TLSv1.2;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers on;
  # sesuaikan dengan nama filenya
  ssl_dhparam /etc/nginx/ssl/dhparam.pem;

  location / {
    root /var/www/bigbluebutton-default;
    index index.html index.htm;
    expires 1m;
  }

  include /etc/bigbluebutton/nginx/*.nginx;
}
```

setelah itu sukses atau tidak nya check bbb-conf dan nginx.service:

```
bbb-conf --check
bbb-conf --restart
sudo systemctl restart nginx.service 
nginx -t
```


## 4. Install Bigbluebutton v2.7
- Install harus lewat root atau sudo sudo
```
sudo su
```
- download script menggunakan wget 

**Tanpa Greenlight:** 
```
wget  https://ubuntu.bigbluebutton.org/bbb-install-2.7.sh
```
**Dengan Greenlight:**
```
wget -qO- https://raw.githubusercontent.com/bigbluebutton/bbb-install/v2.7.x-release/bbb-install.sh | bash -s -- -w -v focal-270 -s vmeet.falahtech.com -e erlanggahidayat.md@gmail.com -g -d
```
Ganti vmeet.falahtech.com dengan domain yang ingin Anda gunakan untuk mengakses BigBlueButton. Dan erlanggahidayat.md@gmail.com dengan alamat email yang ingin digunakan untuk menerbitkan sertifikat SSL Let's Encrypt. Selain itu, pastikan domain yang Anda rencanakan untuk menggunakan catatan A -nya mengarah ke alamat IP Server tempat Anda berencana menginstal BBB.

## 5. konfigurasi greenlight nginx

```
sudo nano /usr/share/bigbluebutton/nginx
```
edit dibagian:

```
location @bbb-fe {
    proxy_pass http://127.0.0.1:5050;
    proxy_redirect off;
    proxy_http_version 1.1;

    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto "https";
    proxy_set_header Connection "";
}
```

menjadi: 
```
location /gl {
    proxy_pass http://127.0.0.1:5050;
    proxy_redirect off;
    proxy_http_version 1.1;

    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto "https";
    proxy_set_header Connection "";
}
```

## 6. Periksa Status

```
sudo bbb-conf --status
```

## 7. Konfigursai etc hosts di windows

agar bbb bisa di akses windows 

- beri izin port 3000, 5050

```
sudo ufw allow 5050
```

- edit file hosts di directory C:\Windows\System32\drivers\etc. edit gunakan administator
dan tambahkan ip virtualbox, misal 

```
192.168.100.233 vmeet.falahtech.com
```

## 8. Kunjungi bigbluebutton 


https://vmeet.falahtech.com/gl




