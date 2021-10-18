### FASE 4 - FASE 4 - Installazione K8S sui WORKER NODES

* Per i 2 Worker Nodes, assicurarsi di aver completatole FASI 1, la FASE 2 e la **FASE 3** fino al punto 3.3 compreso (3.3 K8S - KUBEADM INSTALL),
* dopo di che da ognuno dei Worker Node, lanciare il seguente comando di aggiunta al Master Node k8smaster.local) :

```bash

[pippo@k8snode01 ~]$ kubeadm join 192.168.122.185:6443 --token 661i0n.3c9joy698fqr3tk2 
--discovery-token-ca-cert-hash sha256:7a3cc1f77
71ad2d4853f0b888dd833f77fe7270079f0aecc5254b61bea23cb77

```