## kube-sandbox

Kubernetes API https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/#-strong-api-overview-strong-

### Install minikube on Ubuntu 16.04

https://medium.com/@nieldw/switching-from-minikube-with-virtualbox-to-kvm-2f742db704c9
https://websiteforstudents.com/virtualbox-5-2-on-ubuntu-16-04-lts-server-headless/
https://medium.com/margarytachepiga/how-to-install-minikube-on-ubuntu-88a3034801b1
https://medium.com/@nieldw/running-minikube-with-vm-driver-none-47de91eab84c?email=antti18%2Bmedium%40kaihola.fi&g-recaptcha-response
https://github.com/kubernetes/minikube


0. Requirements:
   * docker version 18.06. If docker version 18.09 installed, remove it first and then run ```sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu``` to re-install the docker version that is supported by kubernetes
   * To execute the docker command without sudo, run ```sudo usermod -aG docker ${USER}``` and then ```su - ${USER}``` 
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
0. Requirements:
   * docker version 18.06. If docker version 18.09 installed, do not purge it, simply run ```sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu``` to re-install the docker version that is supported by kubernetes
   * To execute the docker command without sudo, run ```sudo usermod -aG docker ${USER}``` and then ```su - ${USER}``` 
1. Run ```sudo swapoff -a``` ti [disable swap](https://docs.platform9.com/support/disabling-swap-kubernetes-node/)
   * [Check if swap is disabled](https://www.cyberciti.biz/faq/linux-check-swap-usage-command/): Run the command ```top``` and look at the information about swap for at least 60 seconds and make sure that swap is disable.
   * If swap is getting re-enabled shortly, run the command ```sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab``` to comment out the lines containing the word ```swap``` in the file ```/etc/fstab```.
   * Run ```sudo swapoff -a``` and check if swap is disabled.
2. Install Kubernetes client
    * ```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
    * ```echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list```
    * ```sudo apt-get update -y```
    * ```sudo apt-get install kubelet kubeadm kubectl -y```
> Make sure that the worker node's kubernetes client version is consistent with that of the master node. If the master node uses an old version, you may need to downgrade your installations. For example, ```sudo apt-get install kubelet=1.12.2-00 kubeadm=1.12.2-00 kubectl=1.12.2-00 -y --allow-downgrades```

3. go to https://github.com/NVIDIA/k8s-device-plugin#quick-start and make sure you have followed the section titled: Preparing your GPU nodes
    * [Install nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
    ```
    $ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    $ sudo apt-get update
    $ sudo apt-get install -y --allow-unauthenticated nvidia-docker2
    $ sudo pkill -SIGHUP dockerd
    #Test nvidia-smi with the latest official CUDA image
    $ docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
    ```
    * Check the file /etc/docker/daemon.json. It may look like this:
    ```
    {
        "runtimes": {
            "nvidia": {
                "path": "nvidia-container-runtime",
                "runtimeArgs": []
            }
         }
    }
    ```
    * Enable the nvidia runtime as your default runtime on your node by editing the file /etc/docker/daemon.json:
    ```
    {
        "default-runtime": "nvidia",
        "runtimes": {
            "nvidia": {
                "path": "nvidia-container-runtime",
                "runtimeArgs": []
            }
         }
    }
    ```
    * To make pip work properly when you build Docker images, set your own DNS server address:
    ```
    {
        "dns": ["135.5.25.53", "8.8.8.8"],
        "default-runtime": "nvidia",
        "runtimes": {
            "nvidia": {
                "path": "nvidia-container-runtime",
                "runtimeArgs": []
            }
         }
    }
    ```
    * Then, you need to restart the docker service:
    ```
    $ sudo service docker restart
    ```
4. Run ```sudo kubeadm join 135.222.154.219:6443 --token odsb5k.ho566dm6817oa696 --discovery-token-ca-cert-hash sha256:903d8bad14f6bb297a596fb39188164508a41bbe68d9f0410e3a429ce0059e0b```
5. Check kubelet status:
```
$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2019-02-05 16:32:28 EST; 14min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 8168 (kubelet)
    Tasks: 30
   Memory: 48.0M
      CPU: 37.261s
   CGroup: /system.slice/kubelet.service
           └─8168 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs
 ```
 6. Troubleshooting: If you run into an error, run ```journalctl --reverse -xeu kubelet```
 
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
I also have the same issue.
i'v solve the problem by deleting the plugins 'loop' within the cm of coredns. but i don't know if this cloud case other porblems.
1、kubectl edit cm coredns -n kube-system
2、delete ‘loop’ ,save and exit
3、restart coredns pods by："kubctel delete pod coredns.... -n kube-system"
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
### Mount NFS Volume
```
mmlu@cheetah:/nfs$ ls
experiments  general
mmlu@cheetah:/nfs$ cd experiments/
mmlu@cheetah:/nfs/experiments$ ls
segmentation
mmlu@cheetah:/nfs/experiments$ cd ../
mmlu@cheetah:/nfs$ sudo chmod 777 experiments/
mmlu@cheetah:/nfs$ sudo chmod 777 experiments/segmentation/
mmlu@cheetah:/nfs$ ls
experiments  general
mmlu@cheetah:/nfs$ sudo mount 135.222.154.219:/var/nfs/experiments/segmentation /nfs/experiments/segmentation
mmlu@cheetah:/nfs$ cd
mmlu@cheetah:~$ df -h
Filesystem                                         Size  Used Avail Use% Mounted on
udev                                                16G     0   16G   0% /dev
tmpfs                                              3.2G   51M  3.1G   2% /run
/dev/sda2                                          1.8T   88G  1.7T   6% /
tmpfs                                               16G   72M   16G   1% /dev/shm
tmpfs                                              5.0M  4.0K  5.0M   1% /run/lock
tmpfs                                               16G     0   16G   0% /sys/fs/cgroup
/dev/loop0                                          90M   90M     0 100% /snap/core/6034
/dev/loop2                                          89M   89M     0 100% /snap/core/5897
/dev/sda1                                          511M  7.0M  505M   2% /boot/efi
tmpfs                                              3.2G  112K  3.2G   1% /run/user/1000
/dev/loop3                                          90M   90M     0 100% /snap/core/6130
135.222.154.219:/var/nfs/experiments/segmentation  1.8T   46G  1.7T   3% /nfs/experiments/segmentation
mmlu@cheetah:~$ cd /nfs/experiments/segmentation/
mmlu@cheetah:/nfs/experiments/segmentation$ ls

 you need to type this into any terminal to mount the directory: 
sudo mount 135.222.154.219:/var/nfs/experiments/segmentation    /nfs/experiments/segmentation
 
Then do this:
sudo vi /etc/fstab
 
add this line so that when you reboot next time, it is automatically mounted:
135.222.154.219:/var/nfs/experiments/segmentation    /nfs/experiments/segmentation   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```
