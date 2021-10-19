### Guida per installazione environment di TEST  - K8S + CENTOS KVM

#### PREMESSA:

Lo scopo di questo documento è quello di descrivere in maniera sintetica e accurata le varie fasi occorrenti - in rigoroso ordine temporale - per la creazione di un **TEST-cluster** Kubernetes all'interno di un ambiente virtuale LINUX-KVM, usando delle CentOS 8.
Seguiranno poi, nei files di esempio sucessivi, le istruzioni per creare altre 2 KVM atte ad ospitare un'installazione di HARBOR Registry ed un altra per lo STACK EFK.
Infine

* #### FASE 1 - Creazione delle VM in ambiente KVM/Cockpit:

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
    
  #### ESEMPIO di SERVICE MESH (BOOKINFO WEB APPS and SERVICES)

> seguire istruzioni nel file locale : ESEMPIO di SERVICE MESH - BOOKINFO WEB APPS and SERVICES.md