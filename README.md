## kube-sandbox

### Add workstation (Ubuntu 16.04) to the Kubernetes cluster:

1. sudo swapoff -a
2. curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
3. echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list
4. sudo apt-get update -y
5. sudo apt-get install kubelet kubeadm kubectl -y
6. go to https://github.com/NVIDIA/k8s-device-plugin#quick-start and make sure you have followed the section titled: Preparing your GPU nodes
7. sudo kubeadm join 135.222.154.219:6443 --token sk3ahu.h237m8u3x9wfwsev --discovery-token-ca-cert-hash sha256:ad8134c6505d3f5e83be5d910dc8cd90c58625773db402e824be059362585d06
