# CI/CD JENKINS

![image](https://user-images.githubusercontent.com/106061407/173822002-c970a6c2-a1c3-4341-909c-9e22d2db3f60.png)

# Apa itu CI/CD?​

CI/CD atau Continuous Integration dan Continuous Deployment merupakan metode untuk mengirimkan perubahan code secara terus menerus hingga aplikasi dapat release ke publik dengan otomatis.

Continuous integration merupakan proses otomatis untuk memastikan semua code sudah berjalan dengan baik, jika terjadi error maka proses tersebut akan diulangi dari awal hingga code tersebut sudah tidak ada error.

Continuous deployment merupakan proses otomatis agar aplikasi yang telah siap di kirim ke server hingga aplikasi dapat diakses secara public.

# Jenkins

Jenkins merupakan sebuah automasi server berbasis open source yang ditulis menggunakan bahasa Java. Salah satu kegunaan Jenkins adalah untuk mengimplementasikan Continuous Integration dan Continous Delivery atau biasa yang disebut CI/CD proses. lebih jelasnya jenkins memudahkan kita untuk Proses seperti testing, building dan deployment yang dapat dijalankan secara otomatis.

# Membuat VPS menggunakan [IDCloudhost](idcloudhost.com)

Saya akan membuat server dengan OS Ubuntu versi 20.04 LTS yang nantinya akan digunakan untuk Jenkins di dalam docker

Apabila aplikasi selesai di build , langsung saja saya login menggunakan terminal


# Install Docker

```
sudo apt  install docker.io
```

Kemudian saya akan memberikan akses root kepada docker supaya saat menjalankan docker tidak perlu menggunakan sudo

```
sudo usermod -aG docker $user
```
Kemudian relogin

```
logout
```

--------

Login Docker

Lakukan login docker terlebih dahulu 

```
docker login
```

# Install Jenkins on Docker


```
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
```

# Install Nginx


```
sudo apt install nginx
```

Kemudian cek pada web browser

# Create a new server block 

Sayaakan mempersiakan domai terlebih dahulu

```
sudo nano /etc/nginx/conf.d/jenkins.conf
```

```
upstream jenkins{
    server 103.171.85.155:8080;
}
 
server{
    listen      80;
    server_name jenkinsalfino.studentdumbways.my.id;
 
    access_log  /var/log/nginx/jenkins.access.log;
    error_log   /var/log/nginx/jenkins.error.log;
 
    proxy_buffers 16 64k;
    proxy_buffer_size 128k;
 
    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;
 
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto https;
    }
 
}
```
Kemudian save dan exit

Kemudian cek pada web browser

# Setup network jenkins

Saya akan membuat network untuk jenkins

```
docker network create jenkins
```

 # Setup work direktory dan run jenkins sebagai container
 
 ```
 docker create --name jenkins --network jenkins --publish 8080:8080 jenkins
 ```
 Untuk menjalankan jenkins gunakan perintah 
 
 ```
 docker container start jenkins
 ```
 
 # Setup jenkins
 
Setelah jenkins containter run, setup jenkins menggunakan browser. Akses jenkin di http://localhost:8080.

Kemudian masuk ke /var/jenkins_home/secrets/initialAdminPassword pada container jenkins dan buka file initialAdminPassword kemudian copy

Kemudian pilih install suggested plugins 

Dan tunggu proses hingga selesai

Isi semua kolom lalu save and continue

Save and finish

Start using jenkins


# Menghubungkan Github dengan mengirimkan ssh key 

Sebelumnya saya sudah menghubungkan akun github saya dengan ssh local

Buat file authorized_keys sebelum mengirimkan ssh

kemudian saya akan memberikan ssh dari local menuju server jenkins saya

```
scp -r id_rsa id_rsa.pub jenkins@103.171.85.155:/home/jenkins/.ssh
```

KET : apabila gagal kalian perlu membuat direktori .ssh pada server jenkins

Kemudian cek ssh di server keygen akan otomatis muncul

# Create a job

Pilih create a job / new item 

Pilih pipeline project lalu ok

Isi description lalu apply


# Add Credential

Sekarang kita akan mencoba membuat credentials pada jenkins, sehingga nantinya jenkins dapat melakukan clone dan deployment otomatis ke Github. Silahkan pilih menu credentials pada menu jenkins yang berada di bawah enu My views, maka akan muncul menu system yang telah di expand, silahkan pilih menu system tersebut. Lalu pilih domain Global credentials (unrestricted), dan kemudian pilih menu Add Credentials. Untuk user jenkins silahkan isikan seperti berikut

Copy id_rsa 

Pilih ssh username with private key

Pastekan id_rsa pada keynya

lalu create

Kemudian saya ingin mencoba untuk me rename jenkins menjadi frontend

# Integrasi dengan git

Langkah pertama saya akan membuat repository baru pada github


Cloning fork https://github.com/dumbwaysdev/wayshub-frontend

Masuk ke direktori wayshub-frontend

Lalu ubah/tambahkan remote

```
git remote add origin git@github.com:pinoezz/Jenkins-Frontend.git
```

```
git remote -v
```

# Build FE

# Membuat [jenkinsfile](https://www.jenkins.io/doc/pipeline/tour/hello-world/)

Kemudian masuk ke server jenkins direktori wayshub-frontend lalu buat file jenkinsfile


```
def secret = 'pinoezz'
def server = 'jenkins.103.171.85.155'
def directory = 'wayshub-frontend'
def branch = 'master'

pipeline{
    agent any
    stages{
        stage ('compose down &  pull'){
            steps{
                sshagent([secret]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker-compose down
                    docker system prune -f
                    git pull origin ${branch}
                    exit
                    EOF"""
                }
            }
        }
        stage ('docker build'){
            steps{
                sshagent([secret]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker-compose build
                    exit
                    EOF"""
                }
            }
        }
        stage ('docker up'){
            steps{
                sshagent([secret]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker-compose up -d
                    exit
                    EOF"""
                }
            }
        }
    }
}

```

Kemudian save dan exit

Selanjutnya add semua , commit dan push ke github

Kemudian cek di github

# Configure Pipeline

Masuk ke localhost:8080 (jenkins)

Pada dashboard pilih project (frontend)

Kemudian apply dan save

Kemudia pilih build now


Untuk melihat log pilih console output

Saya mendapatkan error bahwa docker-compose tidak support harus mengganti versi kemudian selanjutnya

Saya upgrade docker compose 

Kemudian

git add . commit dan push , apabila gagal git pull terlebih dahulu 

dan coba build ulang di jenkins

Sudah berhasil di build dan deploy selanjutnya saya akan cek image yang barusan dibuat pada docker

```
docker images
```

Muncul image baru "pinoezz/wayshub-fe"

Kemudian cek melalui web browser


# Webhook for Jenkins


Pilih repository lalu pilih setting kemudian pilih webhook

Add webhook dan masukan password

Masukan payload url kemudian save


# Build BE


Sebelum build backend saya sarankan untuk membuat database mysql terlebih dahulu dan membuat database yang akan kita gunakan terlebih dahulu

```
docker run -d --name database -p 3306:3306 -v ~/mysql-database:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=P4ssw0rd -e MYSQL_DATABASE=wayshub mysql:latest
```



Setelah building FE dan DB selesai kalian perlu membuat repository dan Jenkinsfile untuk BE

Isi dari Jenkinsfile

```
def secret = 'pinoezz'
def server = 'jenkins@103.171.85.155'
def dir = 'wayshub-backend'
def branch = 'master'

pipeline{
	agent any
	stages{
		stage ('Delete container and images & git pull'){
			steps{
				sshagent([secret]) {
					sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
					cd ${dir}
					docker-compose down
					docker system prune -f
					git pull origin ${branch}
					exit
					EOF"""
				}
			}
		}
	stage ('Build Images'){
                        steps{
                                sshagent([secret]) {
                                        sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                                        cd ${dir}
                                        docker-compose build
                                        exit
                                        EOF"""
                                }
                        }
                }
	stage ('Deploy Container'){
                        steps{
                                sshagent([secret]) {
                                        sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                                        cd ${dir}
                                        docker-compose up -d
                                        exit
                                        EOF"""
                                }
				
                        }
		
               }

	}
	
}
```

Kemudian save lalu lakukan buil pipeline pada jenkins 

Lalu lakukan build

Apabila hasilnya berhasil akan seperti ini semuanya tidak ada error dan otomatis akan build di server Jenkins saya

Kemudian bisa di cek pada web browser

Saya akan melakukan registrasi terlebih dahulu

Apabila dapat melakukan registrasi dan dapat maasuk ke dashboard artinya semua konfigurasi berjalan lancar dan juga bisa dilihat web sudah menggunakan domain dengan reverse proxy yang sebelumnya saya konfigurasikan pada server gateway 

Saya juga sudah melakukan SSL untuk memperaman website menggunakan certbot 
