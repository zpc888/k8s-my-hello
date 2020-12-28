# minikube installation

* install conntract, which is needed by minikube

```shell
sudo apt-get install -y conntrack
```

* install kubectl

```sh
snap install kubectl --classic
```

* install mimikube

```shell
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 -O minikube
chmod 755 minikube
mv minikube /usr/local/bin/
```

* verification

```shell
minikube version
minikube start --vm-driver=none
minikube status
minikube service list
kubectl cluster-info
kubectl get nodes -o wide

# access minikube VM using SSH
minikube ssh

# however 'none' driver does not support 'minikube ssh' command
# so switch to 'docker' driver
minikube config set driver docker        # set docker as default
minikube delete                          # delete "minikube" cluster in none
minikube start

minikube ssh
minikube stop
minikube addones list                    # to check dashboard enabled

minikube addons enable dashboard
```

* Deploy applications

```shell
kubectl create deployment hello-minikube --image=k8s.gc.io/echoserver:1.4
kubectl get services hello-minikube
minikube service hello-minkube
kubectl port-forward service/hello-minikube 7080:8080

# http://localhost:7080
```

# Hello-World deployment

``` shell
kubectl create -f deployment.xml               # deploy hello-world app and expose as service
minikube ip                                    # get ip
```

# Deploy my own app

``` sh
# create a folder for this app
mkdir my-hello 
cd my-hello

npm init
npm install express --save
touch index.js
vi index.js              # expose 8080 having one http-get method to say something

# verify with docker
docker build -t georgezhou888/my-hello:v1.0.0 .
docker run -p 8080:8080 georgezhou888/my-hello:v1.0.0 
curl http://localhost:8080                # see the output of hello-something

vi package.json
# change scripts content to {"start": "node index.js"}

vi .dockerignore
# contents 
node_modules
Dockerfile
deployment.yml
# end .dockerignore

vi Dockerfile
# contents
FROM node:carbon
WORKDIR /usr/src/app
COPY package.json .
COPY package-lock.json .
RUN npm install
COPY . .
EXPOSE 8080
CMD [ "npm", "start" ]
# end Dockerfile

# build docker image
docker build -t georgezhou888/my-hello:v1.0.1 .
docker login
docker push georgezhou888/my-hello:v1.0.1

vi deployment.xml
# see https://github.com/zpc888/k8s-my-hello/blob/master/my-hello/deployment.yml

kubuctl create -f deployment.xml

minikube ip                                   # get cluster ip

kubuctl get nodes -o wide                     # when at least one pod is running
curl http://${minikube-ip-output}:30002       # see the result
```

# K8s Basic

## Definition of Kubernetes

* Open source container orchestration tool
* Helps you manage containerized applications in different deployment environments

## K8s Components

* Pod: Node and Pod
  * Smallest unit of K8s
  * Abstraction over container
  * Usually 1 application per Pod
  * Each Pod gets its own IP address
  * New IP is assigned for new created Pod
* Service: Service and Ingress
  * permanent IP address
  * Life cycle of Pod and Service Not connected
  * External Service vs Internal Service
* Ingress
  * Wrap Service for external access so that it goes Ingress first and the forward to Service
  * App access URL usually in the built application
* Volumes
  * store database-like data
  * can be local or remote
* Secrets
  * Like ConfigMap, but used to store secret data
  * base64 encoded
* ConfigMap
  * external configuration of your application
  * APP_URL = mongo-db
  * Don't put password in the ConfigMap, but instead of in Secret
* StatefulSet
  * for STATEFUL apps, e.g. Database server 
  * Deployment for stateLESS Apps
* Deployment
  * blueprint for an app pods
  * abstraction of Pods
  * DB can't be replicated via Deployment! -- See StatefulSet

## K8s Architecture

* Worker machine in K8s cluster
  * each Node has multiple Pods on it
  * 3 processes must be installed on every Node
    * container runtime, such as docker
    * Kubelet -- interacts with the container runtime and node / starts the pod with a container inside
    * Kube Proxy -- forwards the requests
  * Worker Nodes do the actual work

* how to interact with the cluster
  * how to schedule pod?
  * monitor?
  * re-schedule/re-start pod?
  * join a new Node?

all managing processes are done by master processes / node

* 4 processes run on every master node
  * Api Server 
    * behaves like a Cluster gateway
    * acts as a gatekeeper for authentication
    * Some request ---> API Server ---> validates request ---> ... other processes ---> Pod
  * Scheduler
    * Schedule new Pod --> API Server --> Scheduler ---> Where to put the Pod?
    * Scheduler just decides on which Node new Pod should be scheduled
  * Controller Manager
    * detects cluster state changes
    * Controller Manager ---> Scheduler ---> Kubelet
  * etcd
    * is the cluster brain
    * Cluster changes get stored in the Key Value Store

## minikube

* creates virtual box on your box
* Node runs in that virtual box
* 1 Node K8s Cluster which runs 4 master processes and 3 worker processes
* Docker container runtime pre-installed
* for testing purposes

## kubectl

* command line tool for k8s cluster

### CRUD commands

* Create deployment - kubectl create deployment [name] --image=image-name
* Edit deployment - kubectl edit deployment [name]
* Delete deployment - kubectl delete deployment [name]

### Status of different K8s Components

* kubectl get nodes | pod | services | replicaset | deployments

### Debugging pods

* Log to console - kubectl logs [pod name]
* Get Interactive Terminal - kubectl exec -it [pod name] -- bin/bash
* Get info about pod - kubectl describe pod [pod name]

### Use configuration file for CRUD

* Apply a configuration file - kubectl apply -f [file name]
* Delete with configuration file - kubectl delete -f [file name]

## YAML Configuration file structure

### Each configuration (Deployment / Service) has 3 parts

1. metadata
2. specification
   1. Attributes of "spec" are specific to KIND
3. status - auto generated and updated by kubenetes <-- from master-process etcd

# K8s Advanced Concepts

## K8s Namespaces - Organize your Components

### What is a Namespace

* Organise resources in namespaces
* Virtual Cluster inside a Cluster
* 4 default namespaces
  * default
    * resources you create are located here
  * kube-node-lease
    * heartbeats of nodes
    * each node has associated lease object in namespace
    * determines the availability of a node
  * kube-public
    * publicly accessible data
    * a configmap, which contains cluster information
  * kube-system
    * DO NOT create or modify by you
    * System processes
    * Master and kubectl processes
  * kubernets-dashboard only with minikube, not in normal one
  * kubectl create namespace [namespace name]
  * create a namespace with a configuration file

### Why to Use Namespace

* Resources grouped in Namespaces
* Conflicts: Many teams, same application
* Resource Sharing: Staging and Development
* Resource Sharing: Blue/Green Deployment - The versions of application differ
* Access and Resource Limits on Namespaces 
  * each team has its own isolated environment
  * Limit: CPU, RAM, Storage per NS
* Node and Volume live globally in a cluster, no namespace

### How to use

* create components in a Namespace
* configuration file over kubectl cmd - better documented
* change the active namepspace with kubens!
  * brew install kubectx    # for macos
  * kubens my-namespace  # switch active namespace to "my-namespace"

## K8s Ingress

* in front of external service, changing it to internal service
* Ingress and internal service configuration
* How to configure Ingress in your Cluster
* What is Ingress Controller
  * Evaludates all the rules
  * Manages redirections
  * entrypoint to cluster
  * many 3rd-party implementations
  * K8s Nginx Ingress Controller
* demo ingress in minikube
  * install Ingress Controller in Minikube - minikube addons enable ingress
    * automatically starts the K8s Nginx implementation of Ingress Controller
  * kubectl get pods -n kube-system -o wide   => check ingress-nginx-controller-{id} is running
* 

## Helm - Package Manager

## Volumes - Persisting Data in K8s

## K8s StatefulSet - Deploying Stateful Apps

## K8s Services

