## ISTIO ON k8s :

#### INSTALLAZIONE IMMAGINE DAI REPO UFFICIALI (occhio alla ver.)

```bash
[centos@k8smaster ~]$ curl -L https://istio.io/downloadIstio | sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   102  100   102    0     0    117      0 --:--:-- --:--:-- --:--:--   117
100  4579  100  4579    0     0   3585      0  0:00:01  0:00:01 --:--:--     0

Downloading istio-1.8.2 from https://github.com/istio/istio/releases/download/1.8.2/istio-1.8.2-linux-amd64.tar.gz ...

Istio 1.8.2 Download Complete!

Istio has been successfully downloaded into the istio-1.8.2 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /home/centos/istio-1.8.2/bin directory to your environment path variable with:
         export PATH="$PATH:/home/centos/istio-1.8.2/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck

Need more information? Visit https://istio.io/latest/docs/setup/install/
```

```bash
[centos@k8smaster ~]$  export PATH="$PATH:/home/centos/istio-1.8.2/bin"
[centos@k8smaster ~]$ cd istio-1.8.2/
```

[centos@k8smaster istio-1.8.2]$ ls -ltra
total 24
drwxr-x---   3 centos centos    83 Jan 13 02:06 tools
drwxr-xr-x  19 centos centos   307 Jan 13 02:06 samples
-rw-r--r--   1 centos centos  5866 Jan 13 02:06 README.md
-rw-r-----   1 centos centos   853 Jan 13 02:06 manifest.yaml
drwxr-xr-x   5 centos centos    52 Jan 13 02:06 manifests
-rw-r--r--   1 centos centos 11348 Jan 13 02:06 LICENSE
drwxr-x---   2 centos centos    22 Jan 13 02:06 bin
drwxr-x---   6 centos centos   115 Jan 13 02:06 .
drwx------.  9 centos centos   268 Jan 27 15:02 ..



```bash
[centos@k8smaster istio-1.8.2]$ istioctl x precheck

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
This Kubernetes cluster supports automatic sidecar injection. To enable automatic sidecar injection see https://istio.io/v1.8/docs/setup/additional-setup/sidecar-injection/#deploying-an-app

-----------------------
Install Pre-Check passed! The cluster is ready for Istio installation.
```
#### DOPO il CONTROLLO OK -> INSTALL

```bash
[centos@k8smaster istio-1.8.2]$ istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete
```
#### LABELING del NAMESPACE e ISTIO-INJECTION = ENABLED (per avere in automatico il sidcarproxy su ogni pod)

```bash
[centos@k8smaster istio-1.8.2]$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
[centos@k8smaster istio-1.8.2]$

NAMESPACE      NAME                                      READY   STATUS    RESTARTS   AGE
istio-system   istio-egressgateway-64d976b9b5-fwbtv      1/1     Running   0          4m48s
istio-system   istio-ingressgateway-68c86b9fc8-2s5xd     1/1     Running   0          4m48s
istio-system   istiod-5c986fb85b-fj2xv                   1/1     Running   0          5m20s
kube-system    coredns-74ff55c5b-7pjvq                   1/1     Running   1          19h
kube-system    coredns-74ff55c5b-lcq9j                   1/1     Running   1          19h
kube-system    etcd-k8smaster.local                      1/1     Running   1          19h
kube-system    kube-apiserver-k8smaster.local            1/1     Running   1          19h
kube-system    kube-controller-manager-k8smaster.local   1/1     Running   1          19h
kube-system    kube-flannel-ds-4ctvz                     1/1     Running   3          19h
kube-system    kube-flannel-ds-ktt85                     1/1     Running   3          19h
kube-system    kube-flannel-ds-wbdmn                     1/1     Running   1          19h
kube-system    kube-proxy-2clbg                          1/1     Running   3          19h
kube-system    kube-proxy-jrxhq                          1/1     Running   3          19h
kube-system    kube-proxy-t8dmw                          1/1     Running   1          19h
kube-system    kube-scheduler-k8smaster.local            1/1     Running   1          19h
```