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
```



* 