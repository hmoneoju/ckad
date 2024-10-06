# Init Containers

* Special case for a multi-container Pod, where the init container runs to completion before the main container is started
* Main container depends on the success of the init container, if it fails main won't start
```
> kubectl apply -f init_containers.yaml
> kubectl get pods

```

# Using port forwarding to access pods

* Pods can be accessed in multiple ways
* Expose a Pod port on the kubectl host that forwards to the Pod
  * **kubectl port-forward fwnginx 8080:80**
* Useful for testing Pod accessibility
```
> kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE                   NOMINATED NODE   READINESS GATES
fwnginx     1/1     Running   0          56s     10.42.0.58   lima-rancher-desktop   <none>           <none>
init-demo   1/1     Running   0          9m44s   10.42.0.57   lima-rancher-desktop   <none>           <none>

> kubectl run fwnginx --image=nginx
> kubectl port-forward fwnginx 8080:80
> curl localhost:8080
```

# RestartPolicy

* Attribute that determines what happens if a container that is managed by a Pod crashes
* It defaults to **restartPolicy=always**, the container will be restarted after crash
* always policy does not affect the state of the entire Pod, if the Pod is stopped or killed this policy won't restart it
```
kubectl run nginx1 --image=nginx

# Print yaml code behind the application 
kubectl get pods nginx1 -o yaml

kubectl delete pods nginx1

kubectl run nginx2 --image=nginx

# Container runtime interface, for running containers
crictl ps 
crictl stop 5bb
```

# Jobs
* Starts a Pod with a **restartPolicy** set to never
* To create a Pod that runs to completion, use Jobs instead
* Useful for one-shot tasks, backup, calculation, batch processing
* Use **spec.ttlSecondsAfterFinish** to clean up completed jobs automatically

## Job types
* 3 different can  beb started, which is specified by the completions and parallelism parameters:
  * **Non-paralell Jobs** one Pod is started unless the Pod fails
    * completion=1
    * parallelism=1
  * **Parallel Jobs with a fixed completion count** The job is complete  after successfully running as many times as specified in **jobs.spec.completions** 
    * completions=n
    * parallelism=m
  * **Parallel Jobs with a work queue** Multiple jobs are started, when one completes successfully, the Job is complete
    * completions=1
    * parallelism=n

```
# Provides running examples
> kubectl create job -h | less

Examples:
  # Create a job
  kubectl create job my-job --image=busybox

  # Create a job with a command
  kubectl create job my-job --image=busybox -- date

  # Create a job from a cron job named "a-cronjob"
  kubectl create job test-job --from=cronjob/a-cronjob
  
....

> kubectl create job onejon --image=busybox -- date

> kubectl get jobs,pods
NAME               COMPLETIONS   DURATION   AGE
job.batch/onejon   1/1           7s         65s

NAME               READY   STATUS      RESTARTS   AGE
pod/fwnginx        1/1     Running     0          30m
pod/init-demo      1/1     Running     0          39m
pod/nginx2         1/1     Running     0          21m
pod/onejon-wtsx5   0/1     Completed   0          65s

> kubectl get pods onejon-wtsx5 -o yaml | grep restartP
  restartPolicy: Never
  
> kubectl delete job onejon
> kubectl create job mynewjob --image=busybox --dry-run=client -o yaml -- sleep 5 > mynewjob.yaml

> kubectl get jobs,pods
NAME                 COMPLETIONS   DURATION   AGE
# After 60 seconds since last completion it will be removed
job.batch/mynewjob   3/3           28s        29s 

NAME                 READY   STATUS      RESTARTS   AGE
pod/fwnginx          1/1     Running     0          37m
pod/init-demo        1/1     Running     0          46m
pod/mynewjob-6pd56   0/1     Completed   0          29s
pod/mynewjob-7twc9   0/1     Completed   0          10s
pod/mynewjob-dxf42   0/1     Completed   0          19s
pod/nginx2           1/1     Running     0          28m
```

# Cronjobs

* Jobs are used to run a task a specific number of times, while cron jobs are scheduled
* To add the schedule, Linux crontab syntax is used
* When running a cronjob, a job will be scheduled
* This job, on its run, will start a Pod
* To test a Cronjon, use **kubectl create job myjob --from=cronjob/mycronjob**

```
> kubectl create cronjob -h | less
Create a cron job with the specified name.

Aliases:
cronjob, cj

Examples:
  # Create a cron job
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"

  # Create a cron job with a command
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" -- date
  
> kubectl create cronjob runme --image=busybox --schedule="*/2 * * * *" -- echo Greetings from the cluster  

> kubectl get cronjobs,jobs,pods
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/runme   */2 * * * *   False     0        14s             30s

NAME                       COMPLETIONS   DURATION   AGE
job.batch/runme-28803534   1/1           4s         14s

NAME                       READY   STATUS      RESTARTS   AGE
pod/fwnginx                1/1     Running     0          52m
pod/init-demo              1/1     Running     0          60m
pod/nginx2                 1/1     Running     0          42m
pod/runme-28803534-5n5gg   0/1     Completed   0          14s

> kubectl create job runme --from=cronjob/runme
> kubectl logs pod/runme-4k4vp
Greetings from the cluster

> kubectl delete cronjobs.batch runme
cronjob.batch "runme" deleted

> kubectl get cronjobs,jobs,pods
NAME            READY   STATUS    RESTARTS   AGE
pod/fwnginx     1/1     Running   0          55m
pod/init-demo   1/1     Running   0          64m
pod/nginx2      1/1     Running   0          46m
  
```

# Cleaning up resources
* Resource not cleanup automatically
* If a pod is managed by deployments, the deployment must be removed, not the Pod
* Try not to force resource to deletion, it may bring them in an unmanageable state

```
> kubectl delete all
error: resource(s) were provided, but no name was specified

> kubectl delete all --all
pod "fwnginx" deleted
pod "init-demo" deleted
pod "nginx2" deleted
service "kubernetes" deleted
deployment.apps "firstapp" deleted
replicaset.apps "firstapp-6bf8f798f8" deleted

> kubectl delete all --all -- force --grace-period=-1
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
service "kubernetes" force deleted

> kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   44s
```

# Lab
* If after 15s 

```
> kubectl explain cronjob.spec | less
> kubectl explain cronjob.spec.jobTemplate.spec | less
> kubectl create cronjob -h | less
> kubectl create cronjob sleepy --image=busybox --schedule="* * * * *" --dry-run=client -o yaml -- sleep 30 > lab6.yaml
> kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   6m53s

NAME                   SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/sleepy   * * * * *   False     0        <none>          4s

```