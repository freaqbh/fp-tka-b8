

 ## ***KELOMPOK B8***
  | Nama      | NRP         |
  |-----------|-------------|
  | Nicholas Emmanuel Fade   | 5027231020  |
  | Muhammad Abhinaya Al-Faruqi  | 5027231011  |  
  | Muhammad Rizq Taufan | 5027231021  |
  | Veri Rahman | 5027231088 |

# Permasalahan

Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: sentiment-analysis.py 

Kemudian juga disediakan sebuah Frontend sederhana menggunakan index.html dan styles.css dengan tampilan antarmuka.

Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah 1 juta rupiah per bulan (65 US$) konfigurasi cloud terbaik seperti apa yang bisa dibuat?

# Rancangan Arsitektur dan Tabel spesifikasi harga
Kami Melakukan Ujicoba terlebih dahulu ke sebuah virtual machine agar dapat memaksimalkan kemampuannya dan mendapatkan hasil yang paling bagus kami menggunakan Flask sebagai backend Mongodb sebagai databasenya dan HAProxy sebagai load balancernya

![Screenshot from 2024-06-21 11-17-48](https://github.com/freaqbh/fp-tka-b8/assets/123524655/a4b59052-30b8-4807-b68a-8720d083b70e)

berikut merupakan spesifikasi tabel harga

![Screenshot from 2024-06-21 11-29-50](https://github.com/freaqbh/fp-tka-b8/assets/123524655/28f71944-7375-4610-b2cb-39432676f2ea)


# Langkah Implementasi Aplikasi

1. Membuat Config Vagrantfile
   berikut merupakan konfigurasi dari vagrant file
```
   Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  # VM untuk Load Balancer
  config.vm.define "loadbalancer" do |loadbalancer|
    loadbalancer.vm.box = 'ubuntu/bionic64'
    loadbalancer.vm.hostname = "loadbalancer"
    loadbalancer.vm.network :private_network, ip: "192.168.56.10"
    loadbalancer.vm.network "forwarded_port", guest: 80, host: 3000
    loadbalancer.vm.network "forwarded_port", guest: 8404, host: 8404
    loadbalancer.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
    loadbalancer.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt -y install haproxy
    SHELL
  end

  # VM untuk Backend 1
  config.vm.define "backend1" do |backend1|
    backend1.vm.box = "ubuntu/bionic64"
    backend1.vm.hostname = "backend1"
    backend1.vm.network "private_network", ip: "192.168.56.11"
    backend1.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
    backend1.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y python3-pip mongodb
      systemctl start mongodb
      systemctl enable mongodb
      pip3 install -r /vagrant/backend/requirements.txt
    SHELL
  end

  # VM untuk Backend 2
  config.vm.define "backend2" do |backend2|
    backend2.vm.box = "ubuntu/bionic64"
    backend2.vm.hostname = "backend2"
    backend2.vm.network "private_network", ip: "192.168.56.12"
    backend2.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
    backend2.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y python3-pip
      pip3 install -r /vagrant/backend/requirements.txt
    SHELL
  end
  config.vm.synced_folder "./backend", "/vagrant/backend"
  config.vm.synced_folder "./frontend", "/vagrant/frontend"
end
```
 setelah membuat Vagrantfile lakukan perintah vagrant up

 ![Screenshot from 2024-06-21 11-35-45](https://github.com/freaqbh/fp-tka-b8/assets/123524655/99c9cfb1-9ef7-4013-b016-f040e833b02a)

2. Masuk ke setiap virtual mesinnya dengan perintah "vagran ssh [nama virtual mesin]"

   
   ![Screenshot from 2024-06-21 11-38-32](https://github.com/freaqbh/fp-tka-b8/assets/123524655/26458342-b9e9-4a37-9f86-62655c57984f)

3. Mengatuh HAproxy config
   untuk melakukan configurasi haproxy lakukan "sudo nano /etc/haproxy/haproxy.cfg" dan tambahkan configurasinya

![Screenshot from 2024-06-21 11-40-30](https://github.com/freaqbh/fp-tka-b8/assets/123524655/704a9401-6c22-4bb1-93de-660a1a0b9dbc)

setelah mengatur jalankan perintah "sudo systemctl restart haproxy"

4. jalankan sentiment-analysis.py di setiap backend VM

   ![Screenshot from 2024-06-21 11-43-47](https://github.com/freaqbh/fp-tka-b8/assets/123524655/4312fb45-4153-42c4-aaa9-c990b81f81f1)

5. Masuk ke loadbalancer virtual machine dan jalankan perintah "python3 -m http.server 8000" di folder frontendnya
  
![Screenshot from 2024-06-21 13-00-00](https://github.com/freaqbh/fp-tka-b8/assets/123524655/4796062c-1645-4b2d-8841-28b6b451def7)

dan lakukan test terhadap aplikasi di browser menggunakan Ip address loadbalancer "http://192.168.56.10:8000"

![Screenshot from 2024-06-21 12-59-50](https://github.com/freaqbh/fp-tka-b8/assets/123524655/5331462e-b046-4bc1-bdf8-688f18dcbf49)


# Hasil pengujian Loadtesting menggunakan Locust

1. RPS Maksimum (load testing 60s)

   ![Screenshot from 2024-06-21 13-22-02](https://github.com/freaqbh/fp-tka-b8/assets/123524655/3bb84657-3d57-4ee3-97cf-459545055d34)
   
![Screenshot from 2024-06-21 13-22-33](https://github.com/freaqbh/fp-tka-b8/assets/123524655/f534b594-1d2f-4177-8fc0-65e8612a8bff)

kami melakukan uji coba dengan 100 number of use dan sebanyak 25 Ramps up dapat dilihat di table tersebut kami mendapatkan 26.2 rps. Pada testing ini tidak ditemukan failure

2. Peak Concurrency Maksimum (spawn rate 25, load testing 60 detik)
   
![Screenshot from 2024-06-21 13-20-24](https://github.com/freaqbh/fp-tka-b8/assets/123524655/7c9e0d6f-c1a1-4afc-be1a-e049aef94e57)

![Screenshot from 2024-06-21 13-22-41](https://github.com/freaqbh/fp-tka-b8/assets/123524655/c3430796-9fda-49c8-92cd-824826574ce3)

pada pengujian kali ini kami mendapatkan rps tertinggi 17.2 dan rata-rata rps nya sebanyak 15.2

3. Peak Concurrency Maksimum (spawn rate 50, load testing 60 detik)

   
![Screenshot from 2024-06-21 13-22-02](https://github.com/freaqbh/fp-tka-b8/assets/123524655/f874a608-ac0a-4001-96ee-4cd8bb71155b)

![Screenshot from 2024-06-21 13-22-33](https://github.com/freaqbh/fp-tka-b8/assets/123524655/034693b2-8080-447e-9516-765e68689040)

pada pengujian kali ini kami mendapatkan rps tertinggi 27.7 dan rata-rata rps 26.2

4. Peak Concurrency Maksimum (spawn rate 100, load testing 60 detik)

   
![Screenshot from 2024-06-21 13-24-56](https://github.com/freaqbh/fp-tka-b8/assets/123524655/13116706-75c0-47bf-96bf-39671702932e)

![Screenshot from 2024-06-21 13-25-09](https://github.com/freaqbh/fp-tka-b8/assets/123524655/75ae91d0-bd8e-4569-9718-29423b373c6c)

5. Peak Concurrency Maksimum (spawn rate 200, load testing 60 detik)

   
![Screenshot from 2024-06-21 13-43-04](https://github.com/freaqbh/fp-tka-b8/assets/123524655/2f851980-e171-4702-97ee-946e4458d39c)
![Screenshot from 2024-06-21 13-43-13](https://github.com/freaqbh/fp-tka-b8/assets/123524655/82f4451e-7a57-4bfd-abbb-d8adaf842269)

6. Peak Concurrency Maksimum (spawn rate 500, load testing 60 detik)
   
![Screenshot from 2024-06-21 13-28-26](https://github.com/freaqbh/fp-tka-b8/assets/123524655/5c32dea8-9544-4696-b0ee-96fbd43bd4d8)

![Screenshot from 2024-06-21 13-28-21](https://github.com/freaqbh/fp-tka-b8/assets/123524655/6077de92-0b8c-4838-8b21-7237f7e273cb)

# Kesimpulan dan saran

kesimpulan yang saya dapatkan adalah load balancing memang dapat mengoptimasikan kemampuan server dalam meningkatkan kemampuan server untuk menampung user, dan untuk setiap pengujian melakukan restart mongodb di vm nya agar dapat menghasilkan hasil optimal ketika dilakukan load testing. 


# Revisi menggunakan satu VM
Kami memutuskan untuk menguji menggunakan satu virtual machine saja dan membandingkan apakah menggunakan satu virtual machine lebih baik dari pada menggunakan 3 virtual machine, yang mana satu virtual machine ini tidak menggunakan loadbalancing hanya menggunakan backend, frontend dan mongodb

# Rancangan Arsitektur dan Tabel spesifikasi harga
berikut adalah rancangan dari ujicoba menggunakan satu virtual machine
![Screenshot from 2024-06-29 20-03-18](https://github.com/freaqbh/fp-tka-b8/assets/123524655/16712229-060c-4339-a9cf-74abf5ef628b)
dan berikut adalah tabel harga menggunakan satu virtual machine yang notabene lebih mahal
![Screenshot from 2024-06-29 20-09-05](https://github.com/freaqbh/fp-tka-b8/assets/123524655/3488433f-bdca-46b0-b069-89967af152df)

# Langkah Implementasi Aplikasi
1. Membuat Config Vagrantfile
   berikut merupakan konfigurasi dari vagrant file
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"  # Ubuntu 18.04 LTS

  config.vm.network "forwarded_port", guest: 5000, host: 5000
  config.vm.network "forwarded_port", guest: 8000, host: 8000

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"          # Set RAM to 2 GB
    vb.cpus = 2                 # Set CPU to 2 vCPUs
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y python3-pip mongodb
    systemctl start mongodb
    systemctl enable mongodb
    pip3 install -r /vagrant/backend/requirements.txt
  SHELL

  config.vm.synced_folder "./backend", "/home/vagrant/backend"
  config.vm.synced_folder "./frontend", "/home/vagrant/frontend"
end
```
 setelah membuat Vagrantfile lakukan perintah "vagrant up"
![Screenshot from 2024-06-29 18-09-36](https://github.com/freaqbh/fp-tka-b8/assets/123524655/8af25b8d-6aa3-42b9-a37c-17c6662a87a3)

2. Masuk ke virtual mesinnya dengan perintah "vagran ssh"
![Screenshot from 2024-06-29 18-22-03](https://github.com/freaqbh/fp-tka-b8/assets/123524655/3f061a72-b8b1-49ce-871d-9e0d17c93b58)

3. jalankan sentiment-analysis.py di VM
![Screenshot from 2024-06-29 18-44-42](https://github.com/freaqbh/fp-tka-b8/assets/123524655/980481e0-97ae-4e08-b606-97fa592b437e)

4. jalankan perintah "python3 -m http.server 8000" di folder frontendnya
![Screenshot from 2024-06-29 18-44-35](https://github.com/freaqbh/fp-tka-b8/assets/123524655/fddf8787-ddb3-43cc-851e-4271e72a5d4f)
dan lakukan test terhadap aplikasi di browser "http://localhost:8000"
![Screenshot from 2024-06-29 18-43-58](https://github.com/freaqbh/fp-tka-b8/assets/123524655/7ae31987-940e-4e15-a862-d39580bf7749)

# Hasil pengujian Loadtesting menggunakan Locust
1. RPS Maksimum (load testing 60s)
![100 15](https://github.com/freaqbh/fp-tka-b8/assets/123524655/3dcfe356-1bc7-44a9-a689-0c8841bc5726)
![100 15 c](https://github.com/freaqbh/fp-tka-b8/assets/123524655/a7adf435-0bc3-4b2e-b83b-e2f2fa18e4e9)
kami melakukan uji coba dengan 100 number of use dan sebanyak 15 Ramps up dapat dilihat di table tersebut kami mendapatkan 17 rps. Pada testing ini tidak ditemukan failure

2. Peak Concurrency Maksimum (spawn rate 50, load testing 60 detik)
![50 20](https://github.com/freaqbh/fp-tka-b8/assets/123524655/c0922b01-4846-45a0-8b14-748e7a276707)
![50 20 c](https://github.com/freaqbh/fp-tka-b8/assets/123524655/a13ad09a-f05d-4e99-832b-4688d2bb395c)
disini kami melakukan dengan 50 number of use dan sebanyak 20 ramps up

3. Peak Concurrency Maksimum (spawn rate 100, load testing 60 detik)
![100 15](https://github.com/freaqbh/fp-tka-b8/assets/123524655/24126797-5b46-4ec1-9fa0-83adb5f07d70)
![100 15 c](https://github.com/freaqbh/fp-tka-b8/assets/123524655/87f440a4-b40f-4b51-a011-20048bcf2d3c)
disini kami melakukan dengan 100 number of use dan sebanyak 15 ramps up

4. Peak Concurrency Maksimum (spawn rate 200, load testing 60 detik)
![200 10](https://github.com/freaqbh/fp-tka-b8/assets/123524655/ea47d6d2-c448-4619-8860-3b8f84403798)
![200 10 c](https://github.com/freaqbh/fp-tka-b8/assets/123524655/884a2461-40e1-46eb-9ed4-a2e81f394c98)
disini kami melakukan dengan 200 number of use dan sebanyak 10 ramps up

5. Peak Concurrency Maksimum (spawn rate 500, load testing 60 detik)
![500 5](https://github.com/freaqbh/fp-tka-b8/assets/123524655/88e2aeaf-a254-4218-9946-2e7eaf2d1ca1)
![500 5 c](https://github.com/freaqbh/fp-tka-b8/assets/123524655/f57776da-567a-4217-af2f-80633a674c7a)
disini kami melakukan dengan 500 number of use dan sebanyak 5 ramps up

# Kesimpulan dan saran
kesimpulan kemampuan menggunakan 3 virtual machine lebih unggul dari pada menggunakan 1 virtual machine bisa dilihat dari setiap ujicoba loadtesting di semua kasus 3 virtual machine lebih unggul hal ini dikarenakan adanya loadbalancing yang dapat mengoptimalkan kemampuan server tersebut.

# Video Penjelasan


https://github.com/freaqbh/fp-tka-b8/assets/123524655/36be7589-be88-4b41-9cfa-0896770230d0

