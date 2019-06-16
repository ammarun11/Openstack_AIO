OPENSTAK_All IN One With Packstack
======================

Requirements untuk VM Untuk menjalankan CirrOS Di Openstack
Controller&Compute Node: 2 processor, 4 GB memory, and 20 GB storage


Instruksi Environment Lab
---------------------

0. Saat ada X maka ubah ke nomor 2 Digit NIM terakhir kalian  
1. Buat 1 VM dengan nama di pod-controller Images: CentOS-7-x86_64

#A> Screenshot Create VM di VirtualBox/VMware dll . Beri nama X-os-adm-A.png

Pastikan IP UP, Gateway, DNS Resolver, Hostname Sesuai
---------------------
```
##### Node pod-controller #####
NETWORK ADAPTER VM
1.Adapter 1 
Bridge ke Perangkat yang sedang konek jaringan
2.Adapter 2 
Host-Only Adapter
Interface: 	
IP Address: 10.X.X.10/24
Gateway: 10.X.X.1 << PC mu
Hostname: podX-controller

##### Eksekusi di Node pod-controller #####
ip address
ip route
cat /etc/resolv.conf

---Verifikasi Konektifitas---
ping -c 3 10.X.X.1
ping -c 3 10.X.X.10
ping -c 3 yahoo.com

##### Name Resolution #####
vi /etc/hosts
.....
10.X.X.10 pod-controller

##### Repositori #####
yum -y update
[ ! -d /etc/yum.repos.d.orig ] && cp -vR /etc/yum.repos.d /etc/yum.repos.d.orig
yum -y install centos-release-openstack-queens epel-release
yum repolist
yum -y update

##### NTP #####
yum -y install chrony
systemctl enable chronyd.service
systemctl restart chronyd.service
systemctl status chronyd.service
chronyc sources

##### Firewall #####
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service

##### Networking #####
systemctl disable NetworkManager.service
systemctl stop NetworkManager.service
systemctl status NetworkManager.service
systemctl enable network.service
systemctl restart network.service
systemctl status network.service

##### Paket Utilities #####
yum -y install vim wget screen crudini htop nano

####Untuk tips, agar saat menggunakan sudo kita tidak harus memasukan ####password lagi
####buka sudoers dan hapus tanda [#] 
nano /etc/sudoers
%wheel ALL=(ALL)            NOPASSWD: ALL

#### Selanjutnya, disabled SELinux :
sed -i ‘s/SELINUX=enforcing/SELINUX=disabled/g’ /etc/selinux/config
nano /etc/selinux/config
.....
SELINUX=disabled
```
PACKSTACK_
---------------------
```
##### Instal Paket Packstack #####
yum -y install openstack-packstack python-tools python-setuptools

##### Generate, Sunting, dan Sesuaikan Packstack Answer File #####
packstack --gen-answer-file=X-openstack.txt

nano X-openstack.txt

CONFIG_CEILOMETER_INSTALL=n
CONFIG_AODH_INSTALL=n
CONFIG_MANILA_INSTALL=n
CONFIG_COMPUTE_HOSTS=10.X.X.10

#CONFIG_KEYSTONE_ADMIN_PW=9288844cb55f4c64
CONFIG_KEYSTONE_ADMIN_PW=nev

CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:enp0s3 ## interface yang ada internetnya
CONFIG_NEUTRON_OVS_BRIDGES_COMPUTE=br-ex
CONFIG_PROVISION_DEMO=n

screen -R packstack

packstack --answer-file=X-openstack.txt

### Tunggu proses pakcstacking: ~25-120 menit (Tergantung Kecepatan Internet dan performa laptop anda )
### Keluar screen tanpa mematikan: tekan Ctrl+A kemudian tekan D
### Menampilkan screen yang aktif: screen -ls
### Kembali ke screen packstack: screen -r packstack

#################################
##### Post Deploy Packstack #####
#################################
##### Post Deploy Node pod-controller #####

#00. Metadata DHCP Agent
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
systemctl restart neutron-dhcp-agent
systemctl status neutron-dhcp-agent
sudo systemctl restart httpd  

##### Post Deploy Node podX-compute0 #####
#01. Error: Failed to connect socket to '/var/run/libvirt/virtlogd-sock' Solusi: Aktifkan dan jalankan service virtlogd
systemctl status virtlogd
systemctl enable virtlogd
systemctl restart virtlogd
systemctl status virtlogd

#02. Set Proxy Client
crudini --set /etc/nova/nova.conf vnc vncserver_proxyclient_address 10.X.X.10
systemctl restart openstack-nova-compute
systemctl status openstack-nova-compute

###############
##### BUI #####
###############

#1. Jalankan web browser lalu buka alamat http://IP-podX-controller/dashboard
#2. Login as admin with password on file
cat /root/keystonerc_admin

#B> Screenshot Dashboard Log in OpenStack. Beri nama X-os-adm-B.png

#3. Create images
Unduh image cirros dari PC/Laptop
https://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img

Admin > Compute > Images
Click Create Image
Image Name: cirros0
Source Type: File
File: Browse: cirros-0.4.0-x86_64-disk.img
Format: QCOW2 - QEMU Emulator
Create Image

#4. Create external network
Admin > Network > Networks
Click Create Network
Name: net-ext
Project: admin
Provider Network Type: Flat
Physical Network: extnet
Segmentation ID: [kosongkan]
Admin State: UP
Shared: Checked
External Network: Checked
Submit

#5. Create external subnet
Admin > Network > Networks
Click net-ext > Subnets
Click Create Subnet
Subnet Name: subnet-ext
Network Address: 10.X.X.0/24
IP Version: IPv4
Gateway IP: 10.X.X.1
Disable Gateway: Unchecked
Enable DHCP: Unchecked
Allocation Pools: 10.X.X.100,10.X.X.199
DNS Name Servers: 10.X.X.1

#6. Create internal network & subnet
Project > Network > Networks
Click Create Network
Network Name: net-int0
Admin State: UP
Shared: Unchecked
Create Subnet: Checked
Subnet Name: subnet-int0
Network Address: 192.168.X.0/24
IP Version: IPv4
Gateway IP: 192.168.X.1
Disable Gateway: Unchecked
Enable DHCP: Checked
Allocation Pools: 192.168.X.100,192.168.X.199
DNS Name Servers: 10.X.X.1

#7. Create router
Project > Network > Routers
Click Create Router
Router Name: router0
Admin State: UP
External Network: net-ext

Click router0
Click Interfaces
Click Add Interface
Subnet: subnet-int0

Dari Host pod-controller ping port net-ext router0
ping -c 3 10.X.X.YYY

#C> Screenshot saat sudah sukses ping ke port net-ext router1. Beri nama X-os-adm-C.png

#8. Add SSH key
Project > Compute > Key Pairs
Click Import Key Pair
Key Pair Name: key0
Public Key: [paste SSH public key]

#9. Add security group rules
Project > Network > Security Groups
Click Create Security Group
Name: sg0
Description: My security group 0
Click Manage Rules on sg0

Click Add Rule
Rule: ALL ICMP
Direction: Ingress
Remote: CIDR
CIDR: 0.0.0.0/0

Click Add Rule
Rule: SSH
Remote: CIDR
CIDR: 0.0.0.0/0

#10. Launch instance
Project > Compute > Instances
Click Launch Instance
Instance Name: instance0
Source: Select Boot Source: Image
Create New Volume: No
Image Name: cirros0
Flavor: m1.tiny
Selected networks: net-int0
Security Group: sg0
Key Pair: key0
Launch Instance

#11. Floating IP address
instance0 > Click Drop Down Menu
Click Associate Floating IP
Click + Alocate Floating IP
Pool: net-ext
Click Allocate IP
IP Address: 10.1X.1X.1YY
Port to be associated: instance0 192.168.X.1YY
```

Tasks Will be Updated SOON ~
