### FASE 6 - INSTALLAZIONE DI ISTIO per il SERVICE MESH

    

#### FASE 6.1 - DOWNLOAD DI ISTIO

    >Controllare la versione sia aggiornata nei repo ufficiali di github

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

#### FASE 6.2 - CHECK ED INSTALLAZIONE DI ISTIO


```bash

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

#### FASE 6.3 - LABEL DEL NAMESPACE E SIDECAR-PROXY INECTION YES DI DEFAULT 

```bash
[pippo@k8smaster ~]$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```


#### FASE 6.4 -  CONTRALLARE il SAMPLEPOD per vedere se INIETTA IL SIDECAR-PROXY 

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



#### FASE 6.5 - INSTALLAZIONE ADDONS di ISTIO : GRAFANA-PROMETHEUS-KIALI 

>(attento alla versione - aggiornata)

```bash
[pippo@k8s_master ~]$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.8.2/samples/addons/prometheus.yaml

[pippo@k8s_master ~]$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.8.2/samples/addons/grafana.yaml

[pippo@k8s_master ~]$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.8.2/samples/addons/kiali.yaml
```

#### FASE 6.5.1 - KIALI (web-interface-mesh)

```bash
[pippo@k8s_master ~]$ cd addons
[pippo@k8s_master ~]$ kubectl apply -f /home/centos/istio-1.8.2/samples/addons/kiali.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/monitoringdashboards.monitoring.kiali.io created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
service/kiali created
deployment.apps/kiali created

```

```bash
[pippo@k8s_master ~]$ kubectl get pods -A

NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
default                httpbin-74fb669cc6-fxn2d                     2/2     Running   6          23h
istio-system           istio-egressgateway-64d976b9b5-fwbtv         1/1     Running   3          25h
istio-system           istio-ingressgateway-68c86b9fc8-2s5xd        1/1     Running   3          25h
istio-system           istiod-5c986fb85b-fj2xv                      1/1     Running   3          25h
istio-system           kiali-7476977cf9-b5xrd                       1/1     Running   0          47s
kube-system            coredns-74ff55c5b-7pjvq                      1/1     Running   4          44h
kube-system            coredns-74ff55c5b-lcq9j                      1/1     Running   4          44h
kube-system            etcd-k8smaster.local                         1/1     Running   9          44h
kube-system            kube-apiserver-k8smaster.local               1/1     Running   9          44h
kube-system            kube-controller-manager-k8smaster.local      1/1     Running   5          44h
kube-system            kube-flannel-ds-4ctvz                        1/1     Running   6          44h
kube-system            kube-flannel-ds-ktt85                        1/1     Running   6          44h
kube-system            kube-flannel-ds-wbdmn                        1/1     Running   4          44h
kube-system            kube-proxy-2clbg                             1/1     Running   6          44h
kube-system            kube-proxy-jrxhq                             1/1     Running   6          44h
kube-system            kube-proxy-t8dmw                             1/1     Running   4          44h
kube-system            kube-scheduler-k8smaster.local               1/1     Running   5          44h
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-d6f4l   1/1     Running   1          4h11m
kubernetes-dashboard   kubernetes-dashboard-7448ffc97b-pmcfm        1/1     Running   1          4h11m
```
#### FASE 6.5.2 - LANCIO WEB INTERFACE di KIALI

[pippo@k8s_master ~]$ istioctl dashboard kiali
http://localhost:20001/kiali


#### FASE 6.5.3 - INSTALL ADDONS : PROMETHEUS

```bash
[pippo@k8s_master ~]$kubectl apply -f /home/centos/istio-1.8.2/samples/addons/prometheus.yaml
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

```bash

[pippo@k8s_master ~]$ kubectl get pods -A

NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
default                httpbin-74fb669cc6-fxn2d                     2/2     Running   6          24h
istio-system           istio-egressgateway-64d976b9b5-fwbtv         1/1     Running   3          25h
istio-system           istio-ingressgateway-68c86b9fc8-2s5xd        1/1     Running   3          25h
istio-system           istiod-5c986fb85b-fj2xv                      1/1     Running   3          25h
istio-system           kiali-7476977cf9-b5xrd                       1/1     Running   0          21m
istio-system           prometheus-7bfddb8dbf-5xgm7                  2/2     Running   0          71s
kube-system            coredns-74ff55c5b-7pjvq                      1/1     Running   4          44h
kube-system            coredns-74ff55c5b-lcq9j                      1/1     Running   4          44h
kube-system            etcd-k8smaster.local                         1/1     Running   9          44h
kube-system            kube-apiserver-k8smaster.local               1/1     Running   9          44h
kube-system            kube-controller-manager-k8smaster.local      1/1     Running   5          44h
kube-system            kube-flannel-ds-4ctvz                        1/1     Running   6          44h
kube-system            kube-flannel-ds-ktt85                        1/1     Running   6          44h
kube-system            kube-flannel-ds-wbdmn                        1/1     Running   4          44h
kube-system            kube-proxy-2clbg                             1/1     Running   6          44h
kube-system            kube-proxy-jrxhq                             1/1     Running   6          44h
kube-system            kube-proxy-t8dmw                             1/1     Running   4          44h
kube-system            kube-scheduler-k8smaster.local               1/1     Running   5          44h
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-d6f4l   1/1     Running   1          4h31m
kubernetes-dashboard   kubernetes-dashboard-7448ffc97b-pmcfm        1/1     Running   1          4h31m
```
```bash
[pippo@k8s_master ~]$ kubectl -n istio-system get svc prometheus
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
prometheus   ClusterIP   10.100.250.202   <none>        9090/TCP   103s
```



#### FASE 6.5.4 - INSTALL ADDONS : GRAFANA

```bash
[pippo@k8s_master ~]$ kubectl apply -f /home/centos/istio-1.8.2/samples/addons/grafana.yaml
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
```
```bash

[pippo@k8s_master ~]$ kubectl get pods -A

NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
default                httpbin-74fb669cc6-fxn2d                     2/2     Running   6          24h
istio-system           istio-egressgateway-64d976b9b5-fwbtv         1/1     Running   3          25h
istio-system           istio-ingressgateway-68c86b9fc8-2s5xd        1/1     Running   3          25h
istio-system           istiod-5c986fb85b-fj2xv                      1/1     Running   3          25h
istio-system           kiali-7476977cf9-b5xrd                       1/1     Running   0          21m
istio-system           prometheus-7bfddb8dbf-5xgm7                  2/2     Running   0          71s
istio-system           grafana-7bfdf54wf-5xt42                      2/2     Running   0          91s
kube-system            coredns-74ff55c5b-7pjvq                      1/1     Running   4          44h
kube-system            coredns-74ff55c5b-lcq9j                      1/1     Running   4          44h
kube-system            etcd-k8smaster.local                         1/1     Running   9          44h
kube-system            kube-apiserver-k8smaster.local               1/1     Running   9          44h
kube-system            kube-controller-manager-k8smaster.local      1/1     Running   5          44h
kube-system            kube-flannel-ds-4ctvz                        1/1     Running   6          44h
kube-system            kube-flannel-ds-ktt85                        1/1     Running   6          44h
kube-system            kube-flannel-ds-wbdmn                        1/1     Running   4          44h
kube-system            kube-proxy-2clbg                             1/1     Running   6          44h
kube-system            kube-proxy-jrxhq                             1/1     Running   6          44h
kube-system            kube-proxy-t8dmw                             1/1     Running   4          44h
kube-system            kube-scheduler-k8smaster.local               1/1     Running   5          44h
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-d6f4l   1/1     Running   1          4h31m
kubernetes-dashboard   kubernetes-dashboard-7448ffc97b-pmcfm        1/1     Running   1          4h31m
```


```bash
[pippo@k8s_master ~]$ kubectl -n istio-system get svc grafana
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
grafana   ClusterIP   10.103.244.103   <none>        3000/TCP   2m25s
```
#### FASE 6.5.5 - LANCIO WEB INTERFACE GRAFANA&PROMETHEUS

```bash
[pippo@k8s_master ~]$ istioctl dashboard grafana
```
[http://localhost:3000/dashboard/db/istio-mesh-dashboard](http://localhost:3000/dashboard/db/istio-mesh-dashboard)


#### N.B : PER TESTARE IL FUNZIONAMENTO DELLE SERVICE MESH, SEGUIRE LE ISTRUZIONI CONTENUTE NEL FILE LOCALE:

    > ESEMPIO di SERVICE MESH - BOOKINFO WEB APPS and SERVICES.md