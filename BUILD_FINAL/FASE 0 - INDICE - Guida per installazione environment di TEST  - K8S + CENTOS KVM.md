### Guida per installazione environment di TEST  - K8S + CENTOS KVM

#### PREMESSA:

Lo scopo di questo documento è quello di descrivere in maniera sintetica e accurata le varie fasi occorrenti, *in rigoroso ordine temporale*, al fine di creare di un **TEST-cluster** **Kubernetes**, uno **STACK EFK** e un Private Regiistry **HARBOR**, all'interno di un ambiente virtuale LINUX-KVM, usando 5 VM CentOS 8.



* #### FASE 1 - Creazione delle VM in ambiente Linux KVM/Cockpit:

  > seguire instruzione nel file locale : ** FASE 1 - Creazione delle VM in ambiente KVM-Cockpit.md**


* #### FASE 2 - Attività sistemistica sulle VM - Propedeutica all’installazione di K8S

  > seguire instruzione nel file locale : ** FASE 2 - Attività sistemistica sulle VM - Propedeutica all’installazione di K8S.md**

* #### FASE 3 - Attività d'installazione di Docker e Cluster Kubernetes sul MASTER NODE

  > seguire instruzione nel file locale : ** FASE 3 - Attività d'installazione di Docker e Cluster Kubernetes sul Master Node.md**


* #### FASE 4 - Installazione K8S sui WORKER NODES

  > seguire istruzioni nel file locale : FASE 4 - Installazione K8S sui WORKER NODES.md


* #### FASE 5 - Configurazione CNI-NETWORK del CLUSTER k8S

  > seguire istruzioni nel file locale : FASE 5 - Configurazione CNI-NETWORK del CLUSTER.md

* #### FASE 6 - INSTALLAZIONE DI ISTIO PER SERVICE MESH (+ PLUGINS)

  > seguire istruzioni nel file locale : FASE 6 - Installazione di ISTIO per il Service Mesh.md

* #### FASE 7 - Installazione JENKINS su K8S MASTER

  >seguire istruzioni nel file locale : FASE 7 - Installazione JENKINS su K8S MASTER.md

* #### FASE 8 - INSTALLAZIONE DELLO STACK EFK e CONNESSIONE AL MASTER NODE K8S

  > seguire istruzioni nel file locale : EFK-STACK installazione on K8S.md
  
* #### FASE 9 - INSTALLAZIONE DI HARBOR - PRIVATE REGISTRY e CONNESSIONE AL MASTER NODE

  > seguire istruzioni nel file locale : FASE 9 - Installazione di HARBOR INSTALL on KVM.md