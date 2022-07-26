# Cloud Computing

Cloud Computing merupakan sebuah teknologi yang menjadikan internet sebagai pusat server untuk mengelola data dan juga aplikasi pengguna. Cloud Computing memudahkan penggunanya untuk menjalankan program tanpa harus menginstall aplikasi terlebih dahulu dan memudahkan pengguna untuk mengakses data dan informasi melalui internet

# Cara Kerja Cloud Computing

Teknologi Cloud Computing ini menjadikan internet sebagai pusat server dalam mengelelola data. Sistem ini memudahkan pengguna untuk login ke internet agar mendapatkan akses untuk menjalankan program atau aplikasi tanpa harus menginstall aplikasi tersebut.

# Membuat Server Gateway dan Aplikasi menggunakan IdCloudhost


PT Cloud Hosting Indonesia (IDCloudHost) Merupakan Salah Satu Web Hosting Provider yang Ada di Indonesia dengan Menawarkan Layanan Seperti Pendaftaran Domain, Cloud Hosting, Server (VPS & Dedicated Server), Reseller Domain & Hosting, dan Beberapa Layanan Lainnya.


Langkah pertama dalam pembuatan server kalian terlebih dahulu harus membuat akun IdCloudhost menggunakan link di bawah ini

[IdCloudhost](https://idcloudhost.com/)

Kemudian kalian bisa login / membuat akun baru IdCloudhost terlebih dahulu

Setelah login kalian akan masuk pada menu dashboard

Kemudian pilih menu Compute di kiri pojok dan pilih create new resource

Saya akan gunakan Ubuntu versi 20.04 LTS , Server Indonesia

Kemudian kalian isi username dan password , untuk ssh ini opsional kemudian pilih create di kanan bawah

Tunggu proses building server aplikasi hingga selesai

# Apabila proses sudah selesai selanjutnya saya akan membuat Server Gateway

Untuk type pilih app catalog dan Os nya Nginx

Kemudian isi username password dan resource name , untuk ssh ini opsional kemudian pilih create

Tunggu hingga proses building Gateway selesai

Selanjutnya saya akan login pada kedua server yang sudah saya buat 

Untuk ip kalian bisa cek dengan mengklik resource 

Kalian copy ip public lalu ketikan ssh "username"@"ip" lalu masukan password
```
"Username(app atau gateway)"@"IP"
```
Apabila berhasil masuk ke server tampilan akan menjadi seperti ini

# Melakukan konfigurasi reverse proxy pada Server Gateway

Saya akan login terlebih dahulu pada server gateway

Pertama tama saya akan melakukan update dan upgrade pada server Gateway menggunakan perintah

```
sudo apt update ; sudo apt upgrade
```
Kemudian check versi nginx dengan menggunakan perintah 

```
nginx -v
```

Seharusnya pada server Gateway ini otomatis terinstall nginx

Selanjutnya buat direktori baru pada /etc/nginx , saya akan membuat direktori baru bernama dumbways dengan perintah

```
sudo mkdir dumbways
```

Selanjutnya saya akan membuat file untuk menyimpan konfigurasi dari reverse proxy 

```
sudo nano proxy.conf
```

```
server {
        server_name wayshub.xyz;

        location / {
                proxy_pass http://10.71.15.131:3000;
        }
}
```

Keterangan : server_name adalah nama server , proxy_pass isi dengan ip dari aplikasi 

Kemudian save 

Kemudian masuk ke file nginx.conf untuk menambahkan konfigurasi proxypass pada list INCLUDE

Setelah menambahkan konfigurasi reverse proxy pada nginx.conf kalian bisa save lalu exit

Untuk mengecek konfigurasi file reverse proxy kalian bisa menggunakan perintah 

```
sudo nginx -t
```

# Menginstall Aplikasi Frontend 

Pertama tama switch atau login terlebih dahulu pada server aplikasi atau server frontend terlebih dahulu

Menggunakan perintah adduser untuk menambahkan user

```
sudo adduser aplikasi
```
Untuk memberikan izin sudo pada user baru gunakan perintah 

```
sudo usermod -aG sudo aplikasi
```

Untuk membuat user aplikasi bisa gunakan perintah diatas

Kemudian untuk login / pindah user bisa menggunakan perintah

```
sudo login aplikasi
```

Untuk login ssh atau user yang baru saja di buat bisa menggunakan cara seperti ini

```
ssh aplikasi@ip
```
Kemudian saya update dan upgrade terlebih dahulu


Berikut tadi adalah cara menggunakan useradd saya akan melanjutkan lagi pada instalasi npm , nvm dan cloning fork

Disini saya sudah login pada user aplikasi

Karena disini saya akan mendeploy aplikasi frontend dengan konfigurasi node js jadi terlebih dahulu saya akan menginstall NPM (Node Package Manager)
dan NVM  (Node Version Manager) terlebih dahulu 
```
sudo apt install npm
```


Selanjutnya saya akan Install NVM (Node Version Manager) menggunakan link di bawah ini

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```
Kemudian saya install nvm nya terlebih dahulu

Selanjutnya cloning fork https://github.com/dumbwaysdev/wayshub-frontend menggunakan perintah

```
git clone https://github.com/dumbwaysdev/wayshub-frontend
```

Setelah itu jalankan perintah
```
npm i
```

Setelah sudah saya akan menginstall PM2 yang tujuannya yaitu supaya aplikasi dapat berjalan pada background


```
npm install pm2 -g
```

Dikarenakan pada direktori diatas tidak ada file index.js untuk menjalankannya kita harus terlebih dahulu membuat file ecosystem untuk menjalankan aplikasinya

```
pm2 ecosystem simple
```

Kemudian masuk ke file ecosystem.config.js


```
module.exports = {
  apps : [{
    name   : "frontend-wayshub",
    script : "npm start"
  }]
}
```
Save dan exit

```
pm2 start ecosystem.config.js
```

Kemudian cek di web browser

Apabila berhasil akan muncul aplikasi frontend pada web browser


# Membuat DNS (Domain Name Server)


Singkatnya, DNS adalah sebuah sistem yang mengubah URL website ke dalam bentuk IP Address. Tanpa DNS, Anda harus mengetikkan IP Address secara lengkap ketika ingin mengunjungi sebuah website

Disini saya akan membuat DNS pada aplikasi frontend wayshub menggunakan cloudflare

Daftar atau login terlebih dahulu di website [CloudFlare](https://dash.cloudflare.com/)

Kemudian pilih DNS

Isi nama dan ipv4(ip gateway) kemudian save

Domain yang saya buat : alfino.studentdumbways.my.id

Kemudian Masuk ke terminal lagi server gateway dan melakukan konfigurasi baru pada /etc/nginx/dumbways 


```
server { 
        server_name alfino.studentdumbways.my.id; 

        location /{
        proxy_pass http://103.31.38.84:3000;
        }
}
```

Kemudian saya akan cek menggunakan domain http://alfino.studentdumbways.my.id/ pada web browser

Apabila berhasil akan tampil aplikasi frontend menggunakan domain http://alfino.studentdumbways.my.id/

# Menggunakan SSL [CertBot](https://certbot.eff.org/) untuk memperaman website

# Apa itu SSL ?

SSL adalah singkatan dari Secure Socket Layer, salah satu komponen penting yang harus dimiliki website. Dengan SSL, transfer data di dalam website menjadi lebih aman dan terenkripsi. Bahkan saking pentingnya, Google Chrome melabeli website tanpa sertifikat SSL sebagai Not Secure.

Apabila sistem keamanan ini ditambahkan pada website Anda, maka URL website akan berubah menjadi HTTPS. Tujuan utama pemasangan SSL adalah sebagai pengaman pertukaran data yang terjadi melalui jaringan internet.

Langkah pertama instalasi certbot

```
sudo snap install core; sudo snap refresh core
```

sudo snap install --classic certbot
Jalankan certbot menggunakan perintah 

```
sudo certbot
```
Apabila berhasil akan muncul kalimat seperti gambar diatas

Konfigurasi SSL Certbot otomatis akan ada pada proxy alfino.studentdumbways.my.id

Kemudian saya akan mengecek juga menggunakan web browser dengan menggunakan alamat https://alfino.studentdumbways.my.id/login

Dan berhasil SSL certbot sudah ada dalam website kalian



