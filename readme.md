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

