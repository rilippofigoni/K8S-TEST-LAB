#### FASE 8 - INSTALLAZIONE di uno STACK EFK

  > Per la creazione della VM adatta, seguire la guida presente nel file locale (FASE 1 - Creazione delle VM in ambiente KVM-Cockpit.md)
  > con l'unica diifferenza di aumentare a 160 GB il size del disco o in alternativa aggiungerne uno da 80 GB.


#### FASE 8.1 - DEFINIZIONE DELL'HOSTNAME

```bash
sudo hostnamectl set-hostname efk_stack.local
```

#### FASE 8.2 - CAMBIO TASTIERA LAYOUT ITA

```bash
localectl set-keymap it
```

#### FASE 8.3 - SETTAGGIO DEL TIME  

```bash
sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-local-rtc true
```

#### FASE 8.4 - CREAZIONE UTENTE : (sudoers)

```bash
adduser pippo

passwd pippo

usermod -aG wheel pippo
```

#### FASE 8.5 - CREAZIONE DEL FILE /ETC/HOSTS

```bash
sudo cat <<EOF>> /etc/hosts
192.168.122.185         k8smaster.local     k8smaster
192.168.122.8	          k8snode01.local     k8snode1
192.168.122.213         k8snode02.local     k8snode2
192.168.122.129         efk_stack.local     efk_stack  
EOF
```

#### FASE 8.6 - DEFINIZIONE DELLE SCHEDE DI RETE

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


#### FASE 8.7 - SETTAGGIO per APETURA FIREWALL

```bash
[pippo@efk_stack ~]$ sudo setenforce 0
[pippo@efk_stack ~]$ sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

#### FASE 8.8 - Update & Upgrade packages e EPEL repo

```bash
yum -y update
yum -y upgrade
yum -y install epel-release
```


#### FASE 8.9 : Installazione & Start Chronyd

```bash
yum -y install chrony
systemctl enable — now chronyd
systemctl status chronyd
```


#### FASE 8.10 - Installazione / update java

```bash
sudo dnf install java-11-openjdk
java -version 

openjdk version "11.0.8" 2020-07-14 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.8+10-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.8+10-LTS, mixed mode, sharing)
```

#### FASE 8.11 - INSTALL DELLO STACK EFK : 

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


sudo dnf update -y
sudo dnf install elasticsearch -y
```


#### FASE 8.12 - CONFIGURAZIONE Di ELASTICSEARCH

  >Ricordarsi : #BACKUP : cp /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.original

```bash

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

#### FASE 8.13 - START DEL SERVIZIO

```bash
systemctl daemon-reload
systemctl enable --now elasticsearch
systemctl status elasticsearch
```

#### FASE 8.14 : TEST #1

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

#### FASE 8.15 : TEST #2 - IP locale
 
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

#### FASE 8.16 - INSTALLAZIONE DI KIBANA
 
```bash
sudo yum -y install kibana

yum install httpd-tools

sudo nano /etc/kibana/kibana.yml

  >>>> AGGIUNGERE : elasticsearch.hosts: ["http://localhost:8200"]
```

#### FASE 8.17 - RESTART dei SERVIZI

```bash
sudo systemctl daemon-reload
sudo systemctl start kibana
sudo systemctl enable kibana
```
 
#### FASE 8.18 - CREAZIONE DEGLI UTENTI :
 
```bash
sudo htpasswd -c /etc/nginx/htpasswd.elastic.users elasticuser01
sudo htpasswd -c /etc/nginx/htpasswd.kibana.users kibanauser01
```

### FASE 8.19 - INSTALLAZIONE e CONFIGURAZIONE NGINX
 
```bash
yum install install nginx

sudo nano /etc/nginx/elk.conf

>>>> **ADD/MODIFY al file :

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

#### FASE 8.20 : CONTROLLO FILE CONF DI NGINX : 

```bash
[pippo@efk nginx]$ sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### FASE 8.21 - START DEI SERVIZI :

```bash
sudo systemctl daemon-reload
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### FASE 8.22 - CONTROLLO WEB ACCESS ELASTICSEARCH

```bash
[pippo@efk_stack] $ curl --user elasticuser01:Xxxyyy.12345 127.0.0.1:8200


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

#### FASE 8.23 - CONTROLLO WEB ACCESS KIBANA

```bash
[root@efk_stack] curl --user kibanauser01:Lollo.2003 127.0.0.1:8080
```



#### FASE 8.24 - CONNECTION TO k8S CLUSTER - CONTROLLO FILE CONFIGARAZIONE :

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


#### FASE 8.25 - DOWNLOAD DEI FILES FILEBEAT E METRICBEAT PER IL DEPLOY

```bash
cd ~
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.9/deploy/kubernetes/filebeat-kubernetes.yaml
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.9/deploy/kubernetes/metricbeat-kubernetes.yaml
```


### FASE 8.26 - DA MODIFICARE SU ENTRAMBE le VM (K8s master e EFK_STACK.LOCAL) ed ovunque (controlla bene)

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


#### FASE 8.27 - DA AGGIUNGERE solo x metricsbeat x le tolleranze

  > vi metricbeat-kubernetes.yaml

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


#### FASE 8.28 -  INSTALLAZIONE NEL K8S MASTER :

```bash
kubectl apply -f metricbeat-kubernetes.yaml
kubectl apply -f filebeat-kubernetes.yaml
```


#### FASE 8.29 - CONTROLLO su k8s master dei pods (ci vuole 20 min almeno)

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


#### N.B : Una volta che i nostri Pod iniziano a funzionare, invieranno immediatamente un pattern di indice a Elasticsearch insieme ai log. Accedi a Kibana e fai clic su "Gestione stack"> "Gestione indici" e dovresti essere in grado di vedere gli indici.


### start the KIBANA web interface
http://localhost:8080


