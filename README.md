
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



# Custom Greenlight 

## 1. Download source code greenlight V3.1.0.1

Link source code:
https://github.com/bigbluebutton/greenlight/releases/

Link greenlight V3.1.0.1: https://github.com/bigbluebutton/greenlight/releases/tag/release-3.1.0.2

extract file .zip tsb

## 2. Install Ruby Rails dan gems

untuk install Ruby Rails bisa di lihat di artikel ini: 
https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-20-04

disarankan untuk versi ruby dan rails nya terbaru

## 3. Konfigurasi environment

buat file .env di folder root greenlight

```
touch .env
nano .env
```
tambahakan environment tersebut:
```
### SECRET KEY BASE
# Verifies the integrity of all secrets created in the application
# Can be generated by running 'docker run --rm --entrypoint /bin/sh bigbluebutton/greenlight:v3 -c "bundle exec rails secret"'
SECRET_KEY_BASE=

### BIGBLUEBUTTON CREDENTIALS
# Set these if you are running GreenLight on a single BigBlueButton server.
# You can retrieve these by running the following command on your BigBlueButton server:
#   bbb-conf --secret
BIGBLUEBUTTON_ENDPOINT=BBB_ENDPOINT
BIGBLUEBUTTON_SECRET=BBB_SECRET

### POSTGRES DATABASE URL
# Must be in the format postgres://username:password@host:port/dbname
#   E.g. postgres://postgres:password@postgres:5432/greenlight-v3-production
DATABASE_URL=postgres://postgres:password@localhost:5432/greenlight-v3-production

### REDIS CACHE URL
# Must be in the format redis://host:port
#   E.g. redis://redis:6379
REDIS_URL=

### OPTIONAL ENV VARS

### SMTP CONFIGURATION
# Emails are required for the basic features of Greenlight to function.
# Please refer to your SMTP provider to get the values for the variables below
# More information: https://docs.bigbluebutton.org/greenlight/v3/install/#email-setup
#SMTP_SENDER_EMAIL=
#SMTP_SENDER_NAME=
#SMTP_SERVER=
#SMTP_PORT=
#SMTP_DOMAIN=
#SMTP_USERNAME=
#SMTP_PASSWORD=
#SMTP_AUTH=
#SMTP_STARTTLS_AUTO=true
#SMTP_STARTTLS=false
#SMTP_TLS=false
#SMTP_SSL_VERIFY=true

### EXTERNAL AUTHENTICATION METHODS
# More information: https://docs.bigbluebutton.org/greenlight/v3/install/#openid-connect-setup
#OPENID_CONNECT_CLIENT_ID=
#OPENID_CONNECT_CLIENT_SECRET=
#OPENID_CONNECT_ISSUER=
#OPENID_CONNECT_REDIRECT=
#OPENID_CONNECT_UID_FIELD=sub

# To enable hCaptcha on the user sign up and sign in, define these 2 keys
# More information: https://docs.bigbluebutton.org/greenlight/v3/install/#hcaptcha-setup
#HCAPTCHA_SITE_KEY=
#HCAPTCHA_SECRET_KEY=

# Set these if you are using a Simple Storage Service (S3)
# Uncomment S3_ENDPOINT only if you are using a S3 OTHER than Amazon Web Service (AWS) S3.
#S3_ACCESS_KEY_ID=
#S3_SECRET_ACCESS_KEY=
#S3_REGION=
#S3_BUCKET=
#S3_ENDPOINT=
#S3_FORCE_PATH_STYLE=

# Set these environment variables if you are using Google Cloud Storage
#GCS_PROJECT=
#GCS_BUCKET=
#GCS_PROJECT_ID=
#GCS_PRIVATE_KEY_ID=
#GCS_PRIVATE_KEY=
#GCS_CLIENT_EMAIL=
#GCS_CLIENT_ID=
#GCS_CLIENT_CERT=

# Define the default locale language code (i.e. 'en' for English) from the following list:
#  [en, ar, fr, es, fa_IR]
# The DEFAULT_LOCALE setting specifies the default language, overriding the browser language which is always set.
# More information: https://docs.bigbluebutton.org/greenlight/v3/install/#default-locale-setup
#DEFAULT_LOCALE=en

# Set this if you like to deploy Greenlight on a relative root path other than /
# More information: https://docs.bigbluebutton.org/greenlight/v3/install/#relative-url-root-path-subdirectory-setup
#RELATIVE_URL_ROOT=/gl

## Define log level in production.
# [debug|info|warn|error|fatal]
# Default 'warn'.
LOG_LEVEL=info

## Use to send logs to external repository (Optional)
# RAILS_LOG_REMOTE_NAME=xxx.papertrailapp.com
# RAILS_LOG_REMOTE_PORT=99999
# RAILS_LOG_REMOTE_TAG=greenlight-v3

```

Wajib di ubah: 

**BIGBLUEBUTTON_ENDPOINT=BBB_ENDPOINT** 

**BIGBLUEBUTTON_SECRET=BBB_SECRET**

**DATABASE_URL=postgres://postgres:password@localhost:5432/greenlight-v3-production** //ganti password dengan password ubuntu

Untuk cek Endpoint dan Secret bigblubutton, gunakan perintah:

```
bbb-conf --secret
```


## 4. Install greenlight 

setelah setting environment selesai. masuk ke directory greenlight tadi lanjut: 
```
bundle install 
```

## 5. Jalankan greenlight 


```
bundle exec rails s
```

maka akan muncul error ruby saat di browser
lakukan: 

- Error Stylesheet :
```
rails css:install
rails css:install:sass
rails css:install:build
```

- Error Javascript :

```
rails javascript:install
rails javascript:build
```

- Error databases : 
biasanya belum konfigurasi .env database atau ada kesalahan akan menghasilkan error database. jika memang sudah setting .env dan lainnya, coba lakukan ini: 

```
rails data:migrate
db:migrate:up:with_data 
```
agar ketika perubahan custom client react javascript:build  seteleah itu wajib menggunakan 

```
rails assets:precompile
```

## 6. Nodemon server (opsional)

agar ketika client di custom dan tidak perlu mematikan server gunakan Nodemon, untuk settinganya: 

- install nodemon global 
```
yarn add -g nodemon
```

- buat file nodemon.json
```
touch nodemon.json
nano nodemon.json
```

paste konfigurasi nodemon: 

```
{
    "ignore": [
      ".git",
      "node_modules/**/node_modules"
    ],
    "watch": [
      "app/controllers/",
      "app/models/",
      "app/assets/",
      "config/",
      "db/"
    ],
    "ext": "rb yml js css scss"
  }
```

untuk menjalakannya cukup perintah: 

```
nodemon -L --exec './rails.sh'
```


# Custom BBB-HTML5

## 1. Clone Bigbluebutton 

```
git clone https://github.com/bigbluebutton/bigbluebutton
cd bigbluebutton/bigbluebutton-html5/
```

## 2. Install Meteor.js 
```
curl https://install.meteor.com/ | sh
```

Ada satu perubahan yang diperlukan pada settings.yml agar webcam dan berbagi layar berfungsi di klien (dengan asumsi Anda sudah menggunakan HTTPS). Langkah pertama adalah mencari nilai untuk kurento.wsUrlpaket settings.yml.

```
grep "wsUrl" /usr/share/meteor/bundle/programs/server/assets/app/config/settings.yml
```

Selanjutnya edit development settings.yml dan ubah wsUrlsesuai dengan yang diambil sebelumnya.

```
vi private/config/settings.yml
```

Anda sekarang siap menjalankan kode HTML5. Pertama-tama matikan versi paket klien HTML5 sehingga Anda tidak menjalankan dua salinan secara paralel.

```
sudo systemctl stop bbb-html5
```

Instal dependensi npm.

```
meteor npm install
```

jalankan bbb-html5 

```
npm start
```

jika ada error dan bbb-html5 tidak jalan, coba jalankan

```
npm run start:dev-fast-mongo
```

Secara default, klien akan berjalan dalam developmentmode dengan hanya satu instance yang NodeJSmenangani peran "backend" dan "frontend".

```
NODE_ENV=production npm start
```

## Ganti NGINX untuk mengalihkan permintaan ke Meteor develop

saat Anda menjalankan bbb-html5 dari paket (yaitu dalam mode produksi) NGINX harus dapat mengarahkan sesi klien ke instance bbb-html5-frontend dari kumpulan. Namun, dalam mode pengembangan (yaitu menjalankan Meteor melalui npm start), kami hanya memiliki satu proses, bukan kumpulan. Kita perlu mengubah konfigurasi NGINX sehingga sesi klien hanya diarahkan ke port 4100tempat Meteor berjalan.

Anda ingin melakukan perubahan ***/usr/share/bigbluebutton/nginx/bbb-html5.nginx*** untuk menggunakan port 4100 daripada pool.

Default - digunakan untuk mode produksi:

```
location ~ ^/html5client/ {
  # proxy_pass http://127.0.0.1:4100; # use for development
  proxy_pass http://poolhtml5servers; # use for production
  ...
```

Mode pengembangan, hanya port 4100 yang digunakan.

```
location ~ ^/html5client/ {
  proxy_pass http://127.0.0.1:4100; # use for development
  # proxy_pass http://poolhtml5servers; # use for production
  ...
  ```

## 3. Custom Client bbb-html5

untuk custom style dan component di bbb-htmlclient bisa masuk ke directory
/imports/...
/client/... 
silahakn di ulik2 kembali.

## 4. Custom Default Slide Presentation

untuk mengganti default slide presentation saat halaman meet di mulai bisa mengubah atau mengganti file default.pdf yg berada di directory

**/var/www/bigbluebutton-default/assets**

wajib menggunakan root atau izin sudo

## 5. Deploy bbb-html5 

deploy ini akan membuat bundle bbb-html5 kedalam **/usr/share/meteor/bundle**

sehingga saat menjalankan bbb-conf --start maka untuk tampilan bbb-html5 atau tampilan meet akan sesuai dengan hasil yg sudah di develop tanpa harus menghentikan systemctl stop bbb-html5 dan blala seperti yg di atas

untuk cara deploy bisa langsung gunakan perintah: 

```
bash deploy_to_usr_share.sh
```

setelah itu restart bbb-conf

```
bbb-conf --restart
bbb-conf --check
```

jika tidak ada error maka proses install berhasil.