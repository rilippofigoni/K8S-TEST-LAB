## Monitoring Kubernetes using Grafana and Prometheus - INSTALL with HELM on KVM CENTOS 8.3


#### Install Helm Package Manager

```bash
[pippo@test_k8s ~]$ curl -fsSL -o get_helm.sh 

[pippo@test_k8s ~]$ https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

[pippo@test_k8s ~]$ chmod 700 get_helm.sh

[pippo@test_k8s ~]$ ./get_helm.sh

[pippo@test_k8s ~]$ helm init
```
#### Creare un namespace per l'install separato

```bash
[pippo@test_k8s ~]$ kubectl create ns monitor
```
#### Installare PROMETHEUS/GRAFANA da helm 

```bash
[pippo@test_k8s ~]$ helm install prometheus-operator stable/prometheus-operator — namespace monitor
```


#### Verificare la creazione dei pod (ci vogliono alcuni min.)

```bash
[pippo@test_k8s ~]$ kubectl get pods -n monitor

NAME                                                   READY   STATUS    RESTARTS   AGE 
alertmanager-prometheus-operator-alertmanager-0        2/2     Running   0          49s 
prometheus-operator-grafana-5bd6cbc556-w9lds           2/2     Running   0          59s 
prometheus-operator-kube-state-metrics-746dc6ccc-gk2p8 1/1     Running   0          59s 
prometheus-operator-operator-7d69d686f6-wpjtd          2/2     Running   0          59s 
prometheus-operator-prometheus-node-exporter-4nwbf     1/1     Running   0          59s
prometheus-operator-prometheus-node-exporter-jrw69     1/1     Running   0          59s 
prometheus-operator-prometheus-node-exporter-rnqfc     1/1     Running   0          60s 
prometheus-prometheus-operator-prometheus-0            3/3     Running   1          39s
```
#### PER ACCEDERE ALLA DASHBOARD DI PROMETHEUS : http://127.0.0.1:9090

```bash
[pippo@test_k8s ~]$ kubectl port-forward -n monitor prometheus-prometheus-operator-prometheus-0 9090
```

#### PER ACCEDERE ALLA DASHBOARD DI GRAFANA :  http://127.0.0.1:3000

```bash
[pippo@test_k8s ~]$ kubectl port-forward $(kubectl get pods --selector=app=grafana -n monitor --output=jsonpath="{.items..metadata.name}") -n monitor 3000
```
#### PER ACCEDERE  : BISOGNA TROVARE E DECODIFICARE (BASE64) LO USER E LA PWD :

```bash
[pippo@test_k8s ~]$ kubectl get secret --namespace monitor grafana-credentials -o yaml

piVersion: v1 data: password: cHJvbS1vcGVyYXRvcgo= user: YWRtaW4=

[pippo@test_k8s ~]$ echo "YWRtaW4=" | base64 --decode echo "cHJvbS1vcGVyYXRvcgo=" | base64 --decode
```
