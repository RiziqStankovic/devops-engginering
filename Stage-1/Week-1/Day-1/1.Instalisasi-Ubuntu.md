# TASK-DAY-1


## Project Management

Konfigurasi environment dan instalasi linux server, dimana server tersebut dapat terkoneksi ke internet dan juga dapat di akses melalui SSH / Browser.



- [1] Definisikan apa itu DevOps menurut pemahamanmu
- [2] Buatlah environment untuk server (CPU, Memory, Storage dan Network)
- [3] Install linux ubuntu server 18.x / 20.x dengan VMware
- [4] Gunakan IP static untuk linux ubuntu server
- [5] Buat swap memory and root partition
- [6] Docker terinstal secara otomatis setelah proses instalasi linux ubuntu server
- [7] Local server dapat terkoneksi dengan internet

### Solve ###


### =>[1]  Devops adalah sebuah kolaborasi antara Development (programmer) dan Operation (operasi,Staf IT lainnya) dengan tujuan untuk meningkatkan aplikasi atau produk agar lebih cepat dalam hal pengembangan maupun dalam hal pelayanan yang lebih efektif.

### =>[2] Sebelum konfigurasi ke server, perlu disiapkan beberapa hal antara lain:
- File ISO Ubuntu Server 20.04.4
- CPU 1 GHz rekomendasi 2 Ghz keatas
- Aplikasi Virtual box atau VMware
- Ram minimal 512 rekomendasi 4 GB
- Hard disk minimal 10 GB rekomendasi 20 GB
- Koneksi Internet

### =>[3] Step Install Ubuntu Server

 Link untuk mendownload https://ubuntu.com/download/server lalu klik "option 2 - Manual server installation"

2. Download VMware terlebih dahulu https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html 

3. Download Virtual Box
https://www.virtualbox.org/

 setelah Download lgsung install dan konfigurasi klik drive (empty)
![Img 1](./assets/1.1.JPG)
 tentukan opsi nama dan pilih juga file .iso yang sudah download [ikuti step gambar dibawah ini]
![Img 1](./assets/1.2.png)
![Img 1](./assets/1.3.png)
![Img 1](./assets/1.4.png)
![Img 1](./assets/1.5.png)
Atur hardisk kemudian langsung create 
![Img 1](./assets/1.6.png)
 
 masuk pada bios configurasi lagi
![Img 1](./assets/1.8.png)
![Img 1](./assets/1.9.png)
![Img 1](./assets/2.0.png)


### =>[4] konfigurasi IP Static menjadi Nat
set sperti gambar dibawah ini

![Img 1](./assets/2.1.png)

![Img 1](./assets/2.2.png)

Ganti menjadi ipv4 kmudian sesuiakan dengan kebutuhan
![Img 1](./assets/2.3.png)
otomatis akan berubah menjadi static
![Img 1](./assets/2.4.png)

### =>[5] Konfigurasi partisi memori
setting partisi swap,boot. dan (/)root

![Img 1](./assets/2.5.png)
![Img 1](./assets/2.6.png)
hitung settingan sesuai kebutuhan
atau ikuti step ini
![Img 1](./assets/3.0.png)
![Img 1](./assets/3.1.png)


![Img 1](./assets/3.2.png)
![Img 1](./assets/3.4.png)

![Img 1](./assets/3.5.png)
configurasi lagi apakah sudah semua, hati-hati untuk (/)root lebih baik diisi paling banyak
![Img 1](./assets/3.6.png)
![Img 1](./assets/3.7.png)
masukan username dan passwd sbagi konfirmasi server
![Img 1](./assets/3.8.png)
![Img 1](./assets/3.9.png)
ikuti step ini dan langsung start Done
![Img 1](./assets/3.91.png)

![Img 1](./assets/3.93.png)

![Img 1](./assets/3.94.png)
kalau udah sampai step ini berarti sudah aman
### =>[6] Install Docker



### =>[7] Koneksi Lokal Server 
login server masukan user dan passwd yang sudah di buat
![Img 29](./assets/4.0.png)
kemudian ping google.com
![Img 30](./assets/4.1.png)

done cukup skian tinggal ikuti step by step sesimpel itu kok