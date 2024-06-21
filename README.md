

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
