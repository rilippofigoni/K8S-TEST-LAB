
### ESEMPIO di SERVICE MESH (BOOKINFO WEB APPS and SERVICES)


  > https://github.com/istio/istio/blob/master/samples/bookinfo/platform/kube/bookinfo.yaml


```bash
[pippo@k8smaster kube]$ kubectl apply -f /home/pippo/istio-1.8.2/samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```
#### CONTROLLO DEI SERVIZI E RELATIVI PODS CREATI :

```bash
[pippo@k8smaster kube]$ kubectl get services
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.104.190.30   <none>        9080/TCP   4m15s
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    20h
productpage   ClusterIP   10.102.78.140   <none>        9080/TCP   4m12s
ratings       ClusterIP   10.108.1.161    <none>        9080/TCP   4m14s
reviews       ClusterIP   10.97.148.126   <none>        9080/TCP   4m14s
[pippo@k8smaster kube]$ kubectl get pods -A
NAMESPACE      NAME                                      READY   STATUS    RESTARTS   AGE
default        details-v1-79c697d759-rgq2l               2/2     Running   0          4m42s
default        productpage-v1-65576bb7bf-kvcnf           2/2     Running   0          4m39s
default        ratings-v1-7d99676f7f-jk9zw               2/2     Running   0          4m42s
default        reviews-v1-987d495c-k4ltf                 2/2     Running   0          4m41s
default        reviews-v2-6c5bf657cf-b5zd8               2/2     Running   0          4m41s
default        reviews-v3-5f7b9f4f77-fggxw               2/2     Running   0          4m40s
istio-system   istio-egressgateway-64d976b9b5-fwbtv      1/1     Running   0          68m
istio-system   istio-ingressgateway-68c86b9fc8-2s5xd     1/1     Running   0          68m
istio-system   istiod-5c986fb85b-fj2xv                   1/1     Running   0          68m
kube-system    coredns-74ff55c5b-7pjvq                   1/1     Running   1          20h
kube-system    coredns-74ff55c5b-lcq9j                   1/1     Running   1          20h
kube-system    etcd-k8smaster.local                      1/1     Running   1          20h
kube-system    kube-apiserver-k8smaster.local            1/1     Running   1          20h
kube-system    kube-controller-manager-k8smaster.local   1/1     Running   1          20h
kube-system    kube-flannel-ds-4ctvz                     1/1     Running   3          20h
kube-system    kube-flannel-ds-ktt85                     1/1     Running   3          20h
kube-system    kube-flannel-ds-wbdmn                     1/1     Running   1          20h
kube-system    kube-proxy-2clbg                          1/1     Running   3          20h
kube-system    kube-proxy-jrxhq                          1/1     Running   3          20h
kube-system    kube-proxy-t8dmw                          1/1     Running   1          20h
kube-system    kube-scheduler-k8smaster.local            1/1     Running   1          20h
[pippo@k8smaster kube]$
```
#### ESECUZIONE DEI VARI ESEMPI DI SERVIZIO 

```bash
[pippo@k8smaster kube]$ kubectl exec "$(kubectl get pod -l app=ratings -o
jsonpath='{.items[0].metadata.name}')" -c ratings -- curl productpage:9080/productpage | grep -o 

"<title>.*</title>"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4183  100  4183    0     0   1825      0  0:00:02  0:00:02 --:--:--  1825
<title>Simple Bookstore App</title>
```

#### CREAZIONE DELL'INGRESS 
```bash
[pippo@k8smaster kube]$ kubectl apply -f 
/home/pippo/istio-1.8.2/samples/bookinfo/platform/kube/bookinfo-ingress.yaml

Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/gateway created

```
```bash
[pippo@k8smaster kube]$ kubectl get gateway
No resources found in default namespace.
[pippo@k8smaster kube]$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
      AGE
istio-ingressgateway   LoadBalancer   10.105.168.66   <pending>     15021:31916/TCP,80:30356/TCP,443:31490/TCP,31400:31999/TCP,15443:31057/TCP   87m
```

```bash
[pippo@k8smaster kube]$  kubectl apply -f - <<EOF
> apiVersion: networking.istio.io/v1alpha3
> kind: Gateway
> metadata:
>   name: httpbin-gateway
> spec:
>   selector:
>     istio: ingressgateway # use Istio default gateway implementation
>   servers:
>   - port:
>       number: 80
>       name: http
>       protocol: HTTP
>     hosts:
>     - "httpbin.example.com"
> EOF
gateway.networking.istio.io/httpbin-gateway created
[pippo@k8smaster kube]$ kubectl apply -f - <<EOF
> apiVersion: networking.istio.io/v1alpha3
> kind: VirtualService
> metadata:
>   name: httpbin
> spec:
>   hosts:
>   - "httpbin.example.com"
>   gateways:
>   - httpbin-gateway
>   http:
>   - match:
>     - uri:
>         prefix: /status
>     - uri:
>         prefix: /delay
>     route:
>     - destination:
>         port:
>           number: 8000
>         host: httpbin
> EOF
virtualservice.networking.istio.io/httpbin created
```

```bash
[pippo@k8smaster kube]$ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/status/200"
```

```bash
[pippo@k8smaster kube]$ cd kube/

[pippo@k8smaster kube]$ ls
bookinfo-certificate.yaml  bookinfo-ingress.yaml              bookinfo-ratings-v2-mysql.yaml  bookinfo.yaml
bookinfo-db.yaml           bookinfo-mysql.yaml                bookinfo-ratings-v2.yaml        cleanup.sh
bookinfo-details-v2.yaml   bookinfo-ratings-discovery.yaml    bookinfo-ratings.yaml           productpage-nodeport.yaml
bookinfo-details.yaml      bookinfo-ratings-v2-mysql-vm.yaml  bookinfo-reviews-v2.yaml        README.md
[pippo@k8smaster kube]$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
error: the path "samples/bookinfo/networking/bookinfo-gateway.yaml" does not exist
[pippo@k8smaster kube]$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml^C
[pippo@k8smaster kube]$ kubectl apply -f /home/pippo/istio-1.8.2/samples/bookinfo/platform/kube/bookinfo.yaml ^C
[pippo@k8smaster kube]$ kubectl apply -f /home/pippo/istio-1.8.2/samples/bookinfo/platform/kube/bookinfo-ingress.yaml
Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/gateway created
[pippo@k8smaster kube]$ kubectl get gateway
No resources found in default namespace.
[pippo@k8smaster kube]$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
      AGE
istio-ingressgateway   LoadBalancer   10.105.168.66   <pending>     15021:31916/TCP,80:30356/TCP,443:31490/TCP,31400:31999/TCP,15443:31057/TCP   83m
```

```bash
[pippo@k8smaster kube]$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP
}')
```

```bash
[pippo@k8smaster kube]$ kubectl get pods -A
NAMESPACE      NAME                                      READY   STATUS    RESTARTS   AGE
default        details-v1-79c697d759-rgq2l               2/2     Running   0          20m
default        productpage-v1-65576bb7bf-kvcnf           2/2     Running   0          20m
default        ratings-v1-7d99676f7f-jk9zw               2/2     Running   0          20m
default        reviews-v1-987d495c-k4ltf                 2/2     Running   0          20m
default        reviews-v2-6c5bf657cf-b5zd8               2/2     Running   0          20m
default        reviews-v3-5f7b9f4f77-fggxw               2/2     Running   0          20m
istio-system   istio-egressgateway-64d976b9b5-fwbtv      1/1     Running   0          84m
istio-system   istio-ingressgateway-68c86b9fc8-2s5xd     1/1     Running   0          84m
istio-system   istiod-5c986fb85b-fj2xv                   1/1     Running   0          84m
kube-system    coredns-74ff55c5b-7pjvq                   1/1     Running   1          20h
kube-system    coredns-74ff55c5b-lcq9j                   1/1     Running   1          20h
kube-system    etcd-k8smaster.local                      1/1     Running   1          20h
kube-system    kube-apiserver-k8smaster.local            1/1     Running   1          20h
kube-system    kube-controller-manager-k8smaster.local   1/1     Running   1          20h
kube-system    kube-flannel-ds-4ctvz                     1/1     Running   3          20h
kube-system    kube-flannel-ds-ktt85                     1/1     Running   3          20h
kube-system    kube-flannel-ds-wbdmn                     1/1     Running   1          20h
kube-system    kube-proxy-2clbg                          1/1     Running   3          20h
kube-system    kube-proxy-jrxhq                          1/1     Running   3          20h
kube-system    kube-proxy-t8dmw                          1/1     Running   1          20h
kube-system    kube-scheduler-k8smaster.local            1/1     Running   1          20h
```

```bash
[pippo@k8smaster kube]$ kubectl get gateway
No resources found in default namespace.
[pippo@k8smaster kube]$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
      AGE
istio-ingressgateway   LoadBalancer   10.105.168.66   <pending>     15021:31916/TCP,80:30356/TCP,443:31490/TCP,31400:31999/TCP,15443:31057/TCP   87m
```

```bash
[pippo@k8smaster kube]$  kubectl apply -f - <<EOF
> apiVersion: networking.istio.io/v1alpha3
> kind: Gateway
> metadata:
>   name: httpbin-gateway
> spec:
>   selector:
>     istio: ingressgateway # use Istio default gateway implementation
>   servers:
>   - port:
>       number: 80
>       name: http
>       protocol: HTTP
>     hosts:
>     - "httpbin.example.com"
> EOF
gateway.networking.istio.io/httpbin-gateway created
[pippo@k8smaster kube]$ kubectl apply -f - <<EOF
> apiVersion: networking.istio.io/v1alpha3
> kind: VirtualService
> metadata:
>   name: httpbin
> spec:
>   hosts:
>   - "httpbin.example.com"
>   gateways:
>   - httpbin-gateway
>   http:
>   - match:
>     - uri:
>         prefix: /status
>     - uri:
>         prefix: /delay
>     route:
>     - destination:
>         port:
>           number: 8000
>         host: httpbin
> EOF
virtualservice.networking.istio.io/httpbin created
```

```bash
[pippo@k8smaster kube]$ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/status/200"
```


```bash
[pippo@k8smaster kube]$ [pippo@k8smaster kube]$ kubectl get gateway
ound in -bash: [pippo@k8smaster: command not found
default namespac[pippo@k8smaster kube]$ No resources found in default namespace.
-bash: No: command not found
[pippo@k8smaster kube]$ [pippo@k8smaster kube]$ kubectl get svc istio-ingressgateway -n istio-system
ME                   -bash: [pippo@k8smaster: command not found
TYPE[pippo@k8smaster kube]$ NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
-bash: syntax error near unexpected token `('
[pippo@k8smaster kube]$       AGE
-bash: AGE: command not found
eway   LoadBalancer   10.105.168.66   [pippo@k8smaster kube]$ istio-ingressgateway   LoadBalancer   10.105.168.66   <pending>     15021:3100:31999/TCP,15443:31057/TCP   87m,3140
-bash: pending: No such file or directory
```

```bash
[pippo@k8smaster kube]$ [pippo@k8smaster kube]$  kubectl apply -f - <<EOF
> > apiVersion: networking.istio.io/v1alpha3
> > kind: Gateway
> > metadata:
> >   name: httpbin-gateway
> > spec:
> >   selector:
> >     istio: ingressgateway # use Istio default gateway implementation
> >   servers:
> >   - port:
> >       number: 80
> >       name: http
> >       protocol: HTTP
> >     hosts:
> >     - "httpbin.example.com"
> > EOF
> gateway.networking.istio.io/httpbin-gateway created
> [pippo@k8smaster kube]$ kubectl apply -f - <<EOF
> > apiVersion: networking.istio.io/v1alpha3
> > kind: VirtualService
> > metadata:
> >   name: httpbin
> > spec:
> >   hosts:
> >   - "httpbin.example.com"
> >   gateways:
> >   - httpbin-gateway
> >   http:
> >   - match:
> >     - uri:
> >         prefix: /status
> >     - uri:
> >         prefix: /delay
> >     route:
> >     - destination:
> >         port:
> >           number: 8000
> >         host: httpbin
> > EOF
> virtualservice.networking.istio.io/httpbin created
```

```bash
> [pippo@k8smaster kube]$ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/status/200"
```

```bash
[pippo@k8smaster kube]$ cd kube/

[pippo@k8smaster kube]$ ls

bookinfo-certificate.yaml  bookinfo-ingress.yaml              bookinfo-ratings-v2-mysql.yaml  bookinfo.yaml
bookinfo-db.yaml           bookinfo-mysql.yaml                bookinfo-ratings-v2.yaml        cleanup.sh
bookinfo-details-v2.yaml   bookinfo-ratings-discovery.yaml    bookinfo-ratings.yaml           productpage-nodeport.yaml
bookinfo-details.yaml      bookinfo-ratings-v2-mysql-vm.yaml  bookinfo-reviews-v2.yaml        README.md
```



```bash
[pippo@k8smaster httpbin]$ kubectl apply -f /home/pippo/istio-1.8.2/samples/httpbin/httpbin.yaml
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created
[pippo@k8smaster httpbin]$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
      AGE
istio-ingressgateway   LoadBalancer   10.105.168.66   <pending>     15021:31916/TCP,80:30356/TCP,443:31490/TCP,31400:31999/TCP,15443:31057/TCP   95m
```

```bash
[pippo@k8smaster httpbin]$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingr[pippo@k8smaster httpbin]$ export SECURE_INGRESS_PORT=$(kubectl [?(@.name=="https")].nodePort}')o-ingressgateway -o jsonpath='{.spec.ports[
rt TCP_INGRESS_PORT=$(kubectl -n istio-system[pippo@k8smaster httpbin]$ export TCP_INGRESS_PORT=$(kubectl -n istio-system get service isti@.name=="tcp")].nodePort}')h='{.spec.ports[?(@
```

```bash
[pippo@k8smaster httpbin]$ echo $INGRESS_PORT
30356
```

```bash
[pippo@k8smaster httpbin]$  export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
[pippo@k8smaster httpbin]$ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/status/200"
HTTP/1.1 200 OK
server: istio-envoy
date: Wed, 27 Jan 2021 15:45:33 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 0
x-envoy-upstream-service-time: 64
```


```bash
[pippo@k8smaster httpbin]$ kubectl get gateway
NAME              AGE
httpbin-gateway   9m32s
[pippo@k8smaster httpbin]$
[pippo@k8smaster httpbin]$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
[pippo@k8smaster httpbin]$ echo $GATEWAY_URL
192.168.146.139:30356
```

[pippo@k8smaster httpbin]$

```bash
[pippo@k8smaster samples]$ curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

```bash
[pippo@k8smaster samples]$ cd bookinfo/networking/
[pippo@k8smaster networking]$ ls
bookinfo-gateway.yaml            virtual-service-all-v1.yaml              virtual-service-reviews-50-v3.yaml
certmanager-gateway.yaml         virtual-service-details-v2.yaml          virtual-service-reviews-80-20.yaml
destination-rule-all-mtls.yaml   virtual-service-ratings-db.yaml          virtual-service-reviews-90-10.yaml
destination-rule-all.yaml        virtual-service-ratings-mysql-vm.yaml    virtual-service-reviews-jason-v2-v3.yaml
destination-rule-reviews.yaml    virtual-service-ratings-mysql.yaml       virtual-service-reviews-test-v2.yaml
egress-rule-google-apis.yaml     virtual-service-ratings-test-abort.yaml  virtual-service-reviews-v2-v3.yaml
fault-injection-details-v1.yaml  virtual-service-ratings-test-delay.yaml  virtual-service-reviews-v3.yaml
[pippo@k8smaster networking]$ pwd
/home/pippo/istio-1.8.2/samples/bookinfo/networking
```

```bash
[pippo@k8smaster networking]$ kubectl apply -f /home/pippo/istio-1.8.2/samples/bookinfo/networking/destination-rule-all.yaml
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
[pippo@k8smaster networking]$ kubectl get destinationrules -o yaml
apiVersion: v1
items:
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"details","namespace":"default"},"spec":{"host":"details","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"}]}}
    creationTimestamp: "2021-01-27T15:52:51Z"
    generation: 1
    managedFields:
    - apiVersion: networking.istio.io/v1alpha3
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
        f:spec:
          .: {}
          f:host: {}
          f:subsets: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2021-01-27T15:52:51Z"
    name: details
    namespace: default
    resourceVersion: "21160"
    uid: 05d590b0-8e61-42c6-bd68-3a4f8253f698
  spec:
    host: details
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"productpage","namespace":"default"},"spec":{"host":"productpage","subsets":[{"labels":{"version":"v1"},"name":"v1"}]}}
    creationTimestamp: "2021-01-27T15:52:51Z"
    generation: 1
    managedFields:
    - apiVersion: networking.istio.io/v1alpha3
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
        f:spec:
          .: {}
          f:host: {}
          f:subsets: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2021-01-27T15:52:51Z"
    name: productpage
    namespace: default
    resourceVersion: "21157"
    uid: adac3618-d748-4056-8691-8f577a085abd
  spec:
    host: productpage
    subsets:
    - labels:
        version: v1
      name: v1
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"host":"ratings","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v2-mysql"},"name":"v2-mysql"},{"labels":{"version":"v2-mysql-vm"},"name":"v2-mysql-vm"}]}}
    creationTimestamp: "2021-01-27T15:52:51Z"
    generation: 1
    managedFields:
    - apiVersion: networking.istio.io/v1alpha3
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
        f:spec:
          .: {}
          f:host: {}
          f:subsets: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2021-01-27T15:52:51Z"
    name: ratings
    namespace: default
    resourceVersion: "21159"
    uid: e4594804-b040-438e-a6d9-73ab564961e9
  spec:
    host: ratings
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v2-mysql
      name: v2-mysql
    - labels:
        version: v2-mysql-vm
      name: v2-mysql-vm
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"host":"reviews","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v3"},"name":"v3"}]}}
    creationTimestamp: "2021-01-27T15:52:51Z"
    generation: 1
    managedFields:
    - apiVersion: networking.istio.io/v1alpha3
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
        f:spec:
          .: {}
          f:host: {}
          f:subsets: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2021-01-27T15:52:51Z"
    name: reviews
    namespace: default
    resourceVersion: "21158"
    uid: ac491db3-021d-4acb-ad0e-076d497eb877
  spec:
    host: reviews
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v3
      name: v3
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```
#### CLEANUP

```bash
[pippo@k8smaster networking]$ cd ..
[pippo@k8smaster bookinfo]$ cd ..
[pippo@k8smaster samples]$ cd bookinfo/platform/kube/
[pippo@k8smaster kube]$ pwd
/home/pippo/istio-1.8.2/samples/bookinfo/platform/kube

[pippo@k8smaster kube]$ ./cleanup.sh
namespace ? [default]
using NAMESPACE=default
I0127 17:01:02.015582   65111 request.go:655] Throttling request took 1.135764407s, request: GET:https://192.168.147.140:6443/apis/networking.k8s.io/v1beta1?timeout=32s
destinationrule.networking.istio.io "details" deleted
I0127 17:01:08.487979   65179 request.go:655] Throttling request took 1.101970435s, request: GET:https://192.168.147.140:6443/apis/apiextensions.k8s.io/v1beta1?timeout=32s
destinationrule.networking.istio.io "productpage" deleted
I0127 17:01:14.622209   65220 request.go:655] Throttling request took 1.062864748s, request: GET:https://192.168.147.140:6443/apis/networking.k8s.io/v1beta1?timeout=32s
destinationrule.networking.istio.io "ratings" deleted
I0127 17:01:21.122153   65284 request.go:655] Throttling request took 1.165552639s, request: GET:https://192.168.147.140:6443/apis/scheduling.k8s.io/v1?timeout=32s
destinationrule.networking.istio.io "reviews" deleted
I0127 17:01:27.586786   65345 request.go:655] Throttling request took 1.026277521s, request: GET:https://192.168.147.140:6443/apis/networking.istio.io/v1alpha3?timeout=32s
virtualservice.networking.istio.io "httpbin" deleted
I0127 17:01:34.480226   65395 request.go:655] Throttling request took 1.147352724s, request: GET:https://192.168.147.140:6443/apis/autoscaling/v2beta2?timeout=32s
gateway.networking.istio.io "httpbin-gateway" deleted
Application cleanup may take up to one minute
service "details" deleted
serviceaccount "bookinfo-details" deleted
deployment.apps "details-v1" deleted
service "ratings" deleted
serviceaccount "bookinfo-ratings" deleted
deployment.apps "ratings-v1" deleted
service "reviews" deleted
serviceaccount "bookinfo-reviews" deleted
deployment.apps "reviews-v1" deleted
deployment.apps "reviews-v2" deleted
deployment.apps "reviews-v3" deleted
service "productpage" deleted
serviceaccount "bookinfo-productpage" deleted
deployment.apps "productpage-v1" deleted
Application cleanup successful
```

#### LOG del PROXY


```bash
❯ kubectl logs httpbin-74fb669cc6-fxn2d istio-proxy
2021-01-28T13:22:50.521802Z     info    FLAG: --concurrency="2"
2021-01-28T13:22:50.521837Z     info    FLAG: --domain="default.svc.cluster.local"
2021-01-28T13:22:50.521853Z     info    FLAG: --help="false"
2021-01-28T13:22:50.521858Z     info    FLAG: --log_as_json="false"
2021-01-28T13:22:50.521863Z     info    FLAG: --log_caller=""
2021-01-28T13:22:50.521872Z     info    FLAG: --log_output_level="default:info"
2021-01-28T13:22:50.521876Z     info    FLAG: --log_rotate=""
2021-01-28T13:22:50.521881Z     info    FLAG: --log_rotate_max_age="30"
2021-01-28T13:22:50.521898Z     info    FLAG: --log_rotate_max_backups="1000"
2021-01-28T13:22:50.521906Z     info    FLAG: --log_rotate_max_size="104857600"
2021-01-28T13:22:50.521917Z     info    FLAG: --log_stacktrace_level="default:none"
2021-01-28T13:22:50.521935Z     info    FLAG: --log_target="[stdout]"
2021-01-28T13:22:50.521946Z     info    FLAG: --meshConfig="./etc/istio/config/mesh"
2021-01-28T13:22:50.521952Z     info    FLAG: --outlierLogPath=""
2021-01-28T13:22:50.522051Z     info    FLAG: --proxyComponentLogLevel="misc:error"
2021-01-28T13:22:50.522059Z     info    FLAG: --proxyLogLevel="warning"
2021-01-28T13:22:50.522066Z     info    FLAG: --serviceCluster="httpbin.default"
2021-01-28T13:22:50.522078Z     info    FLAG: --stsPort="0"
2021-01-28T13:22:50.522083Z     info    FLAG: --templateFile=""
2021-01-28T13:22:50.522089Z     info    FLAG: --tokenManagerPlugin="GoogleTokenExchange"
2021-01-28T13:22:50.522113Z     info    Version 1.8.2-bfa8bcbc116a8736c301a5dfedc4ed2673e2bfa3-Clean
2021-01-28T13:22:50.522432Z     info    Obtained private IP [10.244.2.160]
2021-01-28T13:22:50.522778Z     info    Apply proxy config from env {"proxyMetadata":{"DNS_AGENT":""}}

2021-01-28T13:22:50.527416Z     info    Effective config: binaryPath: /usr/local/bin/envoy
concurrency: 2
configPath: ./etc/istio/proxy
controlPlaneAuthPolicy: MUTUAL_TLS
discoveryAddress: istiod.istio-system.svc:15012
drainDuration: 45s
envoyAccessLogService: {}
envoyMetricsService: {}
parentShutdownDuration: 60s
proxyAdminPort: 15000
proxyMetadata:
  DNS_AGENT: ""
serviceCluster: httpbin.default
statNameLength: 189
statusPort: 15020
terminationDrainDuration: 5s
tracing:
  zipkin:
    address: zipkin.istio-system:9411

2021-01-28T13:22:50.527785Z     info    Proxy role: &model.Proxy{RWMutex:sync.RWMutex{w:sync.Mutex{state:0, sema:0x0}, writerSem:0x0, readerSem:0x0, readerCount:0, readerWait:0}, Type:"sidecar", IPAddresses:[]string{"10.244.2.160"}, ID:"httpbin-74fb669cc6-fxn2d.default", Locality:(*envoy_config_core_v3.Locality)(nil), DNSDomain:"default.svc.cluster.local", ConfigNamespace:"", Metadata:(*model.NodeMetadata)(nil), SidecarScope:(*model.SidecarScope)(nil), PrevSidecarScope:(*model.SidecarScope)(nil), MergedGateway:(*model.MergedGateway)(nil), ServiceInstances:[]*model.ServiceInstance(nil), IstioVersion:(*model.IstioVersion)(nil), VerifiedIdentity:(*spiffe.Identity)(nil), ipv6Support:false, ipv4Support:false, GlobalUnicastIP:"", XdsResourceGenerator:model.XdsResourceGenerator(nil), WatchedResources:map[string]*model.WatchedResource(nil)}
2021-01-28T13:22:50.527826Z     info    JWT policy is third-party-jwt
2021-01-28T13:22:50.527944Z     info    PilotSAN []string{"istiod.istio-system.svc"}
2021-01-28T13:22:50.528077Z     info    sa.serverOptions.CAEndpoint == istiod.istio-system.svc:15012 Citadel
2021-01-28T13:22:50.528151Z     info    Using CA istiod.istio-system.svc:15012 cert with certs: var/run/secrets/istio/root-cert.pem
2021-01-28T13:22:50.528371Z     info    citadelclient   Citadel client using custom root: istiod.istio-system.svc:15012 -----BEGIN CERTIFICATE-----
MIIC/DCCAeSgAwIBAgIQbJXBav7Aq51/HMd+sXi/YzANBgkqhkiG9w0BAQsFADAY
MRYwFAYDVQQKEw1jbHVzdGVyLmxvY2FsMB4XDTIxMDEyNzE0MDg0NFoXDTMxMDEy
NTE0MDg0NFowGDEWMBQGA1UEChMNY2x1c3Rlci5sb2NhbDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAO/43CahIEZQ94vmhZU0eQ7jc4Ar9Cn4y7DecxGc
h3h80zSrYbLCVS6ZiAY6Ldu0+tCCAcGOJYHmWxN48caw2x2hZUJ1VA0kmeDNXJcx
Nz8eEvFKRR7YyJOgY6EgnzMGfyo2/Uce8QmA9lpc9E3b670tq/4wgnMsuZuRGMJP
5V3Gwcxj1Adz9Oq4c7ZAGGGahYpsnpy6S3ewcPQOuDYTM7eUJuAn6wybidHUR5IO
Y09e5UmoPOPaAMnPymJfbv6pVWuqaVRjUh0vomC7Nh2MQjIK5U6/8qgQNzf+jMIX
Sy3kXkcC9cCuHaTa881AG9y4L/KMLANRYlqAZDqa5jQJjA0CAwEAAaNCMEAwDgYD
VR0PAQH/BAQDAgIEMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFCADFahayt82
jS1r+WwYPO+bocq5MA0GCSqGSIb3DQEBCwUAA4IBAQB43YNsDoWFvFF0aNn1WuXf
/EbcmK/JGaobCF3V2GNMgSaA3O+kxsxG8Ybx1cj2ZXUMUoOukLOnPaybnYg8o69T
cAA2sCa+F7cOOJc6Ho4StdqcMW/TADZPtwaij0zClMvZz+Z9INvsxo9ehN1DCNXL
VN4F6tB6+cLuP4oBGcQlbfUUHJktsCWP0VV8/c9c0HZ/CZVgr7RN+tD9A/MbLLUY
oyQAAp44N0MfLrxshz7kPGFuODW9u8sWOvw1wUrBC4LgVeYtYs1F/kY+iNCM/v9s
EET0Z5sgR3ynAfS8E/7Khzq6oAl1zwwh3NvBKss7JQsPtMwK7FdolD0VTVH9jog1
-----END CERTIFICATE-----

2021-01-28T13:22:51.003014Z     info    sds     SDS gRPC server for workload UDS starts, listening on "./etc/istio/proxy/SDS"

2021-01-28T13:22:51.003058Z     info    xdsproxy        Initializing with upstream address istiod.istio-system.svc:15012 and cluster Kubernetes
2021-01-28T13:22:51.003292Z     info    xdsproxy        adding watcher for certificate var/run/secrets/istio/root-cert.pem
2021-01-28T13:22:51.003504Z     info    Starting proxy agent
2021-01-28T13:22:51.057120Z     info    sds     Start SDS grpc server
2021-01-28T13:22:51.057279Z     info    Received new config, creating new Envoy epoch 0
2021-01-28T13:22:51.057332Z     info    Epoch 0 starting
2021-01-28T13:22:51.059070Z     info    Opening status port 15020

2021-01-28T13:22:51.325780Z     info    Envoy command: [-c etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --parent-shutdown-time-s 60 --service-cluster httpbin.default --service-node sidecar~10.244.2.160~httpbin-74fb669cc6-fxn2d.default~default.svc.cluster.local --local-address-ip-version v4 --bootstrap-version 3 --log-format-prefix-with-location 0 --log-format %Y-%m-%dT%T.%fZ      %l      envoy %n   %v -l warning --component-log-level misc:error --concurrency 2]
2021-01-28T13:22:51.873834Z     warning envoy runtime   Unable to use runtime singleton for feature envoy.http.headermap.lazy_map_min_size
2021-01-28T13:22:51.873921Z     warning envoy runtime   Unable to use runtime singleton for feature envoy.http.headermap.lazy_map_min_size
2021-01-28T13:22:51.915149Z     warning envoy runtime   Unable to use runtime singleton for feature envoy.http.headermap.lazy_map_min_size
2021-01-28T13:22:51.915766Z     warning envoy runtime   Unable to use runtime singleton for feature envoy.http.headermap.lazy_map_min_size
2021-01-28T13:22:52.128168Z     info    xdsproxy        Envoy ADS stream established
2021-01-28T13:22:52.128313Z     info    xdsproxy        connecting to upstream XDS server: istiod.istio-system.svc:15012
2021-01-28T13:23:10.669620Z     error   xdsproxy        failed to create upstream grpc client: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial tcp: lookup istiod.istio-system.svc on 10.96.0.10:53: read udp 10.244.2.160:48221->10.96.0.10:53: i/o timeout"
2021-01-28T13:23:10.669675Z     info    xdsproxy        disconnected from XDS server: istiod.istio-system.svc:15012
2021-01-28T13:23:10.673128Z     warning envoy config    StreamAggregatedResources gRPC config stream closed: 14, connection error: desc = "transport: Error while dialing dial tcp: lookup istiod.istio-system.svc on 10.96.0.10:53: read udp 10.244.2.160:48221->10.96.0.10:53: i/o timeout"
2021-01-28T13:23:10.729078Z     info    xdsproxy        Envoy ADS stream established
2021-01-28T13:23:10.729155Z     info    xdsproxy        connecting to upstream XDS server: istiod.istio-system.svc:15012
2021-01-28T13:23:42.156135Z     warning envoy main      there is no configured limit to the number of allowed active connections. Set a limit via the runtime key overload.global_downstream_max_connections
2021-01-28T13:23:42.413232Z     info    sds     resource:ROOTCA new connection
2021-01-28T13:23:42.413875Z     info    sds     Skipping waiting for gateway secret
2021-01-28T13:23:42.414705Z     info    sds     resource:default new connection
2021-01-28T13:23:42.414772Z     info    sds     Skipping waiting for gateway secret
2021-01-28T13:23:43.587980Z     info    cache   Root cert has changed, start rotating root cert for SDS clients
2021-01-28T13:23:43.588035Z     info    cache   GenerateSecret default
2021-01-28T13:23:43.588656Z     info    sds     resource:default pushed key/cert pair to proxy
2021-01-28T13:23:43.825441Z     info    cache   Loaded root cert from certificate ROOTCA
2021-01-28T13:23:43.825647Z     info    sds     resource:ROOTCA pushed root cert to proxy
2021-01-28T13:23:44.498707Z     warning envoy filter    mTLS PERMISSIVE mode is used, connection can be either plaintext or TLS, and client cert can be omitted. Please consider to upgrade to mTLS STRICT mode for more secure configuration that only allows TLS connection with client cert. See https://istio.io/docs/tasks/security/mtls-migration/
2021-01-28T13:23:44.529898Z     warning envoy filter    mTLS PERMISSIVE mode is used, connection can be either plaintext or TLS, and client cert can be omitted. Please consider to upgrade to mTLS STRICT mode for more secure configuration that only allows TLS connection with client cert. See https://istio.io/docs/tasks/security/mtls-migration/
2021-01-28T13:23:45.132189Z     info    Envoy proxy is ready
2021-01-28T13:54:35.680208Z     info    xdsproxy        disconnected from XDS server: istiod.istio-system.svc:15012
2021-01-28T13:54:35.685051Z     warning envoy config    StreamAggregatedResources gRPC config stream closed: 0,
2021-01-28T13:54:35.942982Z     info    xdsproxy        Envoy ADS stream established
2021-01-28T13:54:35.943141Z     info    xdsproxy        connecting to upstream XDS server: istiod.istio-system.svc:15012
2021-01-28T14:23:19.878005Z     info    xdsproxy        disconnected from XDS server: istiod.istio-system.svc:15012
2021-01-28T14:23:19.879839Z     warning envoy config    StreamAggregatedResources gRPC config stream closed: 0,
2021-01-28T14:23:20.045705Z     info    xdsproxy        Envoy ADS stream established
2021-01-28T14:23:20.045818Z     info    xdsproxy        connecting to upstream XDS server: istiod.istio-system.svc:15012
2021-01-28T14:53:14.776846Z     info    xdsproxy        disconnected from XDS server: istiod.istio-system.svc:15012
2021-01-28T14:53:14.778036Z     warning envoy config    StreamAggregatedResources gRPC config stream closed: 0,
2021-01-28T14:53:14.885843Z     info    xdsproxy        Envoy ADS stream established
2021-01-28T14:53:14.885988Z     info    xdsproxy        connecting to upstream XDS server: istiod.istio-system.svc:15012
```




