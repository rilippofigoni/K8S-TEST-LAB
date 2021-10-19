#### CREAZIONE E GESTIONE DEGLI SNAPSHOT KVM ########

```bash
sudo virsh list --title
12   k8smaster02.local   running
13   k8smaster01.local   running
14   k8smaster.local     running
```


#### LISTA SNAPSHOT

```bash
virsh snapshot-list xxxxx
```


#### CREATE A SNAPSHOT #####

```bash
virsh snapshot-create-as --domain k8smaster.local
Domain snapshot 1613568800 created

backup effettuato alle 14:35 del 17/02/2021
```


#### TO REVERT A SNAPSHOT #####

```bash
virsh shutdown --domain freebsd
virsh snapshot-revert --domain freebsd --snapshotname 5Sep2016_S1 --running
```


#### DELETE A SNAPSHOT ####

```bash
virsh snapshot-delete --domain freebsd --snapshotname 5Sep2016_S2
```


####  IN CASO DI ERRORE VM NOT START :Requested operation is not valid: network 'default' is not active

```bash
systemctl restart libvirtd
virsh net-start default
```
