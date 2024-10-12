# Labels and selectors

* A label is a key-vale pait that is commonly used in Kubernetes
  * Deployments created with **kubectl create deploy** automatically get the label app=deploymentname
  * Pod started with **kubectl run** automatically get the label run=podname
* The purpose of the label is to connect different objects:
  * Deployment tracking pods
  * Services connecting to pods
* The **selector** is used on resources to specify which label to track
* As a command line argument, **--selector** can be used to filter output on the presence of a label

```
> kubectl create deploy webapp --image=nginx --replicas=3

> kubectl get deploy webapp -o yaml | less

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-10-12T15:50:02Z"
  generation: 1
  labels:
    app: webapp
  name: webapp
  namespace: default
  resourceVersion: "1401147"
  uid: b32ba5c0-8c17-4156-874a-68dc6d8215ad
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
  
> kubectl get all --selector app=webapp
NAME                          READY   STATUS    RESTARTS   AGE
pod/webapp-78d6769cf8-44pzm   1/1     Running   0          46s
pod/webapp-78d6769cf8-nzmm6   1/1     Running   0          46s
pod/webapp-78d6769cf8-rcwrj   1/1     Running   0          46s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   3/3     3            3           46s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-78d6769cf8   3         3         3       46s

> kubectl get all --show-labels

NAME                          READY   STATUS    RESTARTS   AGE   LABELS
pod/webapp-78d6769cf8-44pzm   1/1     Running   0          66s   app=webapp,pod-template-hash=78d6769cf8
pod/webapp-78d6769cf8-nzmm6   1/1     Running   0          66s   app=webapp,pod-template-hash=78d6769cf8
pod/webapp-78d6769cf8-rcwrj   1/1     Running   0          66s   app=webapp,pod-template-hash=78d6769cf8

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE     LABELS
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   2m20s   component=apiserver,provider=kubernetes

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/webapp   3/3     3            3           66s   app=webapp

NAME                                DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/webapp-78d6769cf8   3         3         3       66s   app=webapp,pod-template-hash=78d6769cf8
```

# Manage labels

* Use **kubectl label** to manually set labels
* When label is set manually to a Deployment after its creation, it will not be inherited to child resources
* Use **kubectl label key-** to remove the label key with is value

```
> kubectl create deploy bluelabel --image=nginx

> kubectl label deployment bluelabel state=demo

> kubectl get deployments --show-labels
NAME        READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
bluelabel   1/1     1            1           29s   app=bluelabel,state=demo
webapp      3/3     3            3           10m   app=webapp

> kubectl get deployments --selector state=demo

NAME        READY   UP-TO-DATE   AVAILABLE   AGE
bluelabel   1/1     1            1           52s

> kubectl get all --selector app=bluelabel
NAME                             READY   STATUS    RESTARTS   AGE
pod/bluelabel-59d6bcf7ff-gm4r8   1/1     Running   0          70s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bluelabel   1/1     1            1           70s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/bluelabel-59d6bcf7ff   1         1         1       70s

> kubectl describe deployments.apps bluelabel

> kubectl get all --selector app=bluelabel
NAME                             READY   STATUS    RESTARTS   AGE
pod/bluelabel-59d6bcf7ff-gm4r8   1/1     Running   0          4m11s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bluelabel   1/1     1            1           4m11s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/bluelabel-59d6bcf7ff   1         1         1       4m11s

> kubectl label pod bluelabel-59d6bcf7ff-gm4r8 app-
pod/bluelabel-59d6bcf7ff-gm4r8 unlabeled


> kubectl get all --show-labels
 NAME                             READY   STATUS    RESTARTS   AGE     LABELS
pod/bluelabel-59d6bcf7ff-gm4r8   1/1     Running   0          5m53s   pod-template-hash=59d6bcf7ff
pod/bluelabel-59d6bcf7ff-x4r4b   1/1     Running   0          44s     app=bluelabel,pod-template-hash=59d6bcf7ff # Started since the deployment needed a bluelabel

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   17m   component=apiserver,provider=kubernetes

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
deployment.apps/bluelabel   1/1     1            1           5m53s   app=bluelabel,state=demo
deployment.apps/webapp      3/3     3            3           16m     app=webapp

NAME                                   DESIRED   CURRENT   READY   AGE     LABELS
replicaset.apps/bluelabel-59d6bcf7ff   1         1         1       5m53s   app=bluelabel,pod-template-hash=59d6bcf7ff
replicaset.apps/webapp-78d6769cf8      3         3         3       16m     app=webapp,pod-template-hash=78d6769cf8

```

# Understanding annotations

* Used to store additional information in resources
* This can be generic information, like notes
* Annotations are also used by certain Kubernetes commands to store auto-managed information on the resource
* Annotations are set automatically but you can use **kubectl annotate deploy notes newnote="mynote"** 

```
> kubectl create deploy notes --image=nginx --dry-run=client -o yaml > notes.yaml
> kubectl apply -f notes.yaml
> kubectl get deploy notes -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"notes"},"name":"notes","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"notes"}},"strategy":{},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"notes"}},"spec":{"containers":[{"image":"nginx","name":"nginx","resources":{}}]}}},"status":{}}
  creationTimestamp: "2024-10-12T16:10:53Z"

```

# Scaling application manually

* Deployments use ReplicaSet to manage the expected number of application instances
* When an application is started using **kubectl create deploy ... --replicas=3** the ReplicaSet ensures the desired number of instances are running
* To manually manage application scalability, use **kubectl scale deploymentname --replicas=3**

```
> kubectl create deployment scaledapp --image=nginx --replicas=3 --dry-run=client -o yaml > scaledapp.yaml
> kubectl apply -f scaledapp.yaml
> kubectl get all --selector app=scaledapp
NAME                             READY   STATUS    RESTARTS   AGE
pod/scaledapp-6487f69d86-5bcrx   1/1     Running   0          23s
pod/scaledapp-6487f69d86-ftpns   1/1     Running   0          23s
pod/scaledapp-6487f69d86-tvznb   1/1     Running   0          23s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/scaledapp   3/3     3            3           23s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/scaledapp-6487f69d86   3         3         3       23s

> kubectl delete po scaledapp-6487f69d86-tvznb
> kubectl scale deployment scaledapp --replicas=2
```

# Deployment updates

* Deployments make updating applications easier
* To manage how applications are updated, an update strategy is used
  * **strategy.type.rollingUpdate** updates application instances in batches to ensure application functionality continues to be offered at any time
  * As a result of rollingUpdate, during the update different versions of the application will be running
  * For applications that don't support offering multiple versions simultaneously, set **strategy.type.recreate**
  * The recreate strategy brings down all application instances, after which the new application version is brought up

# Managing rolling updates

* To manage how rollingUpdate will happen, two parameters are used:
  * **maxSurge** specifies how many application instances cab be running during the update above the regular number of application instances
  * **maxUnavailable** defines how many application instances can be temporary unavailable
* Both parameters take an absolute number or a percentage as their argument

```
> kubectl create deploy upapp --image=nginx:1.17 --replicas=5 --dry-run=client -o yaml > upapp.yaml

> kubectl apply -f upapp.yaml
deployment.apps/upapp created

> kubectl get all --selector app=upapp
NAME                         READY   STATUS    RESTARTS   AGE
pod/upapp-6d679db6b4-8ssjd   1/1     Running   0          43s
pod/upapp-6d679db6b4-dtbdv   1/1     Running   0          43s
pod/upapp-6d679db6b4-lqblz   1/1     Running   0          43s
pod/upapp-6d679db6b4-q8952   1/1     Running   0          43s
pod/upapp-6d679db6b4-zv2tq   1/1     Running   0          43s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/upapp   5/5     5            5           44s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/upapp-6d679db6b4   5         5         5       44s


> kubectl get deployments.apps upapp -o yaml | grep -A5 strategy
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"upapp"},"name":"upapp","namespace":"default"},"spec":{"replicas":5,"selector":{"matchLabels":{"app":"upapp"}},"strategy":{},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"upapp"}},"spec":{"containers":[{"image":"nginx:1.17","name":"nginx","resources":{}}]}}},"status":{}}
  creationTimestamp: "2024-10-12T16:57:08Z"
  generation: 1
  labels:
    app: upapp
  name: upapp
--
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
  
> kubectl set image deploy/upapp nginx=nginx:1.18

> kubectl get all --selector app=upapp
NAME                         READY   STATUS              RESTARTS   AGE
pod/upapp-6d679db6b4-8ssjd   1/1     Running             0          4m26s
pod/upapp-6d679db6b4-dtbdv   1/1     Running             0          4m26s
pod/upapp-6d679db6b4-lqblz   1/1     Running             0          4m26s
pod/upapp-6d679db6b4-q8952   1/1     Running             0          4m26s
pod/upapp-8d6765999-njhf6    0/1     ContainerCreating   0          4s
pod/upapp-8d6765999-wcmrj    0/1     ContainerCreating   0          4s
pod/upapp-8d6765999-xdv6l    0/1     ContainerCreating   0          4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/upapp   4/5     3            4           4m26s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/upapp-6d679db6b4   4         4         4       4m26s
replicaset.apps/upapp-8d6765999    3         3         0       4s

>  kubectl edit deployments.apps upapp
>  kubectl set image deploy/upapp nginx=nginx:1.19

 kubectl get all --selector app=upapp
NAME                        READY   STATUS              RESTARTS   AGE
pod/upapp-dc68b4f8b-bfbzt   0/1     ContainerCreating   0          2s
pod/upapp-dc68b4f8b-mbk8d   0/1     ContainerCreating   0          2s
pod/upapp-dc68b4f8b-mzgjf   0/1     ContainerCreating   0          2s
pod/upapp-dc68b4f8b-xp88p   0/1     ContainerCreating   0          2s
pod/upapp-dc68b4f8b-zwnpr   0/1     ContainerCreating   0          2s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/upapp   0/5     5            0           11m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/upapp-6d679db6b4   0         0         0       11m
replicaset.apps/upapp-8d6765999    0         0         0       6m58s
replicaset.apps/upapp-dc68b4f8b    5         5         0       2s


```  

# Deployment History

* During the deployment update procedure, the Deployment creates a new ReplicaSet that uses the new properties
* The old replica set is kept, but the number of pods will be set to 0
* This makes easy to roll back to the previous state
* **kubectl rollout history** will show the rollout history of a specific deployment, which can easily be reverted as well
* Use **kubectl rollout history deployment mynginx --revision=1**

```
> kubectl rollout history deployment
deployment.apps/scaledapp
REVISION  CHANGE-CAUSE
1         <none>

deployment.apps/upapp
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>

> kubectl rollout history deployment upapp --revision=2

# Rollback 
> kubectl rollout undo deployment upapp --to-revision=1

```

# StatefulSet

* StatefulSet maintains the identity of Pods, even if they are restarted
* This is required by stateful applications, like databases
* Useful when one of the following are required:
  * Stable, unique network identifiers
  * Stable, persistent storage
  * Ordered, graceful deployment and scaling
  * Order and automated rolling updates
* Easier when using Helm Charts

# StatefulSet Workings

* Storage must be provisioned by a PersistentVolume and will be claimed by the StatefulSet volumeClaimTemplate, which generates a PVC
* Deleting a StatefulSet does not delete associated volumes
* Requires a headless Service
* While creating the Pods, every nex Pod copies the configuration of the Pod before and adds its unique identity
* Pods are created one by one, and it will take a while before all Pod instances are available
* It may be required to scale down the StatefulSet replicas to 0 prior to deletion

# StatefulSet Storage

* The StatefulSet volumeClaimTemplate generates a PVC that connects to a PV
