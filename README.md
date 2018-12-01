## kube-sandbox

### Install minikube on Ubuntu 16.04

0. Requirements:
   * docker version 18.06. If docker version 18.09 installed, remove it first and then run ```sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu``` to re-install the docker version that is accepted by kubernetes
1. Install kubectl 
   * ```sudo apt-get update && sudo apt-get install -y apt-transport-https```
   * ```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
   * ```echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list```
   * ```sudo apt-get update```
   * ```sudo apt-get install -y kubectl```
2. Install minikube 
   * ```curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube```
3. Launch minikube 
   * ```sudo -E minikube --vm-driver=none```
   * ```sudo chown -R $USER $HOME/.kube```
   * ```sudo chgrp -R $USER $HOME/.kube```
   * ```sudo chown -R $USER $HOME/.minikube```
   * ```sudo chgrp -R $USER $HOME/.minikube```
> When using the none driver, the kubectl config and credentials generated will be root owned. You will need to set the correct permissions as above. This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true.
```
export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
export KUBECONFIG=$HOME/.kube/config
```
   

### Add workstation (Ubuntu 16.04) to the Kubernetes cluster:

1. sudo swapoff -a
2. curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
3. echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list
4. sudo apt-get update -y
5. sudo apt-get install kubelet kubeadm kubectl -y
6. go to https://github.com/NVIDIA/k8s-device-plugin#quick-start and make sure you have followed the section titled: Preparing your GPU nodes
7. sudo kubeadm join 135.222.154.219:6443 --token fmvwrm.1ln5wodfmzf774sd --discovery-token-ca-cert-hash sha256:ad8134c6505d3f5e83be5d910dc8cd90c58625773db402e824be059362585d06

### How to run a simple TFjob in the cluster (tested in AWS only)

1. KF_ENV=default
2. ks init video-analytics-app
3. cd video-analytics-app
4. CNN_JOB_NAME=mycnnjob
5. VERSION=v0.3-branch
6. ks registry add kubeflow-git github.com/kubeflow/kubeflow/tree/${VERSION}/kubeflow
7. ks pkg install kubeflow-git/examples
8. ks generate tf-job-simple ${CNN_JOB_NAME} --name=${CNN_JOB_NAME}
9. ks apply ${KF_ENV} -c ${CNN_JOB_NAME}
10. export NAMESPACE=kubeflow\nkubectl port-forward -n ${NAMESPACE}  `kubectl get pods -n ${NAMESPACE} --selector=service=ambassador -o jsonpath='{.items[0].metadata.name}'` 8080:80
11. go to localhost:8080 to see the kubeflow dashboard
