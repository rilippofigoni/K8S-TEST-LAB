### INSTALL JENKINS ON K8S MASTER - VM-CENTOS

#### CREAZIONE DEL NAMESPACE :

```bash
[centos@k8s_master ~]$ kubectl create namespace jenkins
```

#### CREAZIONE DEL PERSISTENT VOLUME DEDICATO da 10 Gb :

```yaml
[centos@k8s_master ~]$ nano jenkins-volume.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  namespace: jenkins
spec:
  storageClassName: jenkins-pv
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/jenkins-volume/
```
#### DEPLOYMENT : PV


```bash
[centos@k8smaster ~]$ kubectl apply -f jenkins-volume.yaml
persistentvolume/jenkins-pv created
```


#### CREAZIONE DEL FILE YAML DI DEPLOYMENT
#### Questo file YAML crea un DEPLOYMENT utilizzando l'immagine Jenkins LTS e apre anche la porta 8080 e 50000. Queste porte vengono utilizzate rispettivamente per accedere a Jenkins e accettare connessioni dai nodi workers di Jenkins. 

```yaml
[centos@k8s_master ~]$ nano jenkins.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
          - name: http-port
            containerPort: 8080
          - name: jnlp-port
            containerPort: 50000
        volumeMounts:
          - name: jenkins-vol
            mountPath: mnt/data/jenkins-volume
      volumes:
        - name: jenkins-vol
          emptyDir: {}
```

#### DEPLOYMENT :

```bash
[centos@k8s_master ~]$ kubectl create -f jenkins.yaml --namespace jenkins

[centos@k8s_master ~]$ kubectl get pods -n jenkins

NAME                       READY   STATUS    RESTARTS   AGE
jenkins-6fb994cfc5-twnvn   1/1     Running   0          95s

```

#### CREAZIONE DEL NODEPORT SERVICE x ESPORRE IL SERVIZIO :


```bash
[centos@k8s_master ~]$ nano jenkins-service.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30000
  selector:
    app: jenkins

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins
```

#### DEPLOYMENT DEL SERVIZIO

```bash
[centos@k8s_master ~]$ kubectl create -f jenkins-service.yaml --namespace jenkins

[centos@k8s_master ~]$ kubectl get services --namespace jenkins

NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins   NodePort   your_cluster_ip   <none>        8080:30000/TCP   15d
```

#### TROVARE LA PWD di ACCESSO x Admin
#### LOGIN WEB : http://127.0.0.1:8080

```bash
[centos@k8s_master ~]$ printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo


0m1kcFLOXQA
```
