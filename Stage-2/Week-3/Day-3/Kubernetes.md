# Kubernetes 

![image](https://user-images.githubusercontent.com/106061407/175206118-a6b8588e-ee85-4235-8cfd-57134e2e2a33.png)

Kubernetes adalah platform open source untuk mengelola kumpulan kontainer dalam suatu cluster server. Platform ini pertama kali dikembangkan oleh Google dan kini dikelola oleh Cloud Native Computing Foundation (CNCF) sebagai platform manajemen kontainer yang cukup populer.

Kontainer sendiri adalah environment dengan sumber daya, CPU, dan sistem file untuk satu aplikasi. Jadi, aplikasi tersebut akan memiliki sumber daya sendiri. Keuntungannya, aplikasi jadi tidak mudah mengalami downtime.

Atau singkatnya KUBERNETES itu adalah salah satu aplikasi opensource  untuk automation scaling dan manajemen aplikasi berbasis container

Untuk konsep cara kerja dari kubernetes adalah seperti di atas atau spesifiknya :

# KUBERNETES MASTER :

Kube-apiserver bertugas sebagai API yang digunakan untuk berinteraksi dengan Kubernetes Cluster

Etcd bertugas untuk sebagai database untuk menyimpan data Kubernetes Cluster

Kube-scheduler bertugas untuk memperhatikan aplikasi yang kita jalankan dan meminta Node untuk menjalankan aplikasi yang kita jalankan

Kube-controller-manager bertugas melakukan kontrol terhadap Kubernetes Cluster

Cloud-controller-manager bertugas untuk melakukan kontrol terhadap interaksi dengan cloud provider

# Kubernetes Nodes : 

Kubelet berjalan di setiap Node dan bertugas untuk memastikan bahwa aplikasi kita berjalan di Node

Kube-proxy berjalan di setiap Node dan bertugas sebagai proxy terhadap arus network yang masuk ke aplikasi kita dan sebagai load balancer juga

Container-manager berjalan di setiap Node dan bertugas sebagai container manager. Kubernetes mendukung beberapa container manager seperti Docker, containerd, cri-o, rktlet, dan yang lainnya.

-----------------------------------

Cluster: Beberapa server yang seolah-olah dijadikan 1 kesatuan sehingga tercipta lingkungan server yang dinamis.

Node / Worker: Merupakan server-server yang dijadikan cluster, berguna untuk menjalankan container

Pods: Satuan terkecil dalam kubernetes yang berguna sebagai pembungkus container. Biasanya dalam 1 pods terdapat 1 container atau lebih.

Containers: Aplikasi yang berjalan di dalam pods seperti halnya docker container, dimana semua kebutuhan untuk aplikasi sudah berada di dalam container tersebut.

CNI: Container Network Interface merupakan penghubung antara cluster, worker, container, volume dan sebagainya. Beberapa cni yang terkenal adalah calico, flannel, weavenet, canal

Persistent Volume: Sama halnya seperti docker volume yang sifatnya untuk menyimpan data di local server.

Ingress: Merupakan cara untuk supaya aplikasi dapat diakses menggunakan http / https, biasanya dalam bentuk domain seperti https://subdomain.example.co

-----------------------------------

# Setup Kubernetes And Deploy Microservices

Sebelumnya apa itu Microservices? Microservices adalah kebalikannya dari Monolith, dimana aplikasi dipecah menjadi kecil-kecil, dimana tiap aplikasi hanya mengurus satu tugas dengan baik, dan semua aplikasi saling berkomunikasi.

Sekarang saya akan mempersiapakan 3 server (1 Manager / sebagai controller) , (2 Worker) dari [IdCloudHost](idcloudhost.com)


Untuk Manager saya gunakan CPU 2 Core, RAM 4 GB, SSD 20 GB

Untuk masing-masing worker saya gunakan CPU 1 Core, RAM 1GB, SSD 20 GB

Tunggu hingga proses building selesai ...

Test login menggunakan terminal

-----------------------------

# Install Docker & Kubernetes

Sebelum melakukan instalasi kubernetes diwajibkan terlebih dahulu untuk NON-Aktifkan firewall dan SWAP

```
ufw disable
```

```
swapoff -a; sed -i '/swap/d' /etc/fstab
```

kemudian kita perlu juga update kernel

```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

Kemudian Restart system

```
sysctl --system
```

--------------------------------------------

KETERANGAN : Dikarenakan saya sudah menambahkan catalog Docker pada saat pembuatan server jadi saya tidak perlu install docker lagi

Untuk Instalasi docker kalian dapat ikuti tutorial di bawah ini :

[Install Docker](https://docs.docker.com/engine/install/)

[Install Docker Compose](https://docs.docker.com/compose/install/)

Saat ini saya gunakan Docker Version:  20.10.7

-------------------------------------------------------

Selanjutnya saya akan konfigurasi docker daemon

```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
```

Lalu restart docker

```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```
-----------------------------------------------

Selanjutnya Instalasi Kubernetes dan sekaligus install kubelet kubeadm kubectl

```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
```

```
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Konfigurasi kubeadm


```
kubeadm init
```

Untuk versi Kubernates yang saya pakai yaitu versi: v1.24.2

-----------------------------------------

Konfigurasi kubernetes agar dapat menjalankan perintah


```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Selanjutnya kita harus deploy Container Network Interface (CNI) yang berbasis add-ons Pod network seperti calico, kube-router, dan weave-net. Add-ons Pod network tersebut berfungsi untuk membuat Pod berhubungan satu sama lain.

Deploy Add-on Pod Network Calico :


KETERANGAN : GUNAKAN Container Network Interface (CNI) CALICO VERSI TERBARU [PROJECTCALICO](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)

```
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```

Mengecek semua pods gunakan perintah

```
kubectl get pods --all-namespaces
```

Lalu lakukan join cluster dengan perintah berikut 

```
kubeadm token create --print-join-command
```

Kemudian copy semua hasil outputnya 

NOTE : INSTALL DAHULU KUBERNATES DENGAN VERSI YANG SAMA !!!

Copy pada kedua worker

[WORKER 1]

[WORKER 2]

Kemudian cek koneksi nodes menggunakan perintah

```
kubectl get nodes
```

# Deploy Simple App

Kemudian saya akan mendeploy nginx 

```
kubectl create deploy nginx --image nginx
```

dan untuk buildnya gunakan perintah

```
kubectl expose deploy nginx --port 80 --type NodePort
```

Cek service gunakan perintah 

```
kubectl get svc
```

Untuk melihat informasi lengkap pada pod gunakan 

```
kubectl describe pod nama pod
```

Nginx berjalan pada worker 2  http://103.186.1.51:31324/

Selanjutnya saya akan cek pada web browser