### Guida per installazione environment di TEST  - K8S + CENTOS KVM



* #### FASE 1 - Creazione delle VM in ambiente KVM/Cockpit (ubutu based) :

  > seguire come riferimento guida al link : https://it.linux-console.net/?p=1454

> da interfaccia web di cockpit creare **3 VM** con le seguenti carattaristiche :
>
> **8 GB di RAM - 4 CPU - 80 GB HDD e 2 schede di rete ETH** (una lan interna e una esterna)
>
> Hostname : **k8smaster.local - k8snode01.local - k8snode02.local**
>
> poi, una volta ricavato l’IP, connettersi in **SSH** al VM Master **k8smaster.local** 

* #### FASE 2 - Attività sistemistica sulle VM - Propedeutica all’installazione di K8S

  > seguire instruzione nel file locale : **DRAFT_ORIONE-{FRANCO}_INSTALL_k8s_001.md**

