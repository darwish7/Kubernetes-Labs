# lab2
## 1-Create a ReplicaSet using the below yaml
we touch a file RS.yml with this yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: new-replica-set
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - command:
        - sh
        - -c
        - echo Hello Kubernetes! && sleep 3600
        image: busybox777
        imagePullPolicy: Always
        name: busybox-container 
```
then we run 
```
kubectl apply -f RS.yml 
controlplane $ kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-2bddg   0/1     ImagePullBackOff   0          16m
new-replica-set-8vskb   0/1     ImagePullBackOff   0          16m
new-replica-set-bf4rj   0/1     ImagePullBackOff   0          16m
new-replica-set-pdnj6   0/1     ImagePullBackOff   0          16m
```
## 2- How many PODs are DESIRED in the new-replica-set?
### 4
```
controlplane $ kubectl describe replicasets new-replica-set 
Name:         new-replica-set
Namespace:    default
Selector:     name=busybox-pod
Labels:       <none>
Annotations:  <none>
Replicas:     4 current / 4 desired
Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox777
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  22m   replicaset-controller  Created pod: new-replica-set-pdnj6
  Normal  SuccessfulCreate  22m   replicaset-controller  Created pod: new-replica-set-2bddg
  Normal  SuccessfulCreate  22m   replicaset-controller  Created pod: new-replica-set-8vskb
  Normal  SuccessfulCreate  22m   replicaset-controller  Created pod: new-replica-set-bf4rj
```
## 3-What is the image used to create the pods in the new-replica-set?
### busybox777
```
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox777
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:
```
## 4-How many PODs are READY in the new-replica-set?  
### none 
```
controlplane $ kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-2bddg   0/1     ImagePullBackOff   0          31m
new-replica-set-8vskb   0/1     ImagePullBackOff   0          31m
new-replica-set-bf4rj   0/1     ImagePullBackOff   0          31m
new-replica-set-pdnj6   0/1     ImagePullBackOff   0          31m
```
## 5-Why do you think the PODs are not ready?
### ImagePullBackOff which means the image can't be pulled 
## 6-Delete any one of the 4 PODs
```
controlplane $ kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-2bddg   0/1     ImagePullBackOff   0          36m
new-replica-set-8vskb   0/1     ImagePullBackOff   0          36m
new-replica-set-bf4rj   0/1     ImagePullBackOff   0          36m
new-replica-set-pdnj6   0/1     ImagePullBackOff   0          36m

controlplane $ kubectl delete pod new-replica-set-2bddg
pod "new-replica-set-2bddg" deleted

controlplane $ kubectl get pods
NAME                    READY   STATUS              RESTARTS   AGE
new-replica-set-8vskb   0/1     ImagePullBackOff    0          37m
new-replica-set-bf4rj   0/1     ImagePullBackOff    0          37m
new-replica-set-pdnj6   0/1     ImagePullBackOff    0          37m
new-replica-set-wj7jm   0/1     ContainerCreating   0          3s
```
## 7-Why are there still 4 PODs, even after you deleted one?
### because the desired in the replica set is 4 
## 8-Create a ReplicaSet using the below yaml
```
apiVersion: v1 ---> apps/v1

replicaset-1-7rzvm      1/1     Running            0          12s
replicaset-1-zwdzs      1/1     Running            0          12s

```
# day 3 lab
## Create basic resources 
---
#### Create the specified Namespace
```bash
k create namespace lab3 
```
#### Create a deployment nginx with 3 replicas
1.  we create lab3-deployment.yaml with the following

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
  name: lab3
  namespace: lab3
spec:
  replicas: 3
  selector:
    matchLabels:
      name: lab3
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: lab3
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
```
2. then we apply the yaml file
```bash
k apply -f lab3-deployment.yaml 
k get pod --namespace=lab3 
NAME                    READY   STATUS    RESTARTS   AGE
lab3-76dc9cbb46-n2sf5   1/1     Running   0          61s
lab3-76dc9cbb46-p75zl   1/1     Running   0          61s
lab3-76dc9cbb46-wgkbf   1/1     Running   0          61s
```
#### Expose the deployment on port 80
```bash
k expose deployment lab3 --type=NodePort --name=my-service --namespace=lab3 
k get service -n lab3 
NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
my-service   NodePort   10.98.66.134   <none>        80:30007/TCP   14s
```
---
## Create a CronJob for listing the EndPoints
---
#### Create a **serviceaccount** cronjob-sa
```bash
k create serviceaccount cronjob-sa --namespace=lab3
```
#### Create a Role that allows listing all the services and endpoints
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: cronjob-role
  namespace: lab3
rules:
- apiGroups:
  - ""
  resources: ["services", "endpoints"]
  verbs: ["get", "watch", "list"]
```
#### Link the Role with the created SA 
```bash
kubectl create rolebinding cronjobe-sa-role --role=cronjob-role --serviceaccount=lab3:cronjob-sa --namespace=lab3
```
#### Create a CronJob that lists the endpoints in that namespace every minute and paste the output for the first pod created
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: list-endpoint
  namespace: lab3
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: cronjob-sa
          containers:
          - name: cronjob-list
            image: bitnami/kubectl:latest
            resources: {}
            command:
            - kubectl
            - get
            - endpoints
          restartPolicy: OnFailure
```
##### **logs of the first pod created by the cronjob**
```bash
k logs pods/list-endpoint-27897426-5r7m7 --namespace=lab3

NAME         ENDPOINTS                                AGE
my-service   10.44.0.3:80,10.44.0.4:80,10.44.0.5:80   99s
```
#### After listing try to delete the 3 nginx pods ? again try to view the logs for the newly created pod for that cronJob what do you think happened ? 
- after deleting 3 nginx pods, the deployment created 3 new pods 
- it still has the same **ENDPOINT**
- which means that the pods got the same ip addresses
```bash
k logs pods/list-endpoint-27897429-8ttsl --namespace=lab3

NAME         ENDPOINTS                                AGE
my-service   10.44.0.3:80,10.44.0.4:80,10.44.0.5:80   4m38s
```
- but if we delete the deployment the **ENDPOINT** will be empty
```
NAME         ENDPOINTS   AGE
my-service   <none>      16m
```
