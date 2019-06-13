#OPENSTACK AUTO_GASIK/ AIO(AllINOne) Using PACKSTACK

-Bikin VM OS CentOS7
*VM nya bebas mau Virtualbox,VMware,KVM,DLL

Requirements 
6144 MB RAM
20GB Virtual Disk 

*Diatas nya lebih jos dong

-SSH ke CentOS7
Dari client 
# ssh -l $USERVMmu $IPVMmu
# yum -y update

-Lalu install paket bernama centos-release-qemu-ev & mariadb-libs
# yum -y install centos-release-qemu-ev
# yum -y remove mariadb-libs

-Selanjutnya, disabled SELinux :
# sed -i ‘s/SELINUX=enforcing/SELINUX=disabled/g’ /etc/selinux/config

-Bisa dilihat di file /etc/selinux/config. Pastikan sudah disabled :
# cat /etc/selinux/config

- Matikan layanan NetworkManager dan Firewalld :
# systemctl disabled NetworkManager firewalld

- Untuk mencari paket OpenStack bisa dengan perintah : 
# yum search openstack

- Lalu install repository dari OpenStack versi terbaru yaitu Queens :
# yum -y install centos-release-openstack-queens

- Selanjutnya, install PackStack :
# yum -y install openstack-packstack

- Generate File Packstack dengan :
# packstack --gen-answer-file=/home/miss/openstack-packstack.txt

Note :
Letak direktori dan nama file bisa disesuaikan.
Selanjutnya, bisa edit file openstack-packstack tadi,
Cari dan ganti menjadi :

CONFIG_DEFAULT_PASSWORD=rahasia
CONFIG_HEAT_INSTALL=y
CONFIG_MAGNUM_INSTALL=y
CONFIG_KEYSTONE_ADMIN_PW=rahasiakita
CONFIG_KEYSTONE_DEMO_PW=onta

Save&Exit

- Paket Utilities
# yum -y install vim wget screen crudini htop

- Lalu bisa install OpenStack dengan perintah :
# screen -R packstack
# packstack --answer-file=/home/miss/openstack-packstack.txt

### Tunggu proses pakcstacking: ~25-45 menit
### Keluar screen tanpa mematikan: tekan Ctrl+A kemudian tekan D
### Menampilkan screen yang aktif: screen -ls
### Kembali ke screen packstack: screen -r packstack

Tunggu hingga selesai.

Untuk menguji coba, bisa menggunakan web browser, ketik 
http://ipaddr/dashboard 


