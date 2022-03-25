# Install Kubernetes on Ubuntu 20.04

## Prerequisites

- Clean machine with Ubuntu Server 20.04 installed.
- Power: as much as you have

## Content

- Install a control-plane and worker
- Add Nginx Ingress Controler

## Update your system

```
sudo apt-get update -y
```

## Install Docker

```
sudo apt-get install -y wget gnupg-agent software-properties-common
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

```
sudo apt-get update -y
```

```
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

## Start and Enable Docker

Configure the Docker daemon, in particular, to use systemd for the management of the container’s cgroups

```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
 "max-size": "100m"
 },
 "storage-driver": "overlay2"
}
EOF
```

Restart Docker and enable on boot

```
sudo systemctl enable --now docker
```

```
sudo systemctl daemon-reload
```

```
sudo systemctl restart docker
```

```
sudo systemctl status docker
```

Set rootless for docker

```
sudo usermod -aG docker ${USER}
```

```
#login again for apply change
su - $USER
```

```
id
```

```
docker --version
```

```
docker ps
```

## Disable SWAP

Disable swap

```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```
sudo swapoff -a
```

Letting iptables see bridged traffic

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
sudo sysctl --system
```

## Install Kubectl and Kubernetes

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update -y
```

```
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository:

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
```

```
sudo apt-get install -y kubelet kubeadm kubectl
```

```
sudo apt-mark hold kubelet kubeadm kubectl
```

## Initialize the Cluster and Master Node

Now that the components are installed on our nodes, we can initialize the cluster and master node. To do that, run the following command on your node that is to be the master node of the Kubernetes cluster.

```
sudo kubeadm init
```

Configure kubectl using commands in the output:

```
mkdir -p $HOME/.kube
```

```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check cluster status:

```
kubectl cluster-info
```

## Install Networking overlay

Kubernetes requires a networking overlay component. Here I am choosing to use the Weave networking overlay for my Kubernetes cluster. To implement the Weave network overlay, use the following command:

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

```
kubectl get node -o wide
```

```
kubectl get pod --all-namespaces
```

## Join the Worker Nodes to Kubernetes Cluster

At this point, you should have a Kubernetes cluster with a single master node joined. We need to join the other worker nodes to the cluster. To do that, instead of use kubeadm inituse kubeadm jointo join master node. The join command that was given is used to add a worker node to the cluster.

```
sudo kubeadm join 10.10.27.234:6443 --token axwp1d.XXXXXXXXXX \
 --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX1e74
```

Operate the master node and verify that worker node joined the cluster

```
kubectl get nodes
```

## Connect to Dashboard

The Dashboard UI is not deployed by default. To deploy it, run the following command:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```

## Accessing the Dashboard UI

To protect your cluster data, Dashboard deploys with a minimal RBAC configuration by default. Currently, Dashboard only supports logging in with a Bearer Token. To create a token for this demo, you can follow our guide on creating a sample user.

### Getting Token

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | awk '/^deployment-controller-token-/{print $1}') | awk '$1=="token:"{print $2}'
```

## Command line proxy

You can enable access to the Dashboard using the kubectl command-line tool, by running the following command:

```
kubectl proxy
```

Acces the dashboard with this address :

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

## Create Role binding

```
kubectl create clusterrolebinding deployment-controller --clusterrole=cluster-admin --serviceaccount=kube-system:deployment-controller
```

## Add Nginx Ingress Controller

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/baremetal/deploy.yaml
```

This is to be deployed on bare metal server or on VMs where k8s was installed manually. If you wish to deploy in a different environment pleaser refer to https://kubernetes.github.io/ingress-nginx/deploy/
