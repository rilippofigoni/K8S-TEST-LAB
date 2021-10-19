### INSTALL ISTIO ADDONS :  

#### KIALI (web-interface-mesh)

```bash
[centos@k8s_master ~]$ cd addons
[centos@k8s_master ~]$ ls
extras  grafana.yaml  jaeger.yaml  kiali.yaml  prometheus.yaml  README.md
[centos@k8s_master ~]$ pwd
/home/centos/istio-1.8.2/samples/addons
[centos@k8s_master ~]$ kubectl apply -f /home/centos/istio-1.8.2/samples/addons/kiali.yaml
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
[centos@k8s_master ~]$ kubectl get pods -A

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
#### LANCIO WEB INTERFACE di KIALI

[centos@k8s_master ~]$ istioctl dashboard kiali
http://localhost:20001/kiali


##### INSTALL ADDONS : PROMETHEUS

```bash
[centos@k8s_master ~]$kubectl apply -f /home/centos/istio-1.8.2/samples/addons/prometheus.yaml
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```
```bash

[centos@k8s_master ~]$ kubectl get pods -A

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
[centos@k8s_master ~]$ kubectl -n istio-system get svc prometheus
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
prometheus   ClusterIP   10.100.250.202   <none>        9090/TCP   103s
```



##### INSTALL ADDONS : GRAFANA

```bash
[centos@k8s_master ~]$ kubectl apply -f /home/centos/istio-1.8.2/samples/addons/grafana.yaml
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
```
```bash

[centos@k8s_master ~]$ kubectl get pods -A

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
[centos@k8s_master ~]$ kubectl -n istio-system get svc grafana
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
grafana   ClusterIP   10.103.244.103   <none>        3000/TCP   2m25s
```
#### WEB INTERFACE

```bash
[centos@k8s_master ~]$ istioctl dashboard grafana
```
[http://localhost:3000/dashboard/db/istio-mesh-dashboard](http://localhost:3000/dashboard/db/istio-mesh-dashboard)