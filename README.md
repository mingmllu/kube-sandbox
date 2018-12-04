## kube-sandbox

### Install minikube on Ubuntu 16.04

https://medium.com/@nieldw/switching-from-minikube-with-virtualbox-to-kvm-2f742db704c9
https://websiteforstudents.com/virtualbox-5-2-on-ubuntu-16-04-lts-server-headless/
https://medium.com/margarytachepiga/how-to-install-minikube-on-ubuntu-88a3034801b1
https://medium.com/@nieldw/running-minikube-with-vm-driver-none-47de91eab84c?email=antti18%2Bmedium%40kaihola.fi&g-recaptcha-response
https://github.com/kubernetes/minikube


0. Requirements:
   * docker version 18.06. If docker version 18.09 installed, remove it first and then run ```sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu``` to re-install the docker version that is supported by kubernetes
1. Install kubectl 
   * ```sudo apt-get update && sudo apt-get install -y apt-transport-https```
   * ```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
   * ```echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list```
   * ```sudo apt-get update```
   * ```sudo apt-get install -y kubectl```
2. Install minikube 
   * ```curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube```
3. Launch minikube 
   * ```sudo -E minikube start --vm-driver=none```
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

### Install ksonnet

See https://github.com/ksonnet/ksonnet

1. Download the binary code ks_0.13.1_linux_amd64.tar.gz at ```https://github.com/ksonnet/ksonnet/releases```
2. Run ```tar -xvf ks_0.13.1_linux_amd64.tar.gz```
3. Run ```~/Downloads/ks_0.13.1_linux_amd64$ sudo cp ks /usr/bin/ks```

### Set up Kubleflow

See https://www.kubeflow.org/docs/started/getting-started/#quick-start
1. Run the following commands to download kfctl.sh
```
export KUBEFLOW_SRC=$HOME/kubeflow
mkdir ${KUBEFLOW_SRC}
cd ${KUBEFLOW_SRC}
export KUBEFLOW_TAG=v0.3.1

curl https://raw.githubusercontent.com/kubeflow/kubeflow/${KUBEFLOW_TAG}/scripts/download.sh | bash
```
> You will see the ```kubeflow``` and ```scripts``` directories created under the directory ```${KUBEFLOW_SRC}```
2. Run the following command to create kubeflow environment variables
```
export KFAPP=kf-basic
${KUBEFLOW_SRC}/scripts/kfctl.sh init ${KFAPP} --platform none
```
> ```${KFAPP}``` is the name of a directory where you want kubeflow configurations to be stored. This directory will be created under the current working directory when you run init. The new directory`kf-basic` will be created and it will contain an ```env.sh``` file that stores environment variables as below
```
PLATFORM=none
KUBEFLOW_REPO=/home/appml/kubeflow
KUBEFLOW_VERSION=master
KUBEFLOW_KS_DIR=/home/appml/kf-basic/ks_app
KUBEFLOW_DOCKER_REGISTRY=
K8S_NAMESPACE=kubeflow
KUBEFLOW_CLOUD=null
```
3. Run the following command to install necessary packages and generate ksonnet application components
```
cd ${KFAPP}
${KUBEFLOW_SRC}/scripts/kfctl.sh generate k8s
```
>The ksonnet app will be created in the directory ```${KFAPP}/ks_app```
4. Run the following command to deploy the components into pods in our kubernetes cluster
```
${KUBEFLOW_SRC}/scripts/kfctl.sh apply k8s
```
> We can check the deployment status by:
```
appml@woodpecker:~/kf-basic$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                                      READY   STATUS             RESTARTS   AGE
kube-system   coredns-c4cffd6dc-7dq22                                   0/1     CrashLoopBackOff   398        1d
kube-system   etcd-minikube                                             1/1     Running            0          1d
kube-system   kube-addon-manager-minikube                               1/1     Running            0          1d
kube-system   kube-apiserver-minikube                                   1/1     Running            0          1d
kube-system   kube-controller-manager-minikube                          1/1     Running            0          1d
kube-system   kube-dns-86f4d74b45-nl5ng                                 3/3     Running            6          1d
kube-system   kube-proxy-pwgdn                                          1/1     Running            0          1d
kube-system   kube-scheduler-minikube                                   1/1     Running            0          1d
kube-system   kubernetes-dashboard-6f4cfc5d87-rjm9c                     1/1     Running            0          1d
kube-system   storage-provisioner                                       1/1     Running            0          1d
kubeflow      ambassador-7fb86f6bc5-5rshb                               3/3     Running            0          20m
kubeflow      ambassador-7fb86f6bc5-dwg2q                               3/3     Running            0          20m
kubeflow      ambassador-7fb86f6bc5-qwq5b                               3/3     Running            0          20m
kubeflow      argo-ui-7b6585d85d-k7xhv                                  1/1     Running            0          20m
kubeflow      centraldashboard-f8d7d97fb-85p4v                          1/1     Running            0          20m
kubeflow      modeldb-backend-69dfc464df-hmqt6                          0/1     CrashLoopBackOff   7          19m
kubeflow      modeldb-db-6cf5bb764-v6frj                                1/1     Running            0          19m
kubeflow      modeldb-frontend-74b66f8dc8-jhpwl                         1/1     Running            0          19m
kubeflow      spartakus-volunteer-65ccbf687-n5cpm                       1/1     Running            0          19m
kubeflow      studyjob-controller-68f5948984-dssvw                      1/1     Running            0          19m
kubeflow      tf-hub-0                                                  1/1     Running            0          20m
kubeflow      tf-job-dashboard-7cddcdf9c4-xhrxd                         1/1     Running            0          20m
kubeflow      tf-job-operator-v1alpha2-6566f45db-kh77j                  1/1     Running            0          20m
kubeflow      vizier-core-d74cbfd98-wlh2q                               1/1     Running            3          19m
kubeflow      vizier-db-cc59bc8bd-khsd6                                 1/1     Running            0          19m
kubeflow      vizier-suggestion-bayesianoptimization-788df66688-mh8dx   1/1     Running            0          19m
kubeflow      vizier-suggestion-grid-76c648b78-n8l4b                    1/1     Running            0          19m
kubeflow      vizier-suggestion-hyperband-5df8cf7bc8-x4tff              1/1     Running            0          19m
kubeflow      vizier-suggestion-random-65b9fd7c48-s59mk                 1/1     Running            0          19m
kubeflow      workflow-controller-59c7967f59-tvfhc                      1/1     Running            0          20m
appml@woodpecker:~/kf-basic$ 
```
```
appml@woodpecker:~/kf-basic$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-c4cffd6dc-qww29                                   1/1     Running   2          24m
kube-system   etcd-minikube                                             1/1     Running   0          28m
kube-system   kube-addon-manager-minikube                               1/1     Running   0          27m
kube-system   kube-apiserver-minikube                                   1/1     Running   0          28m
kube-system   kube-controller-manager-minikube                          1/1     Running   0          27m
kube-system   kube-dns-86f4d74b45-rgtvw                                 3/3     Running   0          28m
kube-system   kube-proxy-c8kct                                          1/1     Running   0          28m
kube-system   kube-scheduler-minikube                                   1/1     Running   0          27m
kube-system   kubernetes-dashboard-6f4cfc5d87-j8kvx                     1/1     Running   0          28m
kube-system   storage-provisioner                                       1/1     Running   0          28m
kubeflow      ambassador-7fb86f6bc5-j2lfd                               3/3     Running   0          8m
kubeflow      ambassador-7fb86f6bc5-q57qm                               3/3     Running   1          8m
kubeflow      ambassador-7fb86f6bc5-z62k2                               3/3     Running   2          8m
kubeflow      argo-ui-7b6585d85d-djk92                                  1/1     Running   0          7m
kubeflow      centraldashboard-f8d7d97fb-kfz4m                          1/1     Running   0          8m
kubeflow      modeldb-backend-69dfc464df-9nw7k                          1/1     Running   4          7m
kubeflow      modeldb-db-6cf5bb764-pxg6c                                1/1     Running   0          7m
kubeflow      modeldb-frontend-74b66f8dc8-62bqq                         1/1     Running   0          7m
kubeflow      spartakus-volunteer-5b9f857d45-s6p6n                      1/1     Running   0          6m
kubeflow      studyjob-controller-68f5948984-n8fll                      1/1     Running   0          6m
kubeflow      tf-hub-0                                                  1/1     Running   0          8m
kubeflow      tf-job-dashboard-7cddcdf9c4-mkjpl                         1/1     Running   0          8m
kubeflow      tf-job-operator-v1alpha2-6566f45db-stq7m                  1/1     Running   0          8m
kubeflow      vizier-core-d74cbfd98-56qj4                               1/1     Running   2          6m
kubeflow      vizier-db-cc59bc8bd-xwxqb                                 1/1     Running   0          6m
kubeflow      vizier-suggestion-bayesianoptimization-788df66688-7d7w7   1/1     Running   0          7m
kubeflow      vizier-suggestion-grid-76c648b78-zsml2                    1/1     Running   0          7m
kubeflow      vizier-suggestion-hyperband-5df8cf7bc8-jwvlr              1/1     Running   0          6m
kubeflow      vizier-suggestion-random-65b9fd7c48-98flb                 1/1     Running   0          6m
kubeflow      workflow-controller-59c7967f59-sg7hx                      1/1     Running   0          7m
```
