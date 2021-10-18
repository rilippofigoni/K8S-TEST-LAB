
### FASE 3 - Attività d'installazione di Docker e Cluster K8S sul Master Node

#### 3.1 DOCKER INSTALL 

>N.B: (controlla priima di installare l'ultima ver. di containerd dal repo ufficiale - file containerd.io-xx.yy.zz.rpm)

```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.3-3.1.el8.x86_64.rpm

sudo dnf install docker-ce

sudo systemctl enable docker

sudo systemctl start docker
```


#### 3.2 K8S - KUBEADM INSTALL REPO - (controlla esistenza repo configurato) :

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


#### 3.3 K8S - KUBEADM INSTALL

```bash
sudo dnf install kubeadm -y 

sudo systemctl enable kubelet
sudo systemctl start kubelet
```


#### 3.4 INSTALLAZIONE DEL CLUSTER CON PREDISPOSIZIONE POD CNI-FLANNEL di default 


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

kubeadm join 192.168.122.185:6443 --token 661i0n.3c9joy698fqr3tk2 
--discovery-token-ca-cert-hash sha256:7a3cc1f77
71ad2d4853f0b888dd833f77fe7270079f0aecc5254b61bea23cb77
```
> *Comando per aggiunger poi u 2 worker nodes (da COPIARE!!!)* 


#### 3.5 CONTROLLO DELLA RIUSCITA INSTALLAZIONE - (10/15 min dopo)

```bash
[pippo@k8smaster ~]$ kubectl get nodes

NAME              STATUS   ROLES                  AGE   VERSION
k8smaster.local   Ready    control-plane,master   30h   v1.20.2

```

#### 3.6 CONFIGURAZIONE UTENTE E PERMESSI [SOLO PER IL MASTER]	
	
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


#### 3.7 INSTALLAZIONE DEL PLUGIN NETWORK CNI - FLANNEL

```bash

[pippo@k8smaster ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```
#### 3.8 CONTROLLO FINALE DELLO STATUS DEL CLUSTER


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
`