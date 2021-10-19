### FASE 4 - Installazione K8S sui WORKER NODES

* Per i 2 Worker Nodes, assicurarsi di aver completato la FASI 1, la FASE 2 e la **FASE 3 fino al punto 3.3 compreso** (3.3 K8S - KUBEADM INSTALL),
* dopo di che da ognuno dei Worker Node, lanciare il seguente comando di aggiunta al Master Node k8smaster.local) :

```bash

[pippo@k8snode01 ~]$ kubeadm join 192.168.122.185:6443 --token 661i0n.3c9joy698fqr3tk2 
--discovery-token-ca-cert-hash sha256:7a3cc1f77
71ad2d4853f0b888dd833f77fe7270079f0aecc5254b61bea23cb77

```

* Dopo circa 5 minuti per ogni nodo, appare il messaggio :

```bash

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
 
Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

* Sul Node Master - CONTROLLO DEI NODI AGGIUNTI

```bash
[pippo@k8smaster ~]$ kubectl get nodes

NAME              STATUS   ROLES                  AGE   VERSION
k8smaster.local   Ready    control-plane,master   30h   v1.20.2
k8snode01.local   Ready    <none>                 23h   v1.20.2
k8snode02.local   Ready    <none>                 17m   v1.20.2
```