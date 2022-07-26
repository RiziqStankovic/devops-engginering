# Server Monitoring On Docker Using Ansible

![image](https://user-images.githubusercontent.com/106061407/174720424-cc037e82-16dd-4bac-9814-8dd1f2bfd007.png)

Monitoring server adalah kegiatan memantau sumber daya sistem server seperti: penggunaan CPU (CPU Usage), konsumsi memori (Memory RAM Consumption), jaringan, penggunaan disk (disk usage), dan lain sebagainya. Karena performa aplikasi atau website Anda sebagian besar bergantung pada performa server

Untuk task kali ini saya akan membuat suatu konfigurasi Monitoring server dan untuk semua instalasinya saya akan membuat menggunakan  Ansible

Menggunakan Ansible untuk Mengelola Server yang Lebih Mudah

Ansible adalah sebuah provisioning tool yang dikembangkan oleh RedHat. Dimana kamu dapat mencatat setiap proses deployment ataupun konfigurasi yang biasa dilakukan berulang - ulang terhadap beberapa server. Misal saat pertama kali kita memasang Ubuntu Server di 10 mesin, maka kita akan melakuan apt-get update serta memasang beberapa komponen seperti PHP5 dan Apache2. Sebenarnya tidak akan menjadi masalah, bila kita hanya melakukan sedikit hal. Tapi bayangkan bila harus melakukan konfigurasi yang cukup kompleks dan dilakukan secara berulang - ulang ke 10 mesin tersebut.

-------------------------------

# Skema Monitoring server

Berikut adalah skema yang akan di bangun, jadi saya akan membuat skema diatas menggunakan server dari [IDCloudHost](https://console.idcloudhost.com/) dan akan berjalan menggunakan docker

--------------------------------

# Membuat VPS (Virtual Private Server) [IDCloudHost](https://console.idcloudhost.com/)

Pertama-tama saya akan membuat 2 buah server yaitu server mentoring dan app menggunakan os Ubuntu 20.04 LTS

# Membuat Konfigurasi Ansible untuk install docker 

Install Ansible 

```
sudo apt install ansible
```

---------------------------------

Cek versi ansible

```
ansible --version
```

------------------------------

# Membuat direktori untuk menyimpan file/konfigurasi ansible

Untuk nama direktori bebas , disini saya akan buat direktori bernama "ansibleku"

```
mkdir ansibleku
```

---------------------------------

# Membuat SSH keygen 

Membuat SSH keygen pada server supaya dapat dikenali oleh local

```
ssh-keygen
```

Copy isi dari id_rsa.pub dan pastekan di authorized_keys

Kemudian copy id_rsa dan buat file .pem pada local/ansibleku

Lalu save dan exit

Jangan lupa untuk mengubah hak akses file .pemnya menjadi read only kemudian test mengoneksikan ke server

----------------------------------

# Membuat Inventory Ansible

Buat file bernama Inventory kemudian isi dengan ip server kalian

```
[monitoring]
103.55.37.187 ansible_user=monitor
```

-------------------------------------

# Membuat Ansible.cfg
 
Buat file baru bernama ansible.cfg kemudian isi seperti berikut kemudian save
 
 ```
 [defaults]
inventory = Inventory
private_key_file = pino.pem
```
Kemudian cek menggunakan 

```
ansible all-m ping
```

# Install nginx menggunakan ansible-playbook

Buat file baru bernama nginx.yml

```
- hosts: monitoring
  become: yes
  gather_facts: yes
  tasks:
        - name: 'install nginx'
          apt: 
            name:
             - nginx
            state: latest
 ```
 
  
Kemudian saya akan install nginx.yml nya 

```
ansible-playbook nginx.yml
```

Kemudian cek pada web browser

Apabila muncul tampilan seperti diatas maka hasilnya berhasil !!!

------------------------------------------------

# Install Docker menggunakan ansible-playbook

Referensi

https://docs.docker.com/engine/install/ubuntu/

https://docs.docker.com/compose/install/compose-plugin/#installing-compose-on-linux-systems

```
- hosts: monitoring
  become: yes
  gather_facts: yes
  tasks:
         - name: 'update'
           apt:
            update_cache: yes

         - name: 'upgrade'
           apt:
            upgrade: dist


         - name: 'install dependencies'
           apt:
             name:
             - ca-certificates
             - curl
             - gnupg
             - lsb-release

         - name: 'add docker gpg key'
           apt_key:
            url: https://download.docker.com/linux/ubuntu/gpg

         - name: 'add repository docker'
           apt_repository:
             repo: deb  https://download.docker.com/linux/ubuntu focal stable

         - name: 'install docker engine'
           apt: 
            name:
             - docker-ce
             - docker-ce-cli
             - containerd.io
             - docker-compose-plugin

         - name: 'update'
           apt:
            update_cache: yes

         - name: 'install docker-compose'
           shell: curl -SL https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose


         - name: 'set permision for docker'
           shell: sudo chmod +x /usr/local/bin/docker-compose
        
         - name: 'docker without sudo'
           shell: sudo usermod -aG docker monitor


```

Setelah beberapa kali kesalahan akhirnya saya dapat membuat playbook untuk docker dan apabila sukses akan seperti gambar diatas

Kemudian saya akan install docker.yml nya

```
ansible-playbook --syntax-check docker.yml 
```

Kemudian saya akan cek hasil install docker pada server

Apabila berhasil , dapat menjalankan docker , dan dapat menjalankan docker tanpa sudo

# Install Monitoring server dengan Node Exporter , Prometheus dan Grafana

Pertama tama saya akan menginstall docker pada server app  menggunakan ansible-playbook

Kirim ssh dari server monitor/local supaya dapat dikenali lokal

Kemudian kirim atau masukan id_rsa.pub ke autorized_keys

Kemudian tambahkan user pada inventory

Cek koneksi ansible

---------------------------------------------

# Install Docker

Edit file docker.yml ganti host dan docker without sudo nya menjadi app lalu save

```
- hosts: app
  become: yes
  gather_facts: yes
  tasks:
         - name: 'update'
           apt:
            update_cache: yes

         - name: 'upgrade'
           apt:
            upgrade: dist


         - name: 'install dependencies'
           apt:
             name:
             - ca-certificates
             - curl
             - gnupg
             - lsb-release

         - name: 'add docker gpg key'
           apt_key:
            url: https://download.docker.com/linux/ubuntu/gpg

         - name: 'add repository docker'
           apt_repository:
             repo: deb  https://download.docker.com/linux/ubuntu focal stable

         - name: 'install docker engine'
           apt: 
            name:
             - docker-ce
             - docker-ce-cli
             - containerd.io
             - docker-compose-plugin

         - name: 'update'
           apt:
            update_cache: yes

         - name: 'install docker-compose'
           shell: curl -SL https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose


         - name: 'set permision for docker'
           shell: sudo chmod +x /usr/local/bin/docker-compose
        
         - name: 'docker without sudo'
           shell: sudo usermod -aG docker app
```

```
ansible-playbook docker.yml
```
Berhasil install docker di server app 

-----------------------------------------------------------------

# Membuat ansible-playbook Monitoring 

Sekarang setelah berhasil install docker menggunakan ansible-playbook kemudian saya akan membuat ansible-playbook untuk menginstall node_exporter , prometheus dan grafana on top docker

 Referensi : 
 
 [Install Node Exporter](https://medium.com/nerd-for-tech/tutorial-how-to-deploy-prometheus-and-node-exporter-as-containers-on-a-remote-server-with-5-af5b449be49b)

 [Pull Image](https://docs.ansible.com/ansible/2.9/modules/docker_image_module.html)
 
 [Bitnami Prometheus Image](https://hub.docker.com/r/bitnami/prometheus/)
 
 [Docker Create Container](https://docs.ansible.com/ansible/2.5/modules/docker_container_module.html)
 
 [Yaml editor online](https://codebeautify.org/yaml-editor-online)
 
 [Grafana Dashboard](https://grafana.com/grafana/dashboards/)
 
 Pertama-tama membuat file kongigurasi prometheus 
 
 ![image](https://user-images.githubusercontent.com/106061407/174966094-2b86314e-4446-4331-9d13-a9e7da4225c2.png)

File diatas saya simpan di folder ansibleku

```

global:
 scrape_interval: 10s
scrape_configs:
 - job_name: 'prometheus_metrics'
   scrape_interval: 5s
   static_configs:
     - targets: ['103.55.37.187:9090'] #Localhost:port
 - job_name: 'node_exporter_metrics'
   scrape_interval: 5s
   static_configs:  
     - targets: ['103.55.37.187:9100','103.55.37.185:9100'] #Localhost:port
 
```

Berikut adalah file konfigurasi prometheus.yml

Kemudian untuk file Inventory tambahkan [all] (menggabungkan semua user)

```
[all]
103.55.37.187 ansible_user=monitor
103.55.37.185 ansible_user=app

[monitoring]
103.55.37.187 ansible_user=monitor

[app]
103.55.37.185 ansible_user=app
```
NOTE : GANTI IP DIATAS DENGAN IP SERVER KALIAN 

Kemudian saya akan membuat ansible-playbook untuk instalasi Node Exporter , Prometheus dan Grafana

```
- hosts: all
  become: yes
  gather_facts: yes
  tasks:
       - name: 'update'
         apt:
          update_cache: yes

       - name: 'upgrade'
         apt:
          upgrade: dist

       - name: 'Run Node Exporter'
         shell: docker run -d --net="host" --pid="host" -v "/:/host:ro,rslave" --name node_exporter quay.io/prometheus/node-exporter --path.rootfs=/host

- hosts: monitoring
  tasks:
        - name: 'Make volume folder'
          file:
           path: /home/monitor/prometheus
           state: directory

        - name: 'Copy Configuration Prometheus'
          copy:
           src: /home/pino/ansibleku/prometheus.yml
           dest: /home/monitor/prometheus
        
        - name: 'install ptyhon docker'
          shell: sudo apt install python3-docker -y


        - name: 'Login DockerHub'
          docker_login:
           username: {username}
           password: {password}

        - name: 'Pull Prometheus'
          docker_image:
           name: bitnami/prometheus
           source: pull
           

        - name: 'Container Prometheus'
          docker_container:
           name: prometheus
           image: bitnami/prometheus
           ports:
             - 9090:9090
           volumes: /home/monitor/prometheus:/etc/prometheus

        - name: 'Pull Grafana'
          docker_image:
           name: grafana/grafana    
           source: pull

        - name: 'Container Grafana'
          docker_container:
           name: grafana
           image: grafana/grafana
           ports:
             - 3000:3000
```             

Saya akan check file yml apakah sudah valid menggunakan [Yaml editor online](https://codebeautify.org/yaml-editor-online)

Apabila hasil seperti ini artinya sudah valid dan untuk mengecek nya jg bisa gunakan perintah 

```
ansible-playbook --syntax-check monitoring.yml
```
Selanjutnya running playbook

```
ansible-playbook monitoring.yml 
```

Apabila proses sudah selesai akan seperti gambar diatas kemudian kita cek pada docker di server monitor dan app

Semua container sudah berjalan kemudian cek juga pada web browser

Node Exporter [OK]

Prometheus [OK]

Grafana [OK]

Selanjutnya saya akan konfigurasi pada grafana 

Masukan username dan password default (admin) / (admin)

Kemudian buat password baru sesuai keinginan kalian lalu submit

Pada tampilan ini kalan "DATA SOURCES"

Kemudian pilih Prometheus

Kemudian pada URL kalian isikan IP dan port dari Prometheus

Save & Test

Kemudian pilih Dashboard dan pilih New Dashboard

Pilih add new panel

Disini pilih metric apa yang mau kita monitoring kemudian run queries

Kemudian untuk mengganti tampilan dashboard kalian bisa mengambil template nya di [Grafana Dashboard](https://grafana.com/grafana/dashboards/)

Contoh saya akan gunakan dashboard ini , Klik bagian copy ID to Clipboard [1860]

Selanjutnya perlu import pada grafana dashboard

Kemudian load dan pilih prometheus

Apabila tampilan seperti ini artinya berhasil di import :D

# Membuat Reverse Proxy untuk Monitoring

Dikarenakan pada step sebelumnya saya sudah install nginx jadi saya akan langsung melakukan konfigurasi reverse proxy

Saya akan menggunakan domain dari [CloudFlare](https://dash.cloudflare.com/)

Domain : monitoring.alfino.studentdumbways.my.id

Kemudian masuk ke server monitoring

Masuk ke direktori /etc/nginx dan buat direktori untuk meyimpan konfigurasi reverse proxy 

```
server {
        server_name monitoring.alfino.studentdumbways.my.id;

        location / {
                proxy_pass http://103.55.37.187:3000;
        }
}
```

Masukan direktori konfigurasi reverse proxy pada nginx.conf 

Cek syntax

```
sudo nginx -t
```
```
sudo systemctl restart nginx.service 
```
Langkah selanjutnya saya akan pasangkan SSL certbot pada web http://monitoring.alfino.studentdumbways.my.id/


```
sudo snap install core; sudo snap refresh core
```

Instalasi certbot

```
sudo snap install --classic certbot
```

Kemudian jalankan dan pilih domain  

```
sudo certbot
```

Berikut hasilnya

https://monitoring.alfino.studentdumbways.my.id/
