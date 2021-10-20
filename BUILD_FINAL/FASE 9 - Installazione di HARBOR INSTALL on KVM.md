### FASE 9 - Installazione di HARBOR INSTALL on KVM.md

 > Per la creazione della VM adatta, seguire la guida presente nel file locale (FASE 1 - Creazione delle VM in ambiente KVM-Cockpit.md)
  > con l'unica diifferenza di aumentare a 160 GB il size del disco o in alternativa aggiungerne uno da 80 GB.

#### FASE 9.1 - DEFINIZIONE DELL'HOSTNAME

```bash
sudo hostnamectl set-hostname efk_stack.local
```

#### FASE 9.2 - CAMBIO TASTIERA LAYOUT ITA

```bash
localectl set-keymap it
```

#### FASE 9.3 - SETTAGGIO DEL TIME  

```bash
sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-local-rtc true
```

#### FASE 9.4 - CREAZIONE UTENTE : (sudoers)

```bash
adduser pippo

passwd pippo

usermod -aG wheel pippo
```

#### FASE 9.5 - CREAZIONE DEL FILE /ETC/HOSTS

```bash
sudo cat <<EOF>> /etc/hosts
192.168.122.185         k8smaster.local     k8smaster
192.168.122.8	          k8snode01.local     k8snode1
192.168.122.213         k8snode02.local     k8snode2
192.168.122.129         efk_stack.local     efk_stack
192.168.122.149         harbor.local        harbor  
EOF
```

#### FASE 9.6 - DEFINIZIONE DELLE SCHEDE DI RETE

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


#### FASE 9.7 - SETTAGGIO per APETURA FIREWALL

```bash
[pippo@efk_stack ~]$ sudo setenforce 0
[pippo@efk_stack ~]$ sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

#### FASE 9.8 - Update & Upgrade packages e EPEL repo

```bash
yum -y update
yum -y upgrade
yum -y install epel-release
```


#### FASE 9.9 -  Installazione dei prerequisiti Linux


```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### FASE 9.10 - INSTALLAZIONE Docker Engine REPO

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

#### FASE 9.11 - INSTALLAZIONE DOCKER-CE

```bash
sudo yum -y install docker-ce docker-ce-cli containerd.io
```
  >(in caso di errore)
```bash
sudo yum install -y --setopt=obsoletes=0 docker-ce docker-ce-selinux
```

#### FASE 9.12 - Start & enable il servizio :

```bash
sudo systemctl start docker && sudo systemctl enable docker
```

#### FASE 9.12 - Installare Docker Compose

#### FASE 9.12.1 - Download pacchetto

```bash
curl -s https://api.github.com/repos/docker/compose/releases/latest \
  | grep browser_download_url \
  | grep docker-compose-Linux-x86_64 \
  | cut -d '"' -f 4 \
  | wget -qi -
```

#### FASE 9.12.2 - Renderlo eseguibile

```bash
chmod +x docker-compose-Linux-x86_64
```

#### FASE 9.12.3 - Spostarlo nel PATH.

```bash
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
```

#### FASE 9.12.4 - Confermare la versione

```bash
$ docker-compose version
docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

#### FASE 9.12.5 - aggiungerlo al nuovo docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

#### FASE 9.13 - Download and Installazione di Harbor

#### FASE 9.13.1 - Download harbor

```bash
curl -s https://api.github.com/repos/goharbor/harbor/releases/latest | grep browser_download_url | cut -d '"' -f 4 | grep '\.tgz$' | wget -i -
```

#### FASE 9.13.2 - Unpack del file.

```bash
tar xvzf harbor-offline-installer*.tgz
```

#### FASE 9.13.3 - Posizionarsi cartella nella cartella harbor

  >se no non va !!!

```bash
cd harbor
```

#### FASE 9.14 - Harbor : Installatione senza SSL

> FASE 9.14.1 - fare una copia di sicurezza!

```bash
cp harbor.yml.tmpl harbor.yml
```

#### FASE 9.14.2 - Modifica l'harbor configuration file : 

```bash
$ nano harbor.yml
```

>Modifica in questo modo :

```bash
# The IP address or hostname to access admin UI and registry service.
hostname: registry.pippo.com

harbor_admin_password: StrongAdminP@s5W0$d

# Harbor DB configuration
database:
  password: StrongdbrootP@s5W0$d
Harbor Installation with Let’s Encrypt SSL
if your server has a public IP, you can use Let’s Encrypt free SSL certificate.
```

#### FASE 9.14.3 - Installare il certbot-auto tool:

  >per generare il proprio certificato

```bash
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
sudo mv certbot-auto /usr/local/bin
```

  >Ottenere SSL certificato :

```bash
export DOMAIN="registry.pippo.com"
export EMAIL="admin@example.com"
certbot-auto certonly --standalone -d $DOMAIN --preferred-challenges http --agree-tos -n -m $EMAIL --keep-until-expiring
```

#### FASE 9.14.4 -  Configurare con i parametri nel file harbor.yml:

>Modifica in questo modo :

```bash
hostname: registry.pippo.com
harbor_admin_password: StrongAdminP@s5W0$d

# Harbor DB configuration
database:
  password: StrongdbrootP@s5W0$d

http:
  port: 80

https:
  port: 443
  certificate: /etc/letsencrypt/live/registry.computingforpippo.com/fullchain.pem
  private_key: /etc/letsencrypt/live/registry.computingforpippo.com/privkey.pem
Harbor Installation with Self Signed SSL Certificates


```

#### FASE - 9.14.5 : MODIFICA FILE CERIFICATO

  >Per i certificati autofirmati, crea un file di configurazione del certificato: modifica il file in modo che corrisponda ai tuoi valori.

```bash
$ cd /etc/pki/tls/certs
$ sudo vim harbor_certs.cnf
```
MODIFICA

```bash
[ req ]  
default_bits       = 4096
default_md         = sha512
default_keyfile    = harbor_registry.key
prompt             = no
encrypt_key        = no
distinguished_name = req_distinguished_name

# distinguished_name
[ req_distinguished_name ]  
countryName            = "IT" 
localityName           = "Verona"
stateOrProvinceName    = "Verona"
organizationName       = "pippo"
commonName             = "registry.pippo.com"
emailAddress           = "webmaster@pippo.com"
```

#### FASE 9.14.6 - Generate key e csr:

```bash
sudo openssl req -out harbor_registry.csr -newkey rsa:4096 --sha512 -nodes -keyout harbor_registry.key -config harbor_certs.cnf
```

#### FASE 9.15.7 - Creare self-singed certificate con 10 anni di expiration date:

```bash
sudo openssl x509 -in harbor_registry.csr -out harbor_registry.crt -req -signkey harbor_registry.key -days 3650
```

#### FASE 9.15.8 - Per viisualzzare il certificato :

```bash
$ openssl x509 -text -noout -in harbor_registry.crt
```

#### FASE 9.16 - Configurare con nuovi parametri https config file : 

```bash
hostname: registry.superpippo.com
harbor_admin_password: StrongAdminP@s5W0$d

# Harbor DB configuration
database:
  password: StrongdbrootP@s5W0$d

http:
  port: 80

https:
  port: 443
  certificate: ./harbor_registry.crt
  private_key: ./harbor_registry.key
```

#### FASE 9.17 - Installare Harbor Docker image registry

  >quando harbor.yml and è configurato, installare and startare Harbor usando lo script: install.sh
  >include Notary o Chartmuseum


```bash
$ sudo ./install.sh --with-notary --with-chartmuseum
```

```bash
[Step 0]: checking installation environment ...

Note: docker version: 19.03.1

Note: docker-compose version: 1.24.1

[Step 1]: loading Harbor images ...
Loaded image: goharbor/harbor-core:v1.8.1
Loaded image: goharbor/harbor-registryctl:v1.8.1
Loaded image: goharbor/redis-photon:v1.8.1
Loaded image: goharbor/notary-server-photon:v0.6.1-v1.8.1
Loaded image: goharbor/chartmuseum-photon:v0.8.1-v1.8.1
Loaded image: goharbor/harbor-db:v1.8.1
Loaded image: goharbor/harbor-jobservice:v1.8.1
Loaded image: goharbor/nginx-photon:v1.8.1
Loaded image: goharbor/registry-photon:v2.7.1-patch-2819-v1.8.1
Loaded image: goharbor/harbor-migrator:v1.8.1
Loaded image: goharbor/prepare:v1.8.1
Loaded image: goharbor/harbor-portal:v1.8.1
Loaded image: goharbor/harbor-log:v1.8.1
Loaded image: goharbor/notary-signer-photon:v0.6.1-v1.8.1
Loaded image: goharbor/clair-photon:v2.0.8-v1.8.1

[Step 2]: preparing environment ...
prepare base dir is set to /root/harbor
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /secret/keys/secretkey
Generated certificate, key file: /secret/core/private_key.pem, cert file: /secret/registry/root.crt
Generated configuration file: /config/clair/postgres_env
Generated configuration file: /config/clair/config.yaml
Generated configuration file: /config/clair/clair_env
Create config folder: /config/chartserver
Generated configuration file: /config/chartserver/env
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

[Step 3]: starting Harbor ...

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://registry.computingforpippo.com. 
For more details, please visit https://github.com/goharbor/harbor .
Confirm that all containers are started.

.....
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating network "harbor_harbor-clair" with the default driver
Creating network "harbor_harbor-notary" with the default driver
Creating network "harbor_harbor-chartmuseum" with the default driver
Creating network "harbor_notary-sig" with the default driver
Creating harbor-log ... done
Creating registry      ... done
Creating registryctl   ... done
Creating harbor-db     ... done
Creating redis         ... done
Creating harbor-portal ... done
Creating chartmuseum   ... done
Creating notary-signer ... done
Creating clair         ... done
Creating harbor-core   ... done
Creating notary-server     ... done
Creating clair-adapter     ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
Harbor log files are stored in the directory /var/log/harbor/:
```



#### FASE 9.17.1 - PER VEDERE I LOGS: 

```bash
$ ls -1 /var/log/harbor/
chartmuseum.log
clair.log
core.log
jobservice.log
portal.log
postgresql.log
proxy.log
redis.log
registryctl.log
registry.log
```

### FASE 9.18 - Accesso alla web interface di Harbor

da browser web : https://registry.pippo.com

Login with:

Username: admin
Password: Set-in-harbor.yml (StrongAdminP@s5W0$d)


#### FASE 9.19 - CONTROLLO DELLO STATO GENERALE :

```bash
$ sudo docker-compose ps
      Name                     Command                       State                                          Ports                               
------------------------------------------------------------------------------------------------------------------------------------------------
chartmuseum         ./docker-entrypoint.sh           Up (healthy)                                                                               
clair               ./docker-entrypoint.sh           Restarting                                                                                 
clair-adapter       /home/clair-adapter/entryp ...   Up (healthy)                                                                               
harbor-core         /harbor/entrypoint.sh            Up (health: starting)                                                                      
harbor-db           /docker-entrypoint.sh            Up (healthy)                                                                               
harbor-jobservice   /harbor/entrypoint.sh            Up (health: starting)                                                                      
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)            127.0.0.1:1514->10514/tcp                                          
harbor-portal       nginx -g daemon off;             Up (healthy)                                                                               
nginx               nginx -g daemon off;             Up (healthy)            0.0.0.0:4443->4443/tcp, 0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp
notary-server       /bin/sh -c migrate-patch - ...   Up                                                                                         
notary-signer       /bin/sh -c migrate-patch - ...   Up                                                                                         
redis               redis-server /etc/redis.conf     Up (healthy)                                                                               
registry            /home/harbor/entrypoint.sh       Up (healthy)                                                                               
registryctl         /home/harbor/start.sh            Up (healthy)
```

#### FASE 9.20 - CARICAMENTO IMMAGINE SUL REGISTRY PUBBLICO di HARBOR

  >Prima creazione, da console web, di un registro pubblico di TEST, chiamato test01

```bash
[pippo@harbor docker]$ docker tag hello-world:latest harbor.local:443/test01/hello-world:latest

[pippo@harbor docker]$ docker images

REPOSITORY                            TAG       IMAGE ID       CREATED         SIZE
goharbor/chartmuseum-photon           v2.0.6    9c64bdcdf16e   3 weeks ago     179MB
goharbor/redis-photon                 v2.0.6    1046dae90f91   3 weeks ago     73.4MB
goharbor/trivy-adapter-photon         v2.0.6    ea2a59d236ae   3 weeks ago     118MB
goharbor/clair-adapter-photon         v2.0.6    320aa2f49b1c   3 weeks ago     70.4MB
goharbor/clair-photon                 v2.0.6    2bebca835a51   3 weeks ago     173MB
goharbor/notary-server-photon         v2.0.6    76ff12789839   3 weeks ago     110MB
goharbor/notary-signer-photon         v2.0.6    c9a3d681945d   3 weeks ago     107MB
goharbor/harbor-registryctl           v2.0.6    0c0af4305582   3 weeks ago     103MB
goharbor/registry-photon              v2.0.6    4d6a010ad0ea   3 weeks ago     85.7MB
goharbor/nginx-photon                 v2.0.6    546306cc116a   3 weeks ago     44.8MB
goharbor/harbor-log                   v2.0.6    f9d83783eda0   3 weeks ago     107MB
goharbor/harbor-jobservice            v2.0.6    cfb920622b03   3 weeks ago     167MB
goharbor/harbor-core                  v2.0.6    d7055c6a1b48   3 weeks ago     146MB
goharbor/harbor-portal                v2.0.6    1722ad76a226   3 weeks ago     53.7MB
goharbor/harbor-db                    v2.0.6    8d3a2b37960f   3 weeks ago     173MB
goharbor/prepare                      v2.0.6    d70c9eb0e05a   3 weeks ago     161MB
harbor.local:443/test01/hello-world   latest    bf756fb1ae65   13 months ago   13.3kB
hello-world                           latest    bf756fb1ae65   13 months ago   13.3kB

[pippo@harbor docker]$ docker push harbor.local:443/test01/hello-world:latest
The push refers to repository [harbor.local:443/test01/hello-world]
9c27e219663c: Pushed
latest: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525
```

#### FASE 9.21 - LOGIN to harbor da console #####

```bash
docker login https://harbor.local:443
```
user : admin
pwd : Harbor12345

```bash
[pippo@harbor ~]$ docker login https://harbor.local:443
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /home/pippo/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
```

#### FASE 9.21.1 - PULL dal Public registry dell'IMMAGINE ex : UBUNTU 14.04 #######

```bash
[pippo@harbor ~]$ docker pull ubuntu:14.04

14.04: Pulling from library/ubuntu
2e6e20c8e2e6: Pull complete
0551a797c01d: Pull complete
512123a864da: Pull complete
Digest: sha256:4a8a6fa8810a3e01352981b35165b0b28403fe2a4e2535e315b23b4a69cd130a
Status: Downloaded newer image for ubuntu:14.04
docker.io/library/ubuntu:14.04

pippo@harbor ~]$ docker tag ubuntu:14.04  harbor.local:443/testpriv/ubuntu:14.04


[pippo@harbor ~]$ docker push harbor.local:443/testpriv/ubuntu:14.04
The push refers to repository [harbor.local:443/testpriv/ubuntu]
83109fa660b2: Pushed
30d3c4334a23: Pushed
f2fa9f4cf8fd: Pushed
14.04: digest: sha256:881afbae521c910f764f7187dbfbca3cc10c26f8bafa458c76dda009a901c29d size: 945
```

#### FASE 9.22 - PER USARE UN REGISTRO IN k8SMASTER : #####

  >bisogna abilitare l'insicure regiistry

```bash
sudo nano /etc/docker/daemon.json

ADD : { "insecure-registries":["https://harbor.local:443"] }

sudo systemctl restart docker
```

#### FASE 9.23 PULL IMAGE da repository Test01 su K8S MASTER

```bash
pippo@k8smaster $ /etc/docker  sudo docker pull harbor.local:443/test01/hello-world
[sudo] password for pippo:
Using default tag: latest
latest: Pulling from test01/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042
Status: Downloaded newer image for harbor.local:443/test01/hello-world:latest
harbor.local:443/test01/hello-world:latest

sudo docker images 

harbor.local:443/test01/hello-world   latest        bf756fb1ae65   15 months ago   13.3kB
```

#### - APPUNTI VARI ed EVENTUALI :

  >SEQUENZA DI RIAVVIO HARBOR :::::>>> in caso di mancato accesso al server : http:/harbor.local

```bash
cd /home/centos/harbor/
sudo docker-compose down -v
sudo docker-compose up -d
sudo docker-compose start

curl http://harbor.local

<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
[centos@harbor harbor]$
```

  >ALTRO METODO x RIAVVIO : (se non funzia sopra)

```bash
docker-compose down -v
cd /home/centos/harbor/
ls
docker-compose down -v
sudo systemctl stop harbor
sudo docker-compose start
curl http://harbor.local
```

#### ALTRO MODO X RESTART (+ pulito)

Stopping Harbor:

```bash
$ sudo docker-compose stop
stopping nginx             ...
Stopping harbor-jobservice ... done
Stopping harbor-portal     ... done
Stopping clair             ... done
Stopping chartmuseum       ... done
Stopping harbor-core       ... done
Stopping harbor-db         ... done
Stopping redis             ... done
Stopping registry          ... done
Stopping registryctl       ... done
Stopping harbor-log        ... done

$ sudo docker-compose start
Starting log         ... done
Starting registry    ... done
Starting registryctl ... done
Starting postgresql  ... done
Starting core        ... done
Starting portal      ... done
Starting redis       ... done
Starting jobservice  ... done
Starting proxy       ... done
Starting clair       ... done
Starting chartmuseum ... done
```
#### VEDERE I LOG in CASO DI ERRORI : xxxx = file

```bash
$ tail -n 100 /var/log/harbor/xxxxx.log
```