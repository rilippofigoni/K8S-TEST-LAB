
 #### FASE 1 - Creazione delle VM in ambiente KVM/Cockpit:
 
 > seguire come riferimento guida al link : https://it.linux-console.net/?p=1454

- da interfaccia web di cockpit creare **3 VM** con le seguenti carattaristiche :
>
  > **8 GB di RAM - 4 CPU - 80 GB HDD e 2 schede di rete ETH** (una lan interna e una esterna)
  >
  > Hostname : **k8smaster.local - k8snode01.local - k8snode02.local**
  >
  > poi, una volta ricavato l’IP, connettersi in **SSH** alla VM Master **k8smaster.local**  (nel caso del master : 192.168.122.185)

  * Per la gestione degli snapshot seguire la guida presente nel file locale : Gestione Snapshot KVM - COCKPIT.md