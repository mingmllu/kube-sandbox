## kube-sandbox

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
