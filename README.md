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
### because the desired in the replica sit is 4 
## 8-Create a ReplicaSet using the below yaml
```
apiVersion: v1 ---> apps/v1

replicaset-1-7rzvm      1/1     Running            0          12s
replicaset-1-zwdzs      1/1     Running            0          12s

```