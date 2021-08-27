#### Update & Upgrade packages on VM
```bash
yum -y update
yum -y upgrade
yum -y install epel-release
```



#### Install & Start Chronyd
```bash
yum -y install chrony
systemctl enable — now chronyd
systemctl status chronyd
```


#### install / update java
```bash
sudo dnf install java-11-openjdk
java -version 

openjdk version "11.0.8" 2020-07-14 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.8+10-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.8+10-LTS, mixed mode, sharing)
```

#### INSTALL EFK STACK : 

```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

sudo nano /etc/yum.repos.d/elasticsearch.repo

[Elasticsearch-7]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```


### Install
```bash
sudo dnf update -y
sudo dnf install elasticsearch -y
```


### CONFIG

```bash
#BACKUP : cp /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.original

sudo nano /etc/elasticsearch/elasticsearch.yml
```


```bash
### VERSIONE 01
 
node.name: efk.local
discovery.zen.minimum_master_nodes: 1
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: [“127.0.0.1”]
cluster.initial_master_nodes: [“efk.local”]

### ALTRA VERSIONE: solo : ####

http.port: 8200
transport.host: localhost
transport.tcp.port: 8300
```

### START SERVICE

```bash
systemctl daemon-reload
systemctl enable --now elasticsearch
systemctl status elasticsearch
```

#### TEST #1

```bash
[root@localhost elasticsearch] curl -X GET "localhost:9200/?pretty"
{
  "name" : "efk.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "aGLz6HbvRP-KGE9vOwwJ7g",
  "version" : {
    "number" : "7.11.2",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "3e5a16cfec50876d20ea77b075070932c6464c7d",
    "build_date" : "2021-03-06T05:54:38.141101Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
```

### TEST 2 : IP locale
 
```bash
[root@efk-elastic-kibana ~] curl http://192.168.122.219:9200/_cluster/health?pretty
{
 “cluster_name” : “elasticsearch”,
 “status” : “green”,
 “timed_out” : false,
 “number_of_nodes” : 1,
 “number_of_data_nodes” : 1,
 “active_primary_shards” : 6,
 “active_shards” : 6,
 “relocating_shards” : 0,
 “initializing_shards” : 0,
 “unassigned_shards” : 3,
 “delayed_unassigned_shards” : 0,
 “number_of_pending_tasks” : 0,
 “number_of_in_flight_fetch” : 0,
 “task_max_waiting_in_queue_millis” : 0,
 “active_shards_percent_as_number” : 66.66666666666666
```

 ### KIBANA INSTALL
 
```bash
sudo yum -y install kibana

yum install httpd-tools

sudo nano /etc/kibana/kibana.yml

>>> ADD : elasticsearch.hosts: ["http://localhost:8200"]
```

### RESTART dei SERVICES

```bash
sudo systemctl daemon-reload
sudo systemctl start kibana
sudo systemctl enable kibana
```
 
### CREATE USERS :
 
```bash
sudo htpasswd -c /etc/nginx/htpasswd.elastic.users elasticuser01
sudo htpasswd -c /etc/nginx/htpasswd.kibana.users kibanauser01
```

### INSTALL NGINX server
 
```bash
yum install install nginx

sudo nano /etc/nginx/elk.conf

>>> ADD/MODIFY al file :

upstream elasticsearch {
  server 127.0.0.1:8200;
  keepalive 15;
}
upstream kibana {
  server 127.0.0.1:5601;
  keepalive 15;
}
server {
  listen 9200;
  location / {
auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.elastic.users;
proxy_pass http://elasticsearch;
    proxy_redirect off;
    proxy_buffering off;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";
  }
}
server {
  listen 8080;
  location / {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.kibana.users;
proxy_pass http://kibana;
    proxy_redirect off;
    proxy_buffering off;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";
  }
}
```

### CONTROLLO FILE CONF : 

```bash
[pippo@efk nginx]$ sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### START SERVICES
```bash
sudo systemctl daemon-reload
sudo systemctl start nginx
sudo systemctl enable nginx
```

### CONTROLLO WEB ACCESS ELASTICSEARCH

```bash
[pippo@efk] $ curl --user elasticuser01:Xxxyyy.12345 127.0.0.1:8200


{
  "name" : "efk.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "aGLz6HbvRP-KGE9vOwwJ7g",
  "version" : {
    "number" : "7.11.2",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "3e5a16cfec50876d20ea77b075070932c6464c7d",
    "build_date" : "2021-03-06T05:54:38.141101Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
```
### CONTROLLO WEB ACCESS KIBANA

```bash
[root@efk nginx] curl --user kibanauser01:Lollo.2003 127.0.0.1:8080
```


#### PER ACCESSO ESTERNO tramite LT (local TUNNEL)
[root@efk nginx]# lt --port 8080
your url is: https://ordinary-fish-17.loca.lt

ET VOILA!!!!!


### CONNECTION TO k8S CLUSTER

### CONTROLLO FILE CONFIGARAZIONE :
```bash
sudo vim /etc/elasticsearch/elasticsearch.yml
```


```bash
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
http.port: 9200
```


#### DISBILITA IL FIREWALL 
```bash
sudo firewall-cmd --permanent --add-port=9200/tcp
sudo firewall-cmd --reload
```


### AGGIUNGI IP A ETC/HOSTS

vedi file INSTALLAZIONE GENERALE (ORIONE)

#### DOWNLOAD DEI FILES

```bash
cd ~
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.9/deploy/kubernetes/filebeat-kubernetes.yaml
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.9/deploy/kubernetes/metricbeat-kubernetes.yaml
```


### DA MODIFICARE SU ENTRAMBE le VM (K8s master e EFK.LOCAL) ed ovunque (controlla bene)

```bash
output.elasticsearch:
      hosts: ['${ELASTICSEARCH_HOST:192.168.122.219}:${ELASTICSEARCH_PORT:8200}']
      #username: ${ELASTICSEARCH_USERNAME}
      #password: ${ELASTICSEARCH_PASSWORD}
 
 env:
        - name: ELASTICSEARCH_HOST
          value: "192.168.122.129"
        - name: ELASTICSEARCH_PORT
          value: "8200"
        #- name: ELASTICSEARCH_USERNAME
        #  value: elasticuser01
        #- name: ELASTICSEARCH_PASSWORD
        #  value: Xxxyyy.123456
        - name: ELASTIC_CLOUD_ID
```


##### DA AGGIUNGERE solo x metricsbeat x le tolleranze

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: metricbeat
  namespace: kube-system
  labels:
    k8s-app: metricbeat
spec:
  selector:
    matchLabels:
      k8s-app: metricbeat
  template:
    metadata:
      labels:
        k8s-app: metricbeat
    spec:
###PART TO EDIT###
      # This toleration is to have the daemonset runnable on master nodes
      # Remove it if your masters can't run pods
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
####END OF EDIT###
```
#### INSTALL in K8S MASTER :

```bash
kubectl apply -f metricbeat-kubernetes.yaml
kubectl apply -f filebeat-kubernetes.yaml
```


#### CONTROLLo su k8s master dei pods (ci vuole 20 min almeno)

```bash

[pippo@k8smaster ~]$ kubectl get pods -n kube-system

NAME                                             READY   STATUS    RESTARTS   AGE
calico-kube-controllers-c9784d67d-k85hf          1/1     Running   5          11d
calico-node-brjnk                                1/1     Running   7          10d
calico-node-nx869                                1/1     Running   1          10d
calico-node-whlzf                                1/1     Running   6          11d
coredns-f9fd979d6-6vztd                          1/1     Running   5          11d
coredns-f9fd979d6-8gz4l                          1/1     Running   5          11d
etcd-kmaster.diab.mfs.co.ke                      1/1     Running   5          11d
filebeat-hlzhc                                   1/1     Running   7          7d23h <==
filebeat-mcs67                                   1/1     Running   1          7d23h <==
kube-apiserver-kmaster.diab.mfs.co.ke            1/1     Running   5          11d
kube-controller-manager-kmaster.diab.mfs.co.ke   1/1     Running   5          11d
kube-proxy-nlrbv                                 1/1     Running   5          11d
kube-proxy-zdcbg                                 1/1     Running   1          10d
kube-proxy-zvf6c                                 1/1     Running   7          10d
kube-scheduler-kmaster.diab.mfs.co.ke            1/1     Running   5          11d
metricbeat-5fw98                                 1/1     Running   7          8d  <==
metricbeat-5zw9b                                 1/1     Running   0          8d  <==
metricbeat-jbppx                                 1/1     Running   1
```
#### Una volta che i nostri Pod iniziano a funzionare, invieranno immediatamente un pattern di indice a Elasticsearch insieme ai log. Accedi a Kibana e fai clic su "Gestione stack"> "Gestione indici" e dovresti essere in grado di vedere gli indici.


### start the KIBANA web interface
http://localhost:8080
lt --port 8080 (in caso ti local tunnel)

