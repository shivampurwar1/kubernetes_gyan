# Kubernetes

## DaemonSets
DaemonSet ensures that all Nodes run a copy of a specific Pod. 
As nodes are added or removed from the cluster, a DaemonSet will add or remove the required pods. 
Deleting a DaemonSet will also clean up all the pods that it created. 
Use case: To run a single log aggregator or monitoring agent on a node. (Monitoring or Logging agent)
nodeSelector -> Specify if you want to run on specific nodes (ex: minikube, etc)
```
kubectl create -f 201-daemonset-example.yaml
kubectl get daemonsets
kubectl get pods

[Testing]
kubectl get node --show-labels
kubectl label node/<node-name> <new-label>=<value> --overwrite   # ex: env=dev
kubectl get nodes --show-labels

kubectl create -f 201-daemonset-dev-label.yaml
kubectl get daemonsets     # Observer node selector with same label

kubectl create -f 201-daemonset-prod-label.yaml
kubectl get daemonsets     # Observer desired, current, ready, up to date and available with all zeros. Because we do not have any nodes with such label

kubectl delete -f 201-daemonset-example.yaml
kubectl delete -f 201-daemonset-dev-label.yaml
kubectl delete -f 201-daemonset-prod-label.yaml

kubectl label node/<node-name> <new-label>-    # ex: env-
```

## Stateful sets
It is helpful for StatefulApp (applications that stores data) or databases ex: ES, mongoDB, mySQL.
Manages the deployment and scaling of a set of Pods. 
Provides guarantees about the ordering and uniqueness of these Pods. 
StatefulSet maintains a sticky identity for each of their Pods.
Readmore: https://kubernetesbyexample.com/statefulset/
Statefull app update data based on previous state. query data. (depends on most up-to-date data/state). deployed using StatefulSet.
Note: Stateless app don;t keep record of state and each request is completely new. It doesn't depend on previous data. (passthrough for data query/update). Deployed using Deployment.
Replicating stateful application is more difficult.
- Cannot be created/deleted at ame time.
- Cannot be randomly addressed.
- Replica Pods are not identical.  (Pod identity) 

Pod Identity 
- Sticky identity for each pod.
- created from same specification, but not interchangeable
- persistent identifier across any  re-scheduling.
Pod dies and replace, replaced pod keeps the identity

Why this identity imp?
To avoid data inconsistency. 

Ex: Mysql (Master + 2 or more worker node)
Write will happen at one node, read can happen at all nodes.
All using the same data but physical storage is not  same.
Worker continuously synchronizing the data.

If new replica joins, then it take care of its own storage and clone data from Previous node.
Once data is upto date continuous syncronization happen.

Data will be lost when all pods dies so data persistence is imp for database.
So use PV for stateful set. because it's lifecycle isn't tied to other component's lifecycle.
Pod state (master, slave, etc) and other info stored in PV. 
If a pod dies and a new one added, PV identifier makes sure it attaches to the new pod.
To let this happen use Remote storage. Because if new pod created in different node then it should be available.
Other Difference in Deployment (random hash) and StatefulSet (fixed ordered names)

In StatefulSet new pod is only create, if previous is up and running.
Deletion happens in reverse order, starting from the last one.
It helps to protect data and state.

Two Endpoints:
1. Loadbalancer service 
2. Individual service name  <pdo-name>.<governing-service-domain>

Pods in StatefulSet has two characteristics:
1. Predictable pod name
2. Fixed individual DNS name
So when pod restarts: IP change but name and endpoint stays same.
Sticy identity: Retain state and Retain role.

Replicating stateful apps:
It's complex, Kubernetes helps you but You still need to do a lot:
- configuring te cloning and data synchronization
- make remote storage available
- managing and back-up

Note: Stateful applications not perfect for containerized environments. (stateless applications are more suitable.)
```
#201-usecase-zookeeper-statefulset.yaml
kubectl get statefulsets 
```

## Logs
Applications running in a pod can write to standard out (stdout). Kubernetes will automatically pick up the logs and show you on kubectl logs command.
This is good for development environment.
```
kubectl logs <pod-name>
kubectl logs --tail=5 <pod-name> -c <container-name> # To view the five most recent log lines of container
kubectl logs -f --since=10s <pod-name> -c <container-name> # tail -f behaviour
kubectl logs -p <pod-name> -c <container-name>  # logs of pods that have already completed their lifecycle. with -p option you can print the logs for previous instances of the container in a pod
```
For application running in higher environment use a logging platform like Kibana with Elasticsearch. 
In this case logs being shipped to it from pods, using Fluentd, Filebeat or Logstash. 

## Monitoring

Monitor
- Node health
- Health of Kubernetes
- Application metrics and health

Open source projects like cAadvisor and Prometheus can  help in monitoring.
Grafana can be used to visualize. 
Enterprise tool like Datadog or Splunk can also be used.

## Authentication and Authorization
Authentication: Does a user have access to the system?
Authorization: Can the user perform an action in the system?
Users -> Normal, Service Account

## Port Forward
Access a service from your local environment without exposing it using, for example, a load balancer or an ingress resource.
You can use port forwarding.
```
kubectl create -f 101-service.yaml
kubectl port-forward service/<service-name>  <local-ports>:<service-port>    # 8085:8082
curl localhost:8085
# Browse localhost:8085
kubectl delete -f 101-service.yaml
```

## Init Container
It’s sometimes necessary to prepare a container running in a pod.
Kubernetes will execute all init containers (and they must all exit successfully) before the main container(s) are executed.
ex: init some data in a database, wait for a service being available
```
kubectl create -f 201-init-container.yaml 
kubectl get pod
kubectl logs <pod-name>
kubectl delete -f 201-init-container.yaml
```

## Accessing API Server
For exploratory or testing purposes you can access API server directly.
```
kubectl get --raw /api/v1   # without proxy

kubectl api-versions       # supported API versions/resources
kubectl api-resources       # supported API versions/resources

kubectl proxy --port=8080     # proxy the API to local environment
curl http://localhost:8080/api/v1

```

# Realworld Examples (Usecase)

## PHP Guestbook using Redis DB
https://kubernetes.io/docs/tutorials/stateless-application/guestbook/
You can enter guest users in the app.
```
kubectl create -f 201-usecase-guestbook-redis.yaml
kubectl get service 
# minikube service <service-name>
Note: Browse the web pager
kubectl delete -f 201-usecase-guestbook-redis.yaml
```

## PHP Guestbook using Mongo DB
https://kubernetes.io/docs/tutorials/stateless-application/guestbook/
```

```

## MongoExpress 
ConfigMap used to store DB service name.
Secret used to store the DB secrets.
ConfigMap and Secret passed as Environment variable.
Mongo image can take variable at run time. 
Secret must be created before the deployment.
```
kubectl create -f 201-usecase-mongoexpress.yaml
kubectl get service 
Note: Browse the web pager
kubectl delete -f 201-usecase-mongoexpress.yaml
```

## Kubernetes Dashboard
https://github.com/kubernetes/dashboard      \
- Monitor and visualize clusters from an operational perspective.
- Manage and troubleshoot running applications. 
```
minikube addons list              # Returns all of the different addons.
minikube addons enable dashboard  # Enable dashboard addons
minikube addons enable metrics-server # Enable metrics-server addons. It will help to enable metrics.
minikube dashboard
```
Overview gives information about clusters.
You can change the namespaces. 
- Kube-system will have all the system information. 

Workload:
- Cron Jobs
- Deployment, etc. 

In Deployments: 
- Edit deployment file (not recommended)
- Scale resources (...  -> three dots)    (Refer Pod section and Replicaset section to see changes) 

In Pods: 
- Exec into pod
- View logs

## Pull image from private repo
```
aws ecr get-login      # print full docker login command for aws ecr
docker login -u username -p password 
cat .docker/config.json | base64

# create docker login secret from config.json file
kubectl create secret generic my-registry-key \
--from-file=.dockerconfigjson=.docker/config.json \
--type=kubernetes.io/dockerconfigjson

kubectl create secret generic my-registry-key --from-file=.dockerconfigjson=.docker/config.json --type=kubernetes.io/dockerconfigjson

kubect get secret

# create docker login secret with login credentials
kubectl create secret docker-registry my-registry-key \
--docker-server=https://private-repo \
--docker-username=user \
--docker-password=pwd

kubectl create secret docker-registry my-registry-key --docker-server=https://private-repo --docker-username=user --docker-password=pwd


minikube ssh      # access minikube console

# copy config.json file from Minikube to my host
scp -i $(minikube ssh-key) docker@$(minikube ip):.docker/config.json .docker/config.json

# refer: 201-pull-image-from-private-repo
```

# Extra

## Mosquitto
```
kubectl create -f 201-usecase-temp-mosquitto.yaml
kubectl get pod
kubectl logs <pod-name>
kubectl delete -f 201-usecase-temp-mosquitto.yaml
```

Statefulset Cassandra:
https://kubernetes.io/docs/tutorials/stateful-application/cassandra/

StatefulApp
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

Role and Role binding
https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-rbac

Storage
https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-storage

Jobs
https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/

Log analysis
https://logz.io/blog/kubernetes-log-analysis/

Monitoring:
https://kubernetes.io/blog/2017/05/kubernetes-monitoring-guide/

RBAC:
https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/

Prometheus Exporter
https://gitlab.com/nanuchi/youtube-tutorial-series/-/tree/master/prometheus-exporter
Prometheus reporter: https://gitlab.com/nanuchi/youtube-tutorial-series/-/blob/master/setup-prometheus-operator/commands.md