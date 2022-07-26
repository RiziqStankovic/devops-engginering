# DOCKER

Docker adalah sistem operasi (atau waktu proses) untuk kontainer. Mesin Docker diinstal pada setiap server tempat Anda ingin menjalankan kontainer dan menyediakan sekumpulan perintah sederhana yang dapat digunakan untuk membuat, memulai, atau menghentikan kontainer.

Hal pertama yang perlu dipersiapkan yaitu dengan membuat akun pada [Docker Hub](https://hub.docker.com/)

# TASK

# Docker Installation

[Tutorial install docker](https://docs.docker.com/engine/install/ubuntu/)

Bagian pertama dalam pembuatan Docker saya akan mempersiapkan 2 buah server yang nantinya akan saya install Nginx untuk gateway dan Docker untuk menjalankan (Frontend, Backend, dan Database)

Saya akan gunakan VPS yang di sediakan oleh [IDCloudHost](https://idcloudhost.com/) 

Kemudian saya akan build app catalog nginx

Kemudian saya akan mengoneksikan VPS menggunakan terminal menggunakan perintah

ssh "username"@"ip host"

```
examp = ssh ziq@103.174.115.37
```
Akan secara otomatis terinstall docker  

Saya akan melakukan update dan upgrade terlebih dahulu

```
sudo apt update ; sudo apt upgrade
```

Kemudian saya akan memerikan hak akses root kepada docker supaya nantinya tidak perlu menggunakan sudo


```
sudo usermod -aG docker pino
```

# Gateway installation

app catalog Nginx

# Create Docker Images


# DOCKER IMAGES IN APP

Pada App saya akan menginstall [Mysql](https://hub.docker.com/_/mysql) sebagai database

search in idcloudhost atau vps nya
```
docker pull mysql:latest
```

# Docker Setup Frontend , Backend and databases

FORK FRONTEND :
```
git clone https://github.com/dumbwaysdev/wayshub-frontend
```

FORK BACKEND  : 

```
git clone https://github.com/dumbwaysdev/wayshub-backend
```

Saya akan cloning kedua fork FE(frontend) dan BE(Backend)

Kemudian saya akan membuat volume dengan direktori mysql-database

```
docker run -d --name database -p 3306:3306 -v ~/mysql-database:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=P4ssw0rd -e MYSQL_DATABASE=wayshub mysql:latest
```
Untuk memasuki container mysql gunakan perintah 

```
docker container exec -it database bash
```

Kemudian saya akan melihat daftar databases

Selanjutnya saya akan konfigurasi backend menggunakan docker compose

Install terlebih dahulu docker compose


```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
sudo chmod +x /usr/local/bin/docker-compose
```

Untuk cek versi docker compose gunakan perintah

```
docker compose --version
```

Selanjutnya saya akan masuk ke direktori backend dan melakukan konfigurasi mengoneksikan backend ke database dan melakukan migrasi sequelize 

app@App-alfino:~/wayshub-backend/config$ 

```
nano config.json
```

Setelah mengubah  lihat ss config kemudian save 

Kemudian kembali  ke direktori wayshub-backend dan saya akan membuat dockerfile

NOTE : SAYA GUNAKAN node:dubnium-alpine3.11 (karena sizenya kecil)


```
nano dockerfile
```

Untuk membuat file dockerfile

build dockerfile 


```
docker build -t pino/wayshub-be:1.0 .
```

Kemudian selanjutnya saya akan membuat docker-compose.yml 

```
version: '3.8'
services:
 backend:  
   build: .    
   container_name: be
   image: pino/wayshub-be:stable
   stdin_open: true
   ports:
    - 5050:5000
```    

Untuk jalankan docker-compose gunakan perintah

```
docker-compose up -d
```

Bisa di cek container app backend sudah berjalan dan selanjutnya saya akan cek menggunakan web browser

Jika hasilnya seperti ini artinya berhasil 

Kemudian saya akan cek juga migrasi datanya pada mysql

Migrasi data berhasil :D

Selanjutnya saya ingin push repo menuju docker hub

Dikarenakan nama repository saya berbeda tidak sesuai username docker hub jadi saya akan membuat tag baru

```
docker tag 98afd9e2ba69 pinoezz/wayshub-be
```

```
docker push pinoezz/wayshub-be
```

KETERANGAN : Apabila nama repository tidak sesuai dengan akun docker hub maka akan muncul pesan "denied: requested access to the resource is denied"

Selanjutnya cek di web browser 

Apabila berhasil akan muncul repository baru

Sebelum lanjut ke step selanjutnya tentang konfigurasi pada app frontend saya akan membuat DNS pada app backend menggunakan [CloudFlare](cloudflare.com)

Selanjutnya saya akan konfigurasi api.js pada direktori /home/app/wayshub-frontend/src/config yang gunanya untuk menghubungkan FE & BE 

Kemudian buat dockerfile pada frontend


```
nano dockerfile
```

```
FROM node:dubnium-alpine3.11
WORKDIR /usr/app
COPY . .
RUN npm install
EXPOSE 3000
CMD [ "npm", "start" ]
```

Selanjutnya saya langsung membuat docker-compose.yml pada frontend

```
nano docker-compose.yml
```
```
version: '3.8'
services: 
 frontend:
   build: .    
   container_name: fe
   image: pinoezz/wayshub-fe:stable
   stdin_open: true
   ports: 
    - 3030:3000
```

Kemudian save lalu exit

Untuk build serta jalankan docker compose secara daemon menggunakan 

```
docker-compose up -d
```
Kemudian cek image pada docker

Kemudian untuk memastikan cek juga menggunakan web browser

# Setup Gateway (Proxy & SSL)

Beralih ke server gateway saya akan cek terlebih dahulu nginxnya

Kali ini saya gunakan nginx version: nginx/1.18.0 (Ubuntu)

Kemudian saya akan membuat reverse proxy FE dan BE , Saya akan membuat direktori baru bernama wayshub pada /etc/nginx

Untuk domain saya mengggunakan domain dari [CloudFlare](cloudflare.com)


KONFIGURASI REVERSE PROXY FE 

```
server {
        server_name alfino.studentdumbways.my.id;
 
        location /{
        proxy_pass http://103.179.57.222:3030;
        }
}
```

KONFIGURASI REVERSE PROXY BE

```
server {
        server_name api.alfino.studentdumbways.my.id;
 
        location /{
        proxy_pass http://103.179.57.222:5050;
        }
}
```

Kemudian kembali pada direktori/etc/nginx dan menambahkan konfigurasi reverse proxy yang ada pada direktori wayshub

```
include /etc/nginc/wayshub/*;
```

Kemudian save dan exit

Cek konfigurasi menggunakan

```
sudo nginx -t
```
Kemudian restart nginx.service 

```
sudo systemctl restart nginx.service 
```

Cek pada web browser 
Menggunakan SSL CertBot untuk memperaman website

Apa itu SSL ?

SSL adalah singkatan dari Secure Socket Layer, salah satu komponen penting yang harus dimiliki website. Dengan SSL, transfer data di dalam website menjadi lebih aman dan terenkripsi. Bahkan saking pentingnya, Google Chrome melabeli website tanpa sertifikat SSL sebagai Not Secure.

Apabila sistem keamanan ini ditambahkan pada website Anda, maka URL website akan berubah menjadi HTTPS. Tujuan utama pemasangan SSL adalah sebagai pengaman pertukaran data yang terjadi melalui jaringan internet.

Langkah pertama instalasi certbot

```
sudo snap install core; sudo snap refresh core
```

```
sudo snap install --classic certbot
```
Jalankan certbot menggunakan perintah

```
sudo certbot
```
Konfigurasi SSL Certbot otomatis akan ada pada proxy alfino.studentdumbways.my.id

Kemudian saya akan melakukan ssl pada api.alfino.studentdumbways.my.id

Konfigurasi SSL Certbot otomatis akan ada pada proxy api.alfino.studentdumbways.my.id

Kemudian saya akan restart nginx.service

```
sudo systemctl restart nginx.service 
```

Kemudian cek melalui web browser
Saya mencoba melakukan registrasi dan login
Saya akan mencoba mengganti Foto profil channel
suksess