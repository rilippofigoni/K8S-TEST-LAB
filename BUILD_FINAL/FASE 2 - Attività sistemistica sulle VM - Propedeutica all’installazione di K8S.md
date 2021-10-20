### FASE 2 - Attività sistemistica sulle 3 VM - Propedeutica all’installazione di K8S
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

#### ricordarsi di disabilitare poi lo swap su tutti i nodi!!!!

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
