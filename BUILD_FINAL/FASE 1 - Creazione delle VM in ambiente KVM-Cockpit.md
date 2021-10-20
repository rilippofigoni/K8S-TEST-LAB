
 ### FASE 1 - Creazione delle VM in ambiente KVM/Cockpit:

 > seguire come riferimento guida al link : https://it.linux-console.net/?p=1454

Da interfaccia web di cockpit creare **3 VM** con le seguenti carattaristiche :

**8 GB di RAM - 4 CPU - 80 GB HDD e 2 schede di rete ETH** (una lan interna e una esterna)

Hostname : **k8smaster.local - k8snode01.local - k8snode02.local**

poi, una volta ricavato l’IP della prima, connettersi in **SSH**, via KVM, alla VM Master **k8smaster.local** 


#### CONNESSIONE SSH AL SERVER KVM e poi connessione al nodo MASTER : k8smaster.local 

```bash
ssh zzxyxyx@192.168.122.1 -p 55022

zzyxyxyx@xxxxxxx.ddns.net's password: 
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.8.0-41-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 updates can be installed immediately.
8 of these updates are security updates.
To see these additional updates run: apt list --upgradable

ssh root@192.168.122.185

root@192.168.122.185's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Tue Feb  9 16:04:08 CET 2021 from 192.168.122.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Tue Feb  9 15:48:52 2021 from 192.168.122.1
[root@localhost ~]
```



#### CREAZIONE E GESTIONE DEGLI SNAPSHOT KVM ########

#### LISTA SNAPSHOT
```bash
sudo virsh list --title
12   k8smaster02.local   running
13   k8smaster01.local   running
14   k8smaster.local     running
```


#### CREATE A SNAPSHOT #####

```bash
virsh snapshot-create-as --domain k8smaster.local
Domain snapshot 1613568800 created

backup effettuato alle 14:35 del 17/02/2021
```


#### TO REVERT A SNAPSHOT #####

```bash
virsh shutdown --domain k8smaster.local
virsh snapshot-revert --domain k8smaster.local --snapshotname 1613568800 --running
```


#### DELETE A SNAPSHOT ####

```bash
virsh snapshot-delete --domain k8smaster.local --snapshotname 1613568800
```


####  IN CASO DI ERRORE VM NOT START :Requested operation is not valid: network 'default' is not active

```bash
systemctl restart libvirtd
virsh net-start default
```
