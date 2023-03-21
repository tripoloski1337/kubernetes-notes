# Cheatsheet Kubernetes

### Replica set
Notes:
- replication controller tidak disarankan 
- replicaset label selector lebih expressive dibandingkan replication controller
- replica sey punya match expression, bisa nambahin expressi, in, NotIn, Exists, NotExists

### Daemon set
Notes:
- use case: untuk aplikasi monitoring Node
- use case: untuk aplikasi mengambil log di Node
- kalo pake daemon set nanti setiap node akan di Install 1 pod

### Job
Notes:
- hanya menjalankan pod untuk sekali saja dan akan mati ketika process sudah selesai
- aplikasi untuk backup/restore db
- aplikasi untuk import atau export data
- aplikasi untuk menjalankan proses batch

### Cron Job Kubernetes
Notes:
- jalanin job sesuai dengan waktu yang udah di define sebelumnya
- cron job bisa berulang sesuai jadwal yang di inginkan
- Aplikasi untuk membuat laporan harian
- Aplikasi untuk backup data berkala
- aplikasi untuk ngirim data tagihan bulanan ke pihak lain
- aplikasi tarik dana pinjaman jatoh tempo

### Node Selector
Notes:
- untuk set node dengan spesifikasi berbeda dari node biasanya
- misal node yang GPU atau hardisk SSD
- bisa jalan di job, cronjob, replicaset, daemonset, pod

### Service
Notes:
- resource untuk gerbang akses 1 atau lebih pod
- service punya IP dan port yang gaakan berubah selama service itu ada
- client bisa akses service, otomatis akan di forward ke pod yang ada di belakang service
- untuk High Availablity, pod bisa nambah, kurang atau berpindah 
- service menggunakan label selector untuk mengetahui pod mana yang ada dibelakang service tersebut

#### Cara akses service
Notes:
- via IP, bisa cek env tiap machine
- via local DNS, contoh nama-service.nama-namespace.svc.cluster.local

### External Service
Notes:
- biasanya service digunakan sebagai gateway untuk internal pod
- service juga bisa digunakan untuk gateway apluikasi external yang berada diluar kubernetes cluster
- jadi ada satu gerbang untuk akses keluar cluster

### Mengexpose Service kubernetes
Notes:
- untuk expose service keluar
- agar aplikasi dari luar kubernetes bisa mengakses Pod yang berada di belakang service

#### Cara expose service
- Bisa pakai nodeport, sehingga node akan membuka port yang akan meneruskan request ke Service yang dituju
- Dengan menggunakan loadbalancer, sehingga service bisa di akses via LoadBalancer, dan load balancer akan meneruskan request ke nodePort dan dilanjutkan ke service
- menggunakan ingress, dimana ingress adalah resource yang memang ditujukan untuk menghekspos service, namun hanya beroperasi di level HTTP

### Multi Container Pod
- kita bisa create lebih dari 1 container dalam 1 pod
- akses container melalui ip pod yang sama namun berbeda port

### Volume
- file di dalem container itu ga permanen, akan terhapus kalo Pod atau container di hapus
- volume adalah direktori yang dapat di akses oleh container - container di Pod
- emptyDir: direktori sederhana yang kosong
- hostPath: digunakan untuk sharing direktori di node ke pod
- gitRepo: direktori yang dibuat pertama kali dengan mengclone git repository
- nfs: shaering network file system

### Sharing Volume
- volume di pod bisa di sahre ke beberapa container yang ada dalam pod
- bisa sharing antar container, misal container 1 membuat file, container 2 memproses file

### Environment Variable
- cocok konfigurasi, kubernetes support variable environment untuk Pod
- cocok untuk config db, config app, dan lain2

### Config Map
- config map isinya key-value
- aplikasi tidak perlu membaca konfigurasi langsung ke configMap, melainkan kubernetes akan mengirimkan config di configMap ke dalam variable env di container
- Data yang ada dalam configMap dianggap tidak sensitive

### Secret
- cocok untuk username, password, apikey, secretkey, dll
- sama seperti configmap, berisikan key-value
- khusus untuk data sensitive
- secret disimpan di memory di node, tidak disimpan di physical storage
- secret di simpan dengan cara di encrypt, sehingga menjadi lebih aman
- di stored di master node sendiri (etcd) 

### Downward API
- untuk konfigurasi dinamis, seperti informasi pod dan node
- bisa di pake untuk ambil info seputar pod dan node via env variable
- downward API bukan RESTful API, cuma cara dapetin informasi seputar pod dan node

### Manage Kubernetes Objects
- perintah untuk manage: update, lihat, remove
- 

#### Imperative Management

    kubectl create -f namafile.yaml             # create kubernetes object
    kubectl replace -f namafile.yaml            # update kubernetes object
    kubectl get -f namafile.yaml -o yaml/json   # melihat kubernetes object
    kubectl delete -f namafile.yaml             # menghapus kubernetest object

#### Declarative Management

    kubectl apply -f namafile.yaml              # membuat/mengupdate kubernetes object
    
- declarative management, file config akan di simpan dalam annotations object
- sangat bagus untuk object deployment
- sekarang kebanyakan pake declarative management

### Deployment
- bagian diatas replicaset, akan memanage pod dan replicaset otomatis
- mudah untuk di update
- sama seperti replica set

### Update Deployment
- gunakan perintah apply lagi
- saat deployment di eksekusi, secara otomatis deployment akan membuat replicaset baru lalu menyalakan pod baru, setelah pod siapa, deployment akan menghapus pod lama secara otomatis
- ini membuat proses update berjalan seamless, dan tidak terjadi downtime

### Rollback Deployment
- kalo update ada masalah, bisa rollback ke previous version
- bisa pake cara manual dengan menggunakan deployment baru, versi aplikasinya di set ke versi sebelumnya
- ada cara mudahnya kita bisa pake fitur rollout kubernetes untuk rollback deployment ke versi deployment sebelumnya

#### Kubernetes Rollout

    kubectl rollout history object name         # melihat history rollout
    kubcetl rollout pasue object name           # menandai sebagai pause
    kubectl rollout resume object name          # resume pause
    kubectl rollout restart object name         # merestart rollout
    kubectl rollout status object name          # melihat status rollout
    kubectl rollout undo object name            # undo ke rollout sebelumnya

Rollback deployment

    kubectl rollout undo deployment namadeployment

### Presistent Volume
mirip volume, cuma cara kerjanya aja yang beda
Jenis2 presistence volume: 
- HostPath, berkas disimpan di Node, jangan pake buat production, hanya untuk testing
- GCEPresistentDisk, Google Cloud Presistence Disk
- AWSElasticBlockStore, amazon web service presistence disk
- AzureFile/AzureDisk, Microsoft Azure Presistence Disk

#### Tahapan Presistent Volume
1. Membuat Presistent Volume
2. Membuat Presisten Volume Claim
3. Menambahkan Presistent Volume Claim ke Pod

### StatefulSet
- stateless artinya tidak menyimpan data
- db itu stateful karna nyimpan data dan gabisa sembarangan di hapus
- statefulset bikinnya sama kaya ReplicaSet
- statefulset memiliki kemampuan untuk menambahkan VolumeClaiTemplate
- cocok untuk aplikasi database 

### Horizontal Pod Autoscaler
- ada 2 jeniL vertical scaling dan horizontal scaling

#### Vertical Scaling
- cara scalingnya dengan upgrade computational resource aplikasi
- ex: 1 cpu jadi 2 cpu, 1gb mem jadi 2gb mem
- masalahnya vertical adalah akan ada batasnya karna pod di k8s tidak bisa pake resource lebih dari resource node yang ada

#### Horizontal Scaling
- akan membuat pod baru agar beban bisa di dsitribusikan ke pod itu
- harusnya scalability paling oke, karna ga perlu upgrade node dengan resource yang lebih tinggi
- kemampuan untuk otomasi scaling secara horizontal dengan cara menambah pod baru dan menurunkan secara otomatis jika diperlukan
- HPA = horizontal pod autoscaler
- bisa membuat HPA dan menghapus HPA di kubernetes
- HPA bekerja dengan cara mendengarkan data metrics dari setiap pod dan jika sudah mencapai batas tertentu HPA akan melakukan auto scaling (baik menaikan atay menurunkan jumlah pod)


### Running yaml file 

    kubectl create -f ./namafile.yaml

### Get list Pod

    kubectl get pod

### Get list pod with label

    kubectl get pod --show-labels

### Get pod list by specific name space

    kubectl get pod --namespace <name space>

### Get namespace info

    kubectl describe namespace <name space>

### Get list Namespace

    kubectl get namespace

### Get list replication controller

    kubectl get replicationcontroller

### Search Pod by label conditions

    kubectl get pod --show-labels
    kubectl get pods -l team
    kubectl get pods -l team=itsec
    kubectl get pods -l environemtn
    kubectl get pods -l team=rnd
    kubectl get pods -l team=itsec
    kubectl get pods -l "
    kubectl get pods -l '!team'
    kubectl get pods -l 'environmen in (production, qa, development)'
    kubectl get pods -l 'environment in (production, qa, development)'
    kubectl get pods -l 'environment not in (production, qa, development)'
    kubectl get pods -l environtment,team
    kubectl get pods -l environment,team
    kubectl get pods -l environment,team=itsec

### Get Pod Details
    
    kubectl describe pod <pod name>

### Add Pod Annotations

    kubectl annotate pod <pod name> <annotation key>:<annotation value>

### Port forwarding a pod

    kubectl port-forward <pod name> 1337:80

### Remove rc without removing pod

    kubectl delete rc name-rc --cascade=false

### Remove all Pod

    kubectl delete pod --all

### Remove a Pod

    kubectl delete pod namepod

### Remove a namespcae

    kubectl delete namespace <namespace>

### Remove all pod on specific namespace

    kubectl delete pod --all --namespace <namespace>

### Remove a Pod with specific label

    kubectl delete pod -l labelKey=labelValue
