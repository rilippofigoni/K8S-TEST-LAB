#### CONNESSIONE SSH AL SERVER KVM

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

### TIME 

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


#### DOCKER INSTALL - (controlla di installare l'ultima ver. di containerd dal repo ufficiale)

```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.3-3.1.el8.x86_64.rpm

sudo dnf install docker-ce

sudo systemctl enable docker

sudo systemctl start docker
```


#### K8S - KUBEADM - controllo esistenza repo configurato :

```bash
sudo su -
cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```


#### INSTALLARE KUBEADM

```bash
sudo dnf install kubeadm -y 

sudo systemctl enable kubelet
sudo systemctl start kubelet
```


#### INSTALLAZIONE DEL CLUSTER CON PREDISPOSIZIONE POD CNI-FLANNEL di default 
#### [SOLO X PER MASTER] 

```bash
sudo kubeadm init --pod-network-cidr 10.244.0.0/16


Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
```

#### CONTROLLO DELLA RIUSCITA INSTALLAZIONE - (10/15 min dopo)

```bash
[pippo@k8smaster ~]$ kubectl get nodes

NAME              STATUS   ROLES                  AGE   VERSION
k8smaster.local   Ready    control-plane,master   30h   v1.20.2

```

#### CONFIGURAZIONE UTENTE E PERMESSI [SOLO PER IL MASTER]	
	
```bash
[root@k8smaster ~] sudo su - pippo
Last login: Tue Feb  9 16:25:40 CET 2021 on pts/0
[pippo@k8smaster ~]$ sudo mkdir -p $HOME/.kube
[sudo] password for pippo:
[pippo@k8smaster ~]$ sudo mkdir -p $HOME/.kube
[sudo] password for pippo:
[pippo@k8smaster ~]$ sudo mkdir -p $HOME/.kube
[pippo@k8smaster ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[pippo@k8smaster ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
[pippo@k8smaster ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


#### INSTALLAZIONE DEL PLUGIN NETWORK CNI - FLANNEL

```bash
[pippo@k8smaster ~]$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[pippo@k8smaster ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```



```bash
[pippo@k8smaster ~]$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-qtjds                   1/1     Running   0          6m3s
kube-system   coredns-74ff55c5b-wnq4f                   1/1     Running   0          6m3s
kube-system   etcd-k8smaster.local                      1/1     Running   0          6m10s
kube-system   kube-apiserver-k8smaster.local            1/1     Running   0          6m10s
kube-system   kube-controller-manager-k8smaster.local   1/1     Running   0          6m10s
kube-system   kube-flannel-ds-hvh5m                     1/1     Running   0          2m22s
kube-system   kube-proxy-c766q                          1/1     Running   0          6m3s
kube-system   kube-scheduler-k8smaster.local            1/1     Running   0          6m10s
```
#### JOIN NODI SECONDARI AL MASTER (da fare dopo su ogni singolo nodo)

```bash
[pippo@k8snode01 ~]$ sudo kubeadm join 192.168.122.185:6443 --token 661i0n.3c9joy698fqr3tk2 
--discovery-token-ca-cert-hash sha256:7a3cc1f77
71ad2d4853f0b888dd833f77fe7270079f0aecc5254b61bea23cb77

[sudo] password for pippo:
[preflight] Running pre-flight checks
        [WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.3. Latest validated version: 19.03
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

#### CONTROLO DEI NODI AGGIUNTI

```bash
[pippo@k8smaster ~]$ kubectl get nodes

NAME              STATUS   ROLES                  AGE   VERSION
k8smaster.local   Ready    control-plane,master   30h   v1.20.2
k8snode01.local   Ready    <none>                 23h   v1.20.2
k8snode02.local   Ready    <none>                 17m   v1.20.2
```

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

[pippo@k8smaster multus-cni]$ ls
cmd  CODE_OF_CONDUCT.md  CONTRIBUTING.md  deployments  docs  e2e  examples  go.mod  go.sum  hack  images  LICENSE  pkg  README.md  vendor

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

	   
	   
#### INSTALLAZIONE DI K9S (vedi LINK) https://github.com/derailed/k9s (interfaccia console x gestione)

```bash
sudo wget https://github.com/derailed/k9s/releases/download/v0.24.2/k9s_Linux_x86_64.tar.gz

export KUBECONFIG=$HOME/.kube/config

./k9s
```

### INSTALLAZIONE DI ISTIO - SERVICE MESH - COntrollare la versione nei repo ufficiali di github


```bash
[pippo@k8smaster ~]$ curl -L https://istio.io/downloadIstio | sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   102  100   102    0     0    152      0 --:--:-- --:--:-- --:--:--   151
100  4579  100  4579    0     0   5015      0 --:--:-- --:--:-- --:--:--  5015

Downloading istio-1.9.0 from https://github.com/istio/istio/releases/download/1.9.0/istio-1.9.0-linux-amd64.tar.gz ...

Istio 1.9.0 Download Complete!

Istio has been successfully downloaded into the istio-1.9.0 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /home/pippo/istio-1.9.0/bin directory to your environment path variable with:
         export PATH="$PATH:/home/pippo/istio-1.9.0/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck

Need more information? Visit https://istio.io/latest/docs/setup/install/
```

```bash
[pippo@k8smaster ~]$ sudo nano ~/.bash_profile
[pippo@k8smaster ~]$ sudo nano $HOME/.bashrc
[pippo@k8smaster ~]$ sudo nano ~/.bash_profile
[pippo@k8smaster ~]$ export PATH="$PATH:/home/pippo/istio-1.9.0/bin"

[pippo@k8smaster ~]$ istioctl x precheck

Checking the cluster to make sure it is ready for Istio installation...

#1. Kubernetes-api
-----------------------
Can initialize the Kubernetes client.
Can query the Kubernetes API Server.

#2. Kubernetes-version
-----------------------
Istio is compatible with Kubernetes: v1.20.2.

#3. Istio-existence
-----------------------
Istio will be installed in the istio-system namespace.

#4. Kubernetes-setup
-----------------------
Can create necessary Kubernetes configurations: Namespace,ClusterRole,ClusterRoleBinding,CustomResourceDefinition,Role,ServiceAccount,Service,Deployments,ConfigMap.

#5. SideCar-Injector
-----------------------
This Kubernetes cluster supports automatic sidecar injection. To enable automatic sidecar injection see https://istio.io/v1.9/docs/setup/additional-setup/sidecar-injection/#deploying-an-app

-----------------------
Install Pre-Check passed! The cluster is ready for Istio installation.

[pippo@k8smaster ~]$ istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete
```

#### LABEL DEL NAMESPACE E SIDECAR-PROXY INECTION YES DI DEFAULT 

```bash
[pippo@k8smaster ~]$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```


#### CONTRALLARE il SAMPLEPOD per vedere se INIETTA IL SIDECAR-PROXY 
```bash
kubect get pod -A

NAMESPACE      NAME                                      READY   STATUS    RESTARTS   AGE
default        pippopod                                  2/2     Running   0          72s

kubectl describe pod pippopod

 Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  10m    default-scheduler  Successfully assigned default/pippopod to k8snode01.local
  Normal  Pulling    10m    kubelet            Pulling image "docker.io/istio/proxyv2:1.9.0"
  Normal  Pulled     9m34s  kubelet            Successfully pulled image "docker.io/istio/proxyv2:1.9.0" in 38.996097098s
  Normal  Created    9m33s  kubelet            Created container istio-init
  Normal  Started    9m33s  kubelet            Started container istio-init
  Normal  Pulling    9m32s  kubelet            Pulling image "alpine"
  Normal  Pulled     9m30s  kubelet            Successfully pulled image "alpine" in 1.728040883s
  Normal  Created    9m30s  kubelet            Created container pippopod
  Normal  Started    9m30s  kubelet            Started container pippopod
  Normal  Pulling    9m30s  kubelet            Pulling image "docker.io/istio/proxyv2:1.9.0"
  Normal  Pulled     9m28s  kubelet            Successfully pulled image "docker.io/istio/proxyv2:1.9.0" in 1.946954206s
  Normal  Created    9m28s  kubelet            Created container istio-proxy
  Normal  Started    9m28s  kubelet            Started container istio-proxy
  
 ### 2 schede di rete ####

Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "",
                    "interface": "eth0",
                    "ips": [
                        "10.244.1.3"
                    ],
                    "mac": "f2:3a:42:0d:37:a5",
                    "default": true,
                    "dns": {}
                },{
                    "name": "default/macvlan-conf",
                    "interface": "net1",
                    "ips": [
                        "192.168.101.101"
                    ],
                    "mac": "92:be:4c:cb:89:1a",
                    "dns": {}
```



#### INSTALLAZIONE ADDONS di ISTIO : GRAFANA-PROMETHEUS-KIALI (attento la versione!!!) (vedi doc specifici)

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/addons/prometheus.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/addons/grafana.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/addons/kiali.yaml
```


#### SITUAZIONE al 29/05/2021

```bash
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
istio-system           grafana-784d98bb85-brdv9                     1/1     Running   9          22d
istio-system           istio-ingressgateway-f7d5dfc66-fnmgm         1/1     Running   9          22d
istio-system           istiod-6478c75455-sg55k                      1/1     Running   9          22d
istio-system           kiali-7596dd69f5-75t6h                       1/1     Running   9          22d
istio-system           prometheus-58c447874c-gsln5                  2/2     Running   18         22d
kube-system            coredns-74ff55c5b-qtjds                      1/1     Running   43         91d
kube-system            coredns-74ff55c5b-wnq4f                      1/1     Running   43         91d
kube-system            etcd-k8smaster.local                         1/1     Running   44         91d
kube-system            filebeat-nfv46                               1/1     Running   19         50d
kube-system            filebeat-ps94h                               1/1     Running   19         50d
kube-system            filebeat-zq9sv                               1/1     Running   26         50d
kube-system            kube-apiserver-k8smaster.local               1/1     Running   47         91d
kube-system            kube-controller-manager-k8smaster.local      1/1     Running   43         91d
kube-system            kube-flannel-ds-hvh5m                        1/1     Running   48         91d
kube-system            kube-flannel-ds-nqmld                        1/1     Running   38         90d
kube-system            kube-flannel-ds-zv5cg                        1/1     Running   40         89d
kube-system            kube-multus-ds-amd64-bkgfc                   1/1     Running   33         90d
kube-system            kube-multus-ds-amd64-c8bqp                   1/1     Running   33         89d
kube-system            kube-multus-ds-amd64-wsvgq                   1/1     Running   43         91d
kube-system            kube-proxy-8l2nv                             1/1     Running   33         90d
kube-system            kube-proxy-c766q                             1/1     Running   44         91d
kube-system            kube-proxy-qwxmq                             1/1     Running   33         89d
kube-system            kube-scheduler-k8smaster.local               1/1     Running   43         91d
kube-system            metricbeat-7b445f56c5-w2s4v                  1/1     Running   20         50d
kube-system            metricbeat-gt6d5                             1/1     Running   25         50d
kube-system            metricbeat-mfj9d                             1/1     Running   19         50d
kube-system            metricbeat-wshf4                             1/1     Running   19         50d
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-6csxt   1/1     Running   34         82d
kubernetes-dashboard   kubernetes-dashboard-9f9799597-7dx6j         1/1     Running   35         82d
opa                    opa-56c49fd8ff-5nnpn                         2/2     Running   18         22d
```

	


	   
	   