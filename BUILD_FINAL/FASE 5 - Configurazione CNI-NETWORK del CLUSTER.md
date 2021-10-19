#### FASE 5 - Configurazione CNI-NETWORK del CLUSTER k8S

>Vista l'esigenza di operare a livello Network con 2 LAN separate, si procede all'installazione del Plugin CNI - MULTUS 
vedi docs : https://github.com/k8snetworkplumbingwg/multus-cni

#### INSTALLAZIONE del POD CNI PER MULTUS (ETH aggiuntiva) #####


```bash
[pippo@k8smaster ~]$ git clone https://github.com/intel/multus-cni.git && cd multus-cni
Cloning into 'multus-cni'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 32216 (delta 0), reused 3 (delta 0), pack-reused 32212
Receiving objects: 100% (32216/32216), 42.56 MiB | 3.50 MiB/s, done.
Resolving deltas: 100% (13394/13394), done.

[pippo@k8smaster]$ cd multus-cni

[pippo@k8smaster multus-cni]$ cat ./images/multus-daemonset.yml | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/network-attachment-definitions.k8s.cni.cncf.io created
clusterrole.rbac.authorization.k8s.io/multus created
clusterrolebinding.rbac.authorization.k8s.io/multus created
serviceaccount/multus created
configmap/multus-cni-config created
daemonset.apps/kube-multus-ds-amd64 created
daemonset.apps/kube-multus-ds-ppc64le created
daemonset.apps/kube-multus-ds-arm64v8 created


[pippo@k8smaster multus-cni]$ kubectl get pod -A

NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-qtjds                   1/1     Running   0          14m
kube-system   coredns-74ff55c5b-wnq4f                   1/1     Running   0          14m
kube-system   etcd-k8smaster.local                      1/1     Running   0          14m
kube-system   kube-apiserver-k8smaster.local            1/1     Running   0          14m
kube-system   kube-controller-manager-k8smaster.local   1/1     Running   0          14m
kube-system   kube-flannel-ds-hvh5m                     1/1     Running   0          10m
kube-system   kube-multus-ds-amd64-wsvgq                1/1     Running   0          59s
kube-system   kube-proxy-c766q                          1/1     Running   0          14m
kube-system   kube-scheduler-k8smaster.local            1/1     Running   0          14m

[pippo@k8smaster multus-cni]$ kubectl get pods --all-namespaces | grep -i multus
kube-system   kube-multus-ds-amd64-wsvgq                1/1     Running   0          116s
```

#### CREAZIONE DELLA 2° INTERFACCIA DI MULTUS 

```bash
cat <<EOF | kubectl create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "type": "macvlan",
      "master": "enp7s0",
      "mode": "bridge",
      "ipam": {
        "type": "host-local",
        "subnet": "192.168.101.0/24",
        "rangeStart": "192.168.101.100",
        "rangeEnd": "192.168.101.216",
        "routes": [
          { "dst": "0.0.0.0/0" }
        ],
        "gateway": "192.168.101.1"
      }
    }'
EOF
networkattachmentdefinition.k8s.cni.cncf.io/macvlan-conf created
```


#### CREAZIONE di UN POD GENERICO : SAMPLEPOD

```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: pippopod
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
  - name: pippopod
    command: ["/bin/ash", "-c", "trap : TERM INT; sleep infinity & wait"]
    image: alpine
EOF
```
#### CONTROLLO DELLA PRESENZA DI UNA SECONDA ETH : net1@eth0

```bash
[pippo@k8smaster ~]$ kubectl exec -it samplepod -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP
    link/ether 4e:aa:c0:42:28:58 brd ff:ff:ff:ff:ff:ff
    inet 10.244.1.2/24 brd 10.244.1.255 scope global eth0
       valid_lft forever preferred_lft forever
4: net1@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 7a:54:a7:bd:fe:8b brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.100/24 brd 192.168.101.255 scope global net1
       valid_lft forever preferred_lft forever
```
