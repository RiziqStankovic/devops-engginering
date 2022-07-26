# Web Server And Load Balancing


# Pengertian Web Server

Pengertian Web Server menurut pemahaman saya yaitu sebuah peragkat lunak yang memberikan layanan berupa data yang berfungsi untuk menerima permintaan client berupa HTTP/HTTPS atau yang biasa kita pakai contohnya seperti halaman halaman web pada browser (chrome,mozila,dll)

 # Membuat 3 buah server (server gateway, server aplikasi1, server aplikasi2)
 
 Untuk task kali ini saya akan menggunakan multipass yang berfungsi untuk server nantinya
 
Multipass  dirancang untuk pengembang yang menginginkan lingkungan Ubuntu baru dengan satu perintah. Pada dasarnya alat ini dirancang untuk menyederhanakan penginstalan berbagai versi Ubuntu pada mesin virtual yang berjalan di sistem virtualisasi Linux, Windows, dan macOS.
 
 Link Download [Multipass](https://multipass.run/)
 
 Perintah Linux
 
# Install multipass
```
sudo snap install multipass
```
 
Kemudian saya akan membuat server baru menggunakan multipass

```
## multipass launch --name server1
```
Untuk melihat list server gunakan

```
multipass list
```

HINT : Apabila kalian mengalami error seperti gambar di bawah ini saat membuat server multipass kalian dapat ikuti langkah langkah berikut

```
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libgtk3-nocsd.so.0
```
```
exec bash
```

```
sudo apt install gtk3-nocsd
```

Kemudian kalian bisa langsung lanjut membuat servernya

Saya akan membuat 2 server lagi bernama server gateway dan server 2



# Instal web server nginx pada server gateway

Langkah awal saya akan masuk pada server gateway dengan menggunakan perintah

```
multipass shell server-gateway
```

Pada step ini saya akan menginstall nginx terlebih dahulu

```
sudo apt update; sudo apt upgrade
```

```
sudo apt install nginx
```

Untuk cek status nginx gunakan perintah 

```
sudo systemctl status nginx
```

Mengecek versi nginx menggunakan

```
nginx -v
```

kemudian saya akan cek pada web browser
Keterangan  : Apabila muncul seperti gambar diatas artinya web server nginx sudah berhasil diinstall dan sudah berjalan

# Membuat Konfigurasi Revese Proxy

Apa itu Reverse Proxy?​
Reverse proxy adalah konfigurasi standar yang digunakan untuk mengubah jalur traffic, misalkan aplikasi menggunakan port 3000 tetapi agar dapat di akses melalui port 80 maka harus menggunakan reverse proxy.

Pertama-tama masuk ke folder nginx setelah itu buat suatu directory baru telebih dahulu.

```
cd /etc/nginx
```


Setelah itu saya membuat direktori baru lalu masuk ke direktori baru

```
mkdir dumbways
```


```
sudo nano proxy.conf
```

```
server { 
    server_name mydomain.xyz; 
  
    location / { 
             proxy_pass http://127.0.0.1:3000;
    }
}
```

Keterangan : ip pada proxypass kalian isi dengan ip aplikasi1 dan port nodejs

Selanjutnya keluar dari directory dumbways, setelah itu masuk ke dalam file nginx.conf.


Selanjutnya pergi ke-bagian include, setelah itu masukan lokasi dari directory yang bersi konfigutasi yang sudah kalian buat tadi.

Jika sudah sekarang kita tinggal melakukan restart/reload Nginx kita.

```
sudo systemctl restart nginx
```

Sekarang kita akan membuat sebuah virtual host. Untuk membuat virtual host kita harus masuk ke local server kita setelah itu masuk ke dalam file /etc/hosts.

```
sudo nano /etc/hosts
```


# Instal aplikasi nodejs pada server aplikasi1


```
git clone https://github.com/dumbwaysdev/dumbflix-frontend.git
```


```
cd dumbflix-frontend
```


```
sudo apt update
sudo apt install nodejs npm
```
```
npm i
```
```
npm start
```



Kemudian cek di web browser 

```
dumbways.xyz
```



# Konfigurasi load balancing pada server gateway yang mengarah ke server aplikasi2 


# Pengertian Load Balancing


Load balancing adalah proses pendistribusian traffic jaringan ke beberapa server. Ini untuk memastikan salah satu server tidak menanggung terlalu banyak beban permintaan. 
Server website yang kelebihan beban membuat proses muat halaman menjadi lambat, atau bahkan tidak terhubung sama sekali.

Secara sederhana, berikut prinsip kerja load balancing:

Mendistribusikan permintaan klien atau beban jaringan secara efisien di beberapa server. Dengan pemerataan distribusi, website atau aplikasi menjadi lebih tanggap dan stabil ketika diakses oleh pengguna.

Memastikan ketersediaan dengan mengirimkan permintaan hanya ke server yang sedang online

Memberikan fleksibilitas untuk menambah atau mengurangi server sesuai permintaan

# Cara Kerja Load Balancing

Apapun bentuknya, perangkat load balancing mendistribusikan traffic ke beberapa server untuk memastikan tidak ada satu server pun yang menanggung beban berlebih. Secara efektif, load balancing meminimalkan waktu respon server. 

# Konfigurasi Load Balancing

Saat ini saya sudah mempunyai 3 server (gateway , aplikasi 1 dan aplikasi2)


Kemudian install dan jalankan npm pada aplikasi2 / server 2

```
sudo apt update
sudo apt install nodejs npm
```
```
npm i
```
```
npm start
```

Kemudian masuk ke dalam konfigurasi reverse proxy yang sudah kita buat sebelumnya di server gateaway

```
cd /etc/nginx/dumbways
```

```
sudo nano proxy.conf
```

Selanjutnya kita akan tambahkan konfigurasi ke dalam file proxy.conf. Sekarang kita akan coba tambahkan beberapa konfigurasi, kalian dapat menggunakan konfigurasi di bawah ini.

```
upstream wayshub {
    server 10.68.49.174:3000;
    server 10.68.49.101:3000;
}
server {
    server_name loadbalance.dumbways.xyz;

    location / {
             proxy_pass http://dumbways.xyz;
    }
} 
```

keterangan :

Pada bagian upstream kalian dapat mengganti nama domain dengan nama yang kalian inginkan.

Pada bagian server masukan IP dari server kalian, setelah itu diikuti dengan port aplikasi.

Selanjutnya pada bagian proxy_pass ubah dari yang sebelumnya adalah alamat IP dari aplikasi kalian, sekarang kalian samakan dengan nama upstream yang ada di konfigurasi kalian.

Jika sudah sekarang kita coba cek apakah konfigurasi yang sudah kita buat tadi itu error atau tidak.sudo

```
sudo nginx -t
```

Jika tidak ada error jalankan perintah restart nginx untuk merestart nginx kita, karena kita sudah menambahkan 
suatu konfigurasi baru di dalam file reverse proxy kita.
 
 ```
sudo systemctl restart nginx
```

Untuk make sure apakah load balancing yang sudah kita buat tadi berjalan dengan baik atau tidak, kita coba untuk mematikan satu aplikasi kita.

Kita masuk ke dalam salah satu server aplikasi kita, setelah itu kalian hentikan aplikasi kalian CTRL + C.

Sekarang kita coba akses web browser kita lagi setelah itu akses nama domain kalian.


Keterangan :

Jika aplikasi kalian masih bisa di akses berarti konfigurasi Load Balance kalian berhasil dan tidak ada error `

