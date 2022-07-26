# Docker Swarm

![image](https://user-images.githubusercontent.com/106061407/173556733-3eb91639-b5d2-4ff9-8167-d26237204c72.png)

Docker Swarm merupakan Container Orchestration System yang disediakan oleh docker untuk memdistribusikan docker contrainer pada system cluster. Dengan kita memanfaatkan docker swarm, kita dapat membuat sebuah management yang centralized baik itu create, destroy, scaling up dan scaling down container pada docker swarm manager.

Kelebihan
Scalable — aplikasi arsitektur layanan mikro umumnya mudah untuk di upgrade dan diatur sesuai dengan kebutuhan pengguna.
Aman — microservices juga umumnya sangat aman dan secure karena telah dirancang untuk mampu mengatasi semua kegagalan yang mungkin terjadi.

Kekurangan 
Rawan error antar service

Pada task kali ini saya akan membuat konfigurasi dari docker swarm yang nantinya saya akan menggunakan VPS dari [IDCLoudHost](idcloudhost.com)


Saya membuaat 3 buah server (Manager, Worker1, Dan Worker2)

Setelah membuat 3 Server kemudian saya login ke3 server nya


Kemudian cek versi docker menggunakan

```
docker version
```

KETERANGAN : Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied

Apabila ada error seperti diatas artinya untuk menjalankan docker kita perlu akses root . Untuk solusinya saya akan memberikan akses root pada docker

```
sudo usermod -aG docker worker
```

Kemudian Relogin dan cek kembali menggunakan docker version

Sekarang saya dapat menjalankan perintah docker tanpa harus mengetikan sudo

KETERANGAN : Lakukan pada ke 3 server supaya masing-masing mendapatkan akses root pada docker

Pertama tama saya akan mengcloning fork https://github.com/sgnd/dumbways-docker-microservices

Kemudian masuk pada direktori /home/pino/dumbways-docker-microservices


Lalu edit file docker-compose.yml

Saya akan mengubah tempat direktori dan image

Kemudian save dan exit

# Konfigurasi Manager untuk Inisialisasi Swarm Cluster

Sekarang kita akan membuat swarm cluster untuk ketiga server. Untuk membuatnya, memerlukan inialisasi pada dockermanager 

KETERANGAN : Saya membuat 3 server docker (pino), (worker), (worker)
user pino 103.183.74.76 (manager) dan worker 103.186.1.150 (worker 1) dan worker 103.186.1.179 (worker 2)

```
docker swarm init –advertise-addr <manager node IP address>
```

Pada hasil diatas, join token sudah digenerate oleh dockermanager, ini nanti digunakan untuk join dockerworkers

Konfigurasi Docker2 dan Docker3 Node untuk join Swarm Cluster

Verifikasi Swarm Cluster

Untuk melihat status node menggunakan command berikut di dockermanager

```
docker node ls
```

Kemudian saya akan build docker-compose 

```
docker-compose build
```

Apabila proses build image sudah selesai bisa cek menggunakan perintah 

```
docker images
```

Selanjutnya saya akan mengcloning fork dan mengedit file docker-compose seperti sebelumnya pada kedua worker dan kemudian di build

Selanjutnya saya akan mendeploy stack todo dan nantinya akan saya scaling pada worker


```
docker stack deploy --compose-file docker-compose.yml stack-todo
```

Selanjutnya saya akan cek salah satu port contohnya port 4000 stack todo service di web browser

Berjalan dengan lancar kemudian saya akan scaling stack-todo_todo-profile kepada kedua worker 

```
docker service scale stack-todo_todo-profile=5
```

Selanjutnya akan terpecah beberapa compose menuju worker

# Contoh 2 docker swarm create yang lebih ringan

Baik cara ke 2 ini saya akan membuat file yang nantinya akan di scale


```
docker service create --replicas 1 --name helloworld alpine ping google.com
```

Untuk melihat service berjalan gunakan 

```
docker service ls
```

Dan untuk melihat container yang sedang berjalan bisa gunakan 
```
docker ps
```
Kemudian saya akan melakukan scaling

```
docker service scale helloworld=5
```
Kemudian cek menggunakan docker ps

Pada manager terdapat 2 container

Pada worker 1 terdapat 1 container

Pada worker 2 terdapat 2 container 