# Running application from yaml files

* A kubernetes application often is a collection of resources
* To manage applications in a consistent way, all related resources can be defined ine one YAML file
* While doing so, it is recommended to use clear separators between application components
* Although this works, the solution is not very portable
* To make an application management more flexible, site specific values should be separated from generic application configuration

# The Helm package manager

* Helm is a k8s package manager and is used to streamline installing and managing k8s applications
* Helm consist of the **helm** tool, which needs to be installed, and a chart
* A chart is a Helm package, which contains thw following:
  * A description of the package
  * One or more templates containing k8s manifest files
* Charts can be stored locally, or accessed from remote Helm repositories

# Helm charts

* Helm doesn't come with a default repository
* The main site for finding Helm charts, is through https://artifacthub.io
* Search for specific software here, and run the commands to install it, for instance, to run the Kubernetes Dashboard:
```
  > helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard
  > helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard
```
* The last command creates the local application name "kubernetes-dashboard" by installing the application "kubernetes-dashboard" from the repo "kubernetes-dashboard" 

## Example

```
> helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

> helm repo list
NAME          	URL
bitnami       	https://charts.bitnami.com/bitnami

> helm search repo bitnami # searches for the word bitnami 
NAME                                             	CHART VERSION	APP VERSION  	DESCRIPTION
helm-bitnami/bitnami-common                      	0.0.9        	0.0.9        	DEPRECATED Chart with custom templates used in ...
helm-releases/kafka-bitnami                      	26.11.4      	3.6.1        	Apache Kafka is a distributed streaming platfor...
helmBitnami/bitnami-common                       	0.0.9        	0.0.9        	DEPRECATED Chart with custom templates used in ...
helmReleases/kafka-bitnami                       	26.11.4      	3.6.1        	Apache Kafka is a distributed streaming platfor...
nexus-releases/kafka-bitnami                     	26.11.4      	3.6.1        	Apache Kafka is a distributed streaming platfor...
bitnami/airflow                                  	21.0.3       	2.10.2       	Apache Airflow is a tool to express and execute...
bitnami/apache                                   	11.2.22      	2.4.62       	Apache HTTP Server is an open-source HTTP serve...
bitnami/apisix                                   	3.5.2        	3.11.0       	Apache APISIX is high-performance, real-time AP...
bitnami/appsmith                                 	5.0.4        	1.47.0       	Appsmith is an open source platform for buildin...
...

> helm search repo file # searches for the word file
NAME                        	CHART VERSION	APP VERSION 	DESCRIPTION
helm-releases/filebeat      	7.17.3       	7.17.3      	Official Elastic helm chart for Filebeat
helm-snapshots/sncr-filebeat	7.17.3       	7.17.3      	A Helm chart for the Filebeat configuration in ...
helmReleases/filebeat       	7.17.3       	7.17.3      	Official Elastic helm chart for Filebeat
...

> helm search repo nginx --versions # shows different versions 
```

# Installing Helm Charts
* After adding repositories, use **helm repo update** to ensure access to the most up-to-date information
* Use **helm install <name> <chart>** to install the chart with default parameters
  * Notice that a chart may be installed multiple times, which is why it's important to assign the right name
* After installation, use **helm list** to list currently installed charts
* Optionally, use **helm delete** to remove currently installed charts

```
> helm install bitnamy/mysql --generate-name
NAME: mysql-1730627562
LAST DEPLOYED: Sun Nov  3 09:52:44 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 11.1.19
APP VERSION: 8.4.3
.....

> kubectl get all
 
NAME                     READY   STATUS    RESTARTS   AGE
pod/mysql-1730627562-0   0/1     Running   0          31s

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                  ClusterIP   10.43.0.1      <none>        443/TCP    20d
service/mysql-1730627562            ClusterIP   10.43.90.127   <none>        3306/TCP   31s
service/mysql-1730627562-headless   ClusterIP   None           <none>        3306/TCP   31s

NAME                                READY   AGE
statefulset.apps/mysql-1730627562   0/1     31s

> helm show chart bitnami/mysql

annotations:
  category: Database
  images: |
    - name: mysql
      image: docker.io/bitnami/mysql:8.4.3-debian-12-r0
    - name: mysqld-exporter
      image: docker.io/bitnami/mysqld-exporter:0.15.1-debian-12-r35
    - name: os-shell
      image: docker.io/bitnami/os-shell:12-debian-12-r31
  licenses: Apache-2.0
...  

> helm list

NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART        	APP VERSION
mysql-1730627562	default  	1       	2024-11-03 09:52:44.036233 +0000 UTC	deployed	mysql-11.1.19	8.4.3

> helm status mysql-1730627562
```

# Managing applications with helm

* A Helm chart consists of templates to which specific values are applied
* The values are stored in the values.yaml file, within the Helm chart
* Use **helm show values** to list current values (a lot!)
* Read documentation on artifacthub.io for a list of all values
* Create a custom values.yaml which will be merged with the generic values.yaml file: **helm install ... -values values.yaml**
* When upgrading an application, you'll have to use **--values values.yaml** as well

# Overriding values

* The default values.yaml file which is a part of the Helm chart contains default values which are defined as a key-value pair
* While installing the chart, you can use a custom values.yaml file, provided with the --values values.yaml as an argument to helm install to overwrite the default values
* Alternatively, use **helm install .. --set-key-value** to set individual values
* Best practice: use a values.yaml file and only use --set when absolutely necessary

```
> helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" already exists with the same configuration, skipping

> helm show values bitnami/nginx | grep commonLabels
## @param commonLabels Add labels to all the deployed resources
commonLabels: {}

> helm show values bitnami/nginx | grep replicaCount
## @param replicaCount Number of NGINX replicas to deploy
replicaCount: 1

> helm install bitnami/nginx --generate-name --values values.yaml

> helm list
helm list
NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART        	APP VERSION
mysql-1730627562	default  	1       	2024-11-03 09:52:44.036233 +0000 UTC	deployed	mysql-11.1.19	8.4.3
nginx-1730629079	default  	1       	2024-11-03 10:18:00.351318 +0000 UTC	deployed	nginx-18.2.4 	1.27.2

> helm get values nginx-1730629079
USER-SUPPLIED VALUES:
commonLabels: 'type: helmapp'
replicaCount: 3

> helm get values --all nginx-1730629079
```

# Helm upgrades

* A Helm chart is the package you install from the Helm repository
* A Helm installation is an installed instance of that package
* A Helm release is a combination between a chart and a installation
* You can upgrade either of these
* For instance, to first run a nginx installation without exposing it. use **helm install bitginx bitnami/nginx --set ingress.enabled=false**
* Nest to make the application accessible, use **helm update bitginx bitnami/nginx --set ingress.enabled=true**
* Use **helm repo update** to fetch the latest version of charts from the repository 
* Use the **helm upgrade bitginx bitnami/nginx** to upgrade to the latest version of the application

# Managing application 