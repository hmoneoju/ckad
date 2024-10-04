# Understanding pods
* Minimum entity managed by Kubernetes
* Pods add Kubernetes properties to containers
  * **nodeSelector** allows selection of the node that runs the Pod
  * **priority** allows for adding priority labels
  * **restartPolicy** tells the cluster what to do if Pod fails
* From a pod one or more containers can be started
* Can also include volumes, which provides storage to containerized applications

```
kubectl explain pod.spec
```

# Standalone pods disadvantages
* Not rescheduled on failure
* Rolling updates do not apply
* Cannot be scaled
* Cannot be replaced automatically
* Used for testing and troubleshooting
* To deploy permanent applications, deployments and StatefulSets are recommended
```
kubectl run podname --image=imagename
kubectl get pods
kubectl describe pod podname
```

# Using Kubernetes the DevOps way
* In DevOps the purpose is to deploy application in a consistent way that is easy to reproduce
* To do so configuration is provided as code
* This works for Kubernetes resources: Instead of running Kubernetes commands, YAML manifest files are used, from which applications can be started
* Creative resources based on YAML files is referred to as the declarative way of working with kubernetes
* The imperative way of working is where resources are created from the command line, using **kubectl** or **kubectl create** (for deployments)

> Tip: In real-life, the declarative way of working is used a lot. On the exam, it may be faster to use the imperative way 

# What is YAML

* Human-readable serialization language
* It uses indentation to identify relations 

# Kubernetes YAML ingredients
* All of the YAML manifest ingredients are defined as resource properties in the API
  * **apiVersion** specifies which version of the API to use for this object
  * **kind** the type of object (Deployment, Pod ...)
  * **metadata** administrative information about the object
  * **spec** specifics to define exactly how the resource is used
  * **status** added by the cluster for running resources
* Use **kubectl explain** to get more information about the basic properties

## pod.spec.containers Components
* Different parts are commonly used:
  * **name** the name of the container
  * **image** the image that should be used
  * **command** the container should run
  * **args** that are used by the command
  * **env** variables that should be used by the container
* They can be checked by **kubectl explain**
```
kubectl explain pod.spec.containers.name
```

# Creating resources from yaml file
* **kubectl create -f resource.yaml** Creates resources from a YAML file. If the resource already exists, it fails
* **kubectl apply -f resource.yaml** creates the resources as defined in the YAML file. If the resource already exists, it updates properties that need updating
* **kubectl delete -f resource.yaml** deletes the resources as defined in the YAML file
* **kubectl replace -f resource.yaml** less commonly used and replaces a current resource with the resource specification as in the YAML file 

# Generating YAML files
* Do not write them, generate them
* To generate YAML files, add **--dry-run=client -o yaml > my.yaml** as an argument to the **kubectl run** and **kubectl create** commands

```
> kubectl run mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml
> less mynginx.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
* Next create the resources in the YAML
```
> kubectl apply -f mynginx.yaml
pod/mynginx created

> kubectl get po
NAME       READY   STATUS    RESTARTS   AGE
mynginx    1/1     Running   0          5s
```

# Multi-Container pods

* The one-container Pod is the standard 
* Single container Pods are easier to build and maintain
* Specific settings can be used that are good for a specific workload
* To create applications that consist of multiple containers, microservices should be used
* In a microservice, different independently managed Pods are connected by the resources that can be provided by Kubernetes
* Defined cases where you might run multiple containers in a Pod:
  * **Sidecar container** enhances the primary application, for instance logging
  * **Ambassador container** represents the primary container to the outside world, such as a proxy
  * **Adapter container** used to adopt the traffic or data pattern to match the traffic or data pattern in other applications in the cluster

## Sidecar Containers
* Provides additional functionality to the main container, where it makes no sense running this functionality in a separate Pod i.e. Logging, monitoring and syncing
* The essence is that the main and the sidecar container have access to shared resources to exchange information
* Istio service mesh is a common addition to Kubernetes that injects sidecar containers in pods for traffic management
* Often, shared volumes are used for this purpose
```
> less multicontainer.yaml 

apiVersion: v1
kind: Pod
metadata: 
  name: multicontainer
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sleep
      - "3600" 
  - name: nginx
    image: nginx

> kubectl apply -f multicontainer.yaml   
```

# Namespaces

* Kubernetes resources leverage Linux kernel namespaces to provide resource isolation
* Different namespaces can be used to strictly separate between customer resources and thus enable multi-tenancy
* Namespaces are used to apply different security-related settings:
  * Role-Based Access Control (RBAC)
  * Quota
* Easier to manage on complex setup

## Managing Namespaces
* To show resources in all namespaces use **kubectl get ... -A**
* To run resources in a namespace **kubectl run ... -n namespace**
* Use **kubectl create ns namespace** to create a ns
```
> kubectl get pods -A
NAMESPACE       NAME                                            READY   STATUS    RESTARTS         AGE
default         firstpod                                        1/1     Running   0                100m
default         mynginx                                         1/1     Running   0                44m
ingress-nginx   ingress-nginx-controller-5bdc4f464b-58ggx       1/1     Running   4 (3d12h ago)    21d
kube-system     coredns-6799fbcd5-zk9lz                         1/1     Running   21 (3d12h ago)   66d
kube-system     local-path-provisioner-6f5d79df6-bhjp6          1/1     Running   21 (3d12h ago)   66d
kube-system     metrics-server-54fd9b65b-xjbx5                  1/1     Running   21 (3d12h ago)   66d
kube-system     svclb-ingress-nginx-controller-fbca3995-plb6v   2/2     Running   30 (3d12h ago)   63d

> kubectl create ns secret
namespace/secret created

> kubectl run pod secretpod --image=nginx -n secret
pod/pod created

> kubectl -n secret get po
NAME   READY   STATUS             RESTARTS      AGE
pod    0/1     CrashLoopBackOff   4 (30s ago)   2m1s

# Failing due to no need to specify resource type, run is for pods

> kubectl -n secret describe po po
Containers:
  pod:
    Container ID:  docker://191d926bfc9825abe6f13c46661986a2ad36cb67831986f5cdbb53f5dc28778f
    Image:         nginx
    Image ID:      docker-pullable://nginx@sha256:d2eb56950b84efe34f966a2b92efb1a1a2ea53e7e93b94cdf45a27cf3cd47fc0
    Port:          <none>
    Host Port:     <none>
    Args:
      secretpod
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127

> kubectl logs -n secret pod

# secretpod interpretted as a variable
/docker-entrypoint.sh: 47: exec: secretpod: not found

> kubectl delete -n secret pod pod
pod "pod" deleted

# Proper way to do things
> kubectl run secret --image=nginx -n secret   
```

# Pods troubleshooting

```
kubectl get pods
kubectl describe pod podname
kubcetl logs podname
kubectl exec -it podname --sh
```

## Passing env variables
```
kubectl run mydb --image=mariadb --env MARIADB_ROOT_PASSWORD=password
```

# Lab

```
> kubectl create ns microservice --dry-run=client -o yaml > lab5.yaml
> kubectl run microdb --image=mariadb --env MARIADB_ROOT_PASSWORD=password -n microservice --dry-run=client -o yaml >> lab5.yaml

> less lab5.yaml

# Add -- to divide definitions
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: microservice
spec: {}
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: microdb
  name: microdb
  namespace: microservice
spec:
  containers:
  - env:
    - name: MARIADB_ROOT_PASSWORD
      value: password
    image: mariadb
    name: microdb
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

> kubcetl apply -f lab5.yaml
```