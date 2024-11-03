# Services

* A Service is an API resource that is used to expose a set of Pods
* Services are applying round-robin load balancing to forward traffic to specific pods
* The set of pods that is targeted by a Service is determined by a selector (which is a label)
* The kube-controller-manager will continuously scan for Pods that match the selector and include these in the Service
* If Pods are added or removed, they immediately show up in the Service

# Services and Decoupling

* Services exist independently of the applications they provide access to
* Services needs to be created independently of the application and after removing an application, it also needs to be removed separately 
* The only thing they do is watch for Pods that have a specific labe; set matching the selector that is specified in the service
* That means that one Service can provide access to Pods in multiple Deployments, and while doing so, k8s will automatically load balance between these pods
* This strategy is used in canary Deployments

# Service types

* **ClusterIP** - this default type exposes the service on an internal cluster IP address
* **NodePort** - allocates a specific port on the node that forwards to the service IP address on the cluster network
* **LoadBalancer** - provisions an external load balancer to handle incoming traffic to applications in public cloud
* **ExternalName** - works on DNS names; redirection us happening at a DNS level, which is useful in migration
* **Headless** - a Service used in cases where direct communication with Pods is required, which is used in StatefulSet

> NOTE: For CKAD focus on ClusterIP and NodePort


# Creating services

* **kubectl expose** can be used to create Services, providing access to Deployments, ReplicaSets, Pods or other Services
* In most cases **kubectl expose** exposes a Deployment, which allocates its Pods as the Service endpoint
* **kubectl create service** can be used as an alternative solution to create Services
* While creating a Servicem the --port argument must be specified to indicate the port on which the Service will be listening for incoming traffic

# Service Ports

* While working with Services, different ports are specified:
  * **targetPort** the port on the application (container) that the service addresses
  * **port** the port on which the Service is accessible
  * **nodePort** the port that is exposes externally while using the **NodePort** Service type

# Example of service creation

```
> kubectl create deployment nginxsvc --image=nginx
deployment.apps/nginxsvc created

> kubectl scale deployment nginxsvc --replicas=3
deployment.apps/nginxsvc scaled

> kubectl expose deployment nginxsvc --port=80
service/nginxsvc exposed

> kubectl describe svc nginxsvc # look for endpoints
Name:              nginxsvc
Namespace:         default
Labels:            app=nginxsvc
Annotations:       <none>
Selector:          app=nginxsvc
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.239.117
IPs:               10.43.239.117
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.42.0.239:80,10.42.0.240:80,10.42.0.241:80
Session Affinity:  None
Events:            <none>

> kubectl get svc nginxsvc -o=yaml
Name:              nginxsvc
Namespace:         default
Labels:            app=nginxsvc
Annotations:       <none>
Selector:          app=nginxsvc
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.239.117
IPs:               10.43.239.117
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.42.0.239:80,10.42.0.240:80,10.42.0.241:80
Session Affinity:  None
Events:            <none>
...

> kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP   4m22s
nginxsvc     ClusterIP   10.43.239.117   <none>        80/TCP    91s

> kubectl get endpoints
NAME         ENDPOINTS                                      AGE
kubernetes   192.168.64.2:6443                              4m44s
nginxsvc     10.42.0.239:80,10.42.0.240:80,10.42.0.241:80   113s

> kubectl get all --selector app=nginxsvc 
NAME                            READY   STATUS    RESTARTS   AGE
pod/nginxsvc-69b846bcff-42z8f   1/1     Running   0          4m34s
pod/nginxsvc-69b846bcff-njdbt   1/1     Running   0          4m56s
pod/nginxsvc-69b846bcff-w5x2m   1/1     Running   0          4m34s

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginxsvc   ClusterIP   10.43.239.117   <none>        80/TCP    4m2s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginxsvc   3/3     3            3           4m56s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginxsvc-69b846bcff   3         3         3       4m56s

> kubectl edit svc nginxsvc
spec:
  type: NodePort
  
> kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP        13m
nginxsvc     NodePort    10.43.239.117   <none>        80:31730/TCP   10m

> curl localhost:31730

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
  
```

> NOTE: Ingress and gateways only work with HTTP (ClusterIP), to provide access to something else use NodePort instead

# Understanding microservices

* In a microservices architecture, different frontend and backend Pods are used to provide the application
* Backend Pods (like databases) should be exposed internally only, using the ClusterIP Service type
* Frontend Pods (like webservers) should be exposed for external access, using the NodePort Service type or the Ingress resource
* For more advanced traffic management in microservices, a service mesh can be used

# NetworkPolicy

* By default, there are no restrictions to network traffic in k8s
* Pods can always communicate, even is they are in other namespaces
* To limit this, NetworkPolicies can be used
* NetworkPolicies need to be supported by the network plugin though, 
  * The Weave plugin DOES not support network policies
  * Calico is a common plugin that does support NetworkPolicy 
* If in a policy there is no match traffic will be denied
* If no NetworkPolicy is used, all traffic is allowed

# NetworkPolicy identifiers

* In NetworkPolicy, three identifiers can be used:
  * podSelector: specifies a label to match Pods
  * namespaceSelector: used to grant access to specific namespaces
  * ipBlock: marks a range of IP addresses that is allowed, notice that traffic to and from the node where a Pod is running is always allowed 
* When defining a Pod or Ns based NetworkPolicy, a selector label is used to specify what traffic is allowed to and from the Pods that match the selector
* NetworkPolicies do not conflict, they are additive

```
>  kubectl apply -f nwpolicy-complete-example.yaml

networkpolicy.networking.k8s.io/access-nginx created
pod/nginx created
pod/busybox created

> kubectl expose pod nginx --port=80
service/nginx exposed

> kubectl exec -it busybox -- wget --spider --timeout=1 nginx
Connecting to nginx (10.43.107.199:80)
wget: download timed out
command terminated with exit code 1

> kubectl describe networkpolicy access-nginx
Name:         access-nginx
Namespace:    default
Created on:   2024-11-03 13:25:03 +0000 GMT
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=nginx
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: access=true
  Not affecting egress traffic
  Policy Types: Ingress

> kubectl label pod busybox access=true
pod/busybox labeled

> kubectl exec -it busybox -- wget --spider --timeout=1 nginx
Connecting to nginx (10.43.107.199:80)
remote file exists
```

> NOTE: for the exam if a Pod cannot access another describe the network policies and ensure the pod has the proper label/selector 

# What is Gateway API

* Gateway API Provides routing and traffic management policies
* It is advanced layer that uses custom resources for managing incoming (ingress) and outgoing (egress) traffic
* Adds features that are not addressed by Ingress, including:
  * Support for TCP, UDP and gRPC
  * Traffic splitting and mirroring
  * Websockets protocols
* Future deployments seem to further integrate Gateway API functionality with Ingress

# What is Istio

* Istio is a ServiceMesh and makes managing complex relations between applications in a microservice easier
* It provides rules for managing traffic in microservices
* It includes features for traffic management, security and observability in the service mesh
* Its focus is on service-to-service communication
* Istio may be used in addition to core Kubernetes networking and is optional
* It makes sense in complex environments


