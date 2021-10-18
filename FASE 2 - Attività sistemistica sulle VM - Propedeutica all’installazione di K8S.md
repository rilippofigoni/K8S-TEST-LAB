### FASE 2 - Attività sistemistica sulle 3 VM - Propedeutica all’installazione di K8S

#### CONNESSIONE SSH AL SERVER KVM e poi connessione al nodo MASTER : k8smaster.local 

```bash
ssh zzfri3b@fp001.ddns.net -p 55022

zzfri3b@fp001.ddns.net's password: 
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.8.0-41-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 updates can be installed immediately.
8 of these updates are security updates.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Web console: https://Orione-ThinkPad-P53:9090/ or https://192.168.1.10:9090/

Last login: Tue Feb  9 15:45:14 2021 from 82.54.117.218
zzfri3b@Orione-ThinkPad-P53:~$ ssh root@192.168.122.141
ssh: connect to host 192.168.122.141 port 22: No route to host
zzfri3b@Orione-ThinkPad-P53:~$ ssh root@192.168.122.185
root@192.168.122.185's password:
Permission denied, please try again.
root@192.168.122.185's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Tue Feb  9 16:04:08 CET 2021 from 192.168.122.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Tue Feb  9 15:48:52 2021 from 192.168.122.1
[root@localhost ~]
```

#### DEFINIZIONE DELL'HOSTNAME

```bash
sudo hostnamectl set-hostname k8smaster.local
sudo hostnamectl set-hostname k8snode01.local
sudo hostnamectl set-hostname k8snode02.local
```

#### CAMBIO TASTIERA LAYOUT ITA

```bash
localectl set-keymap it
```

#### TIME 

```bash
sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-local-rtc true
```

#### CREA UTENTE : (sudoers)

```bash
adduser pippo

passwd pippo

usermod -aG wheel pippo
```

#### CREAZIONE DEL FILE /ETC/HOSTS ####

```bash
sudo cat <<EOF>> /etc/hosts
192.168.122.185         k8smaster.local     k8smaster
192.168.122.8	        k8snode01.local     k8snode1
192.168.122.213         k8snode02.local     k8snode2
EOF
```

#### TOGLIERE LO SWAP

#### ricordarsi di disabilitare lo swap su tutti i nodi!!!!

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```


#### SCHEDE DI RETE

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
lsmod | grep br_netfilter
sudo modprobe br_netfilter
lsmod | grep br_netfilter

[centos@k8s_master ~]$ sudo su -
[root@k8s_master ~] echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```


#### FIREWALL - ELIMINA

```bash
[centos@k8s_master ~]$ sudo setenforce 0
[centos@k8s_master ~]$ sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```


#### IN ALTERNATIVA per essere un po paraculi

```bash
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```
