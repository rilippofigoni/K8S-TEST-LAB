## Step 1: Installazione Terraform su CentOS-7

#### Aggiungi il repo :

```
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

#### Conferma che il repository è stato aggiunto e funzionante:

```
$ sudo dnf repolist
repo id                                                              repo name
appstream                                                            CentOS Stream 8 - AppStream
baseos                                                               CentOS Stream 8 - BaseOS
centos-advanced-virtualization                                       CentOS-8 - Advanced Virtualization
centos-ceph-nautilus                                                 CentOS-8 - Ceph Nautilus
centos-nfv-openvswitch                                               CentOS-8 - NFV OpenvSwitch
centos-openstack-wallaby                                             CentOS-8 - OpenStack wallaby
centos-rabbitmq-38                                                   CentOS-8 - RabbitMQ 38
epel                                                                 Extra Packages for Enterprise Linux 8 - x86_64
epel-modular                                                         Extra Packages for Enterprise Linux Modular 8 - x86_64
extras                                                               CentOS Stream 8 - Extras
hashicorp                                                            Hashicorp Stable - x86_64
powertools                                                           CentOS Stream 8 - PowerTools
```

#### Se riesci a vedere hashicorp dall'elenco, procedi con l'installazione di terraform:

```
sudo yum -y install terraform
```

#### Controlla la versione di Terraform

```
$ terraform  version
Terraform v1.0.0
on linux_amd64
```

## Step 3: Installazione  Terraform KVM provider

#### Inizializzazione Terraform working directory.

```
$ cd ~
$ terraform init
Terraform initialized in an empty directory!
```

#### Crea una directory per memorizzare i plugin Terraform

```
cd ~/.terraform.d
mkdir plugins
```

#### Controlla [Github releases](https://github.com/dmacvicar/terraform-provider-libvirt/releases) 

#### Install Terraform KVM provider on CentOS 7 / Fedora / OpenSUSE

Run commands below if you are running Fedora or CentOS in your Workstation.

```
wget https://github.com/dmacvicar/terraform-provider-libvirt/releases/download/v0.6.0/terraform-provider-libvirt-0.6.0+git.1569597268.1c8597df.Fedora_28.x86_64.tar.gz
tar xvf terraform-provider-libvirt-0.6.0+git.1569597268.1c8597df.Fedora_28.x86_64.tar.gz
mv terraform-provider-libvirt ~/.terraform.d/plugins/
```

```

```

## Using Terraform KVM Provider

#### Crea la cartella dei tuoi progetti Terraform.

```
mkdir ~/projects/terraform
cd ~/projects/terraform
```

#### Crea il file `libvirt.tf` per la distribuzione della tua VM su KVM.

```
provider "libvirt" {
  uri = "qemu:///system"
}

#provider "libvirt" {
#  alias = "server2"
#  uri   = "qemu+ssh://root@192.168.100.10/system"
#}

resource "libvirt_volume" "centos7-qcow2" {
  name = "centos7.qcow2"
  pool = "default"
  source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
  #source = "./CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

# Define KVM domain to create
resource "libvirt_domain" "db1" {
  name   = "db1"
  memory = "1024"
  vcpu   = 1

  network_interface {
    network_name = "default"
  }

  disk {
    volume_id = "${libvirt_volume.centos7-qcow2.id}"
  }

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}
```

#### Inizializzare una directory di lavoro Terraform:

```
$ terraform init
Initializing provider plugins…

Terraform has been successfully initialized!
You may now begin working with Terraform. Try running "terraform plan" to see any changes that are required for your infrastructure. All Terraform commands should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

#### Genera e mostra PLAN di Terraform

```
$ terraform plan

Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + libvirt_domain.db1
      id:                               <computed>
      arch:                             <computed>
      console.#:                        "1"
      console.0.target_port:            "0"
      console.0.target_type:            "serial"
      console.0.type:                   "pty"
      disk.#:                           "1"
      disk.0.scsi:                      "false"
      disk.0.volume_id:                 "${libvirt_volume.centos7-qcow2.id}"
      emulator:                         <computed>
      graphics.#:                       "1"
      graphics.0.autoport:              "true"
      graphics.0.listen_address:        "127.0.0.1"
      graphics.0.listen_type:           "address"
      graphics.0.type:                  "spice"
      machine:                          <computed>
      memory:                           "1024"
      name:                             "db1"
      network_interface.#:              "1"
      network_interface.0.addresses.#:  <computed>
      network_interface.0.hostname:     <computed>
      network_interface.0.mac:          <computed>
      network_interface.0.network_id:   <computed>
      network_interface.0.network_name: "default"
      qemu_agent:                       "false"
      running:                          "true"
      vcpu:                             "1"

  + libvirt_volume.centos7-qcow2
      id:                               <computed>
      format:                           "qcow2"
      name:                             "centos7.qcow2"
      pool:                             "default"
      size:                             <computed>
      source:                           "./CentOS-7-x86_64-GenericCloud.qcow2"


Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

#### Crea la tua infrastruttura Terraform se lo stato desiderato è confermato .

```
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + libvirt_domain.db1
      id:                               <computed>
      arch:                             <computed>
      console.#:                        "1"
      console.0.target_port:            "0"
      console.0.target_type:            "serial"
      console.0.type:                   "pty"
      disk.#:                           "1"
      disk.0.scsi:                      "false"
      disk.0.volume_id:                 "${libvirt_volume.centos7-qcow2.id}"
      emulator:                         <computed>
      graphics.#:                       "1"
      graphics.0.autoport:              "true"
      graphics.0.listen_address:        "127.0.0.1"
      graphics.0.listen_type:           "address"
      graphics.0.type:                  "spice"
      machine:                          <computed>
      memory:                           "1024"
      name:                             "db1"
      network_interface.#:              "1"
      network_interface.0.addresses.#:  <computed>
      network_interface.0.hostname:     <computed>
      network_interface.0.mac:          <computed>
      network_interface.0.network_id:   <computed>
      network_interface.0.network_name: "default"
      qemu_agent:                       "false"
      running:                          "true"
      vcpu:                             "1"

  + libvirt_volume.centos7-qcow2
      id:                               <computed>
      format:                           "qcow2"
      name:                             "centos7.qcow2"
      pool:                             "default"
      size:                             <computed>
      source:                           "./CentOS-7-x86_64-GenericCloud.qcow2"


Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

#### Premere “**yes**” per confermare l'esecuzione. 

#### Di seguito è riportato il mio output di esecuzione di terraform.

```
libvirt_volume.centos7-qcow2: Creating...
  format: "" => "qcow2"
  name:   "" => "db.qcow2"
  pool:   "" => "default"
  size:   "" => "<computed>"
  source: "" => "./CentOS-7-x86_64-GenericCloud.qcow2"
libvirt_volume.centos7-qcow2: Creation complete after 8s (ID: /var/lib/libvirt/images/db.qcow2)
libvirt_domain.db1: Creating...
  arch:                             "" => "<computed>"
  console.#:                        "" => "1"
  console.0.target_port:            "" => "0"
  console.0.target_type:            "" => "serial"
  console.0.type:                   "" => "pty"
  disk.#:                           "" => "1"
  disk.0.scsi:                      "" => "false"
  disk.0.volume_id:                 "" => "/var/lib/libvirt/images/db.qcow2"
  emulator:                         "" => "<computed>"
  graphics.#:                       "" => "1"
  graphics.0.autoport:              "" => "true"
  graphics.0.listen_address:        "" => "127.0.0.1"
  graphics.0.listen_type:           "" => "address"
  graphics.0.type:                  "" => "spice"
  machine:                          "" => "<computed>"
  memory:                           "" => "1024"
  name:                             "" => "db1"
  network_interface.#:              "" => "1"
  network_interface.0.addresses.#:  "" => "<computed>"
  network_interface.0.hostname:     "" => "<computed>"
  network_interface.0.mac:          "" => "<computed>"
  network_interface.0.network_id:   "" => "<computed>"
  network_interface.0.network_name: "" => "default"
  qemu_agent:                       "" => "false"
  running:                          "" => "true"
  vcpu:                             "" => "1"
libvirt_domain.db1: Creation complete after 0s (ID: e5ee28b9-e1da-4945-9eb0-0cda95255937)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

#### Conferma la creazione della VM con il comando "virsh".

```
$ sudo virsh  list
 Id   Name   State
----------------------
 7    db1    running
```

#### Ottieni l’ IP :

```
$ sudo virsh net-dhcp-leases default 
 Expiry Time           MAC address         Protocol   IP address           Hostname   Client ID or DUID
------------------------------------------------------------------------------------------------------------------------------------------------
 2019-03-24 16:11:18   52:54:00:3e:15:9e   ipv4       192.168.122.61/24    -          -
 2019-03-24 15:30:18   52:54:00:8f:8c:86   ipv4       192.168.122.198/24   rhel8      ff:61:69:21:bd:00:02:00:00:ab:11:0e:9c:c6:63:ee:7d:c8:d1
```

```
$  ping -c 1 192.168.122.61 
PING 192.168.122.61 (192.168.122.61) 56(84) bytes of data.
64 bytes from 192.168.122.61: icmp_seq=1 ttl=64 time=0.517 ms

--- 192.168.122.61 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.517/0.517/0.517/0.000 ms
```

#### Per distruggere la tua infrastruttura, esegui:

```
terraform destroy
```

