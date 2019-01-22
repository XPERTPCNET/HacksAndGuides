# DEPLOY-KUBERNETES-CLUSTER-UBUNTU-18.04
Kubernetes is a production-grade, open-source platform that orchestrates the placement (scheduling) and execution of application containers within and across computer clusters.

## Architecture

A Kubernetes cluster consists of two types of resources:

- The Master coordinates the cluster
- Nodes are the workers that run applications

The nodes communicate with the master using the Kubernetes API, which the master exposes. End users can also use the Kubernetes API directly to interact with the cluster.

A Kubernetes cluster can be deployed on either physical or virtual machines.

# Kubernetes Deployment with Kubeadm

## Prepare the VMs
#### VM Requirements
- 2 vCPU 
- 2GB RAM 
  
### Update Ubuntu 18.04
 ```
 sudo apt-get update && sudo apt-get dist-upgrade -y
 sudo apt-get install -y linux-image-extra-virtual
 ```
### Setup unique hostnames for all nodes
Give unique names to your master:
```
sudo hostnamectl set-hostname kubernetes-master
```
And to your worker nodes:
```
sudo hostnamectl set-hostname kubernetes-worker1
sudo hostnamectl set-hostname kubernetes-worker2
```
### Disable swap memory (if running) on nodes
> Kubernetes does not perform properly on a system that is using swap memory. Run the following command on both the nodes in order to disable swap memory

```
sudo swapoff -a 
````
> This is not persistent across host restarts so add to /etc/fstab accordingly for persistence.
```
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Comment out for swap line looks like this on my VM:
----------------------------------------------------
#/swap.img      none    swap    sw      0       0
```
> Servers require a host reboot in order for the hostname change to take effect.

---
 ## Install Kubernetes

### Install Docker and enable docker service
```
sudo apt-get install -y docker.io
sudo systemctl enable docker
```
### Install Kubernetes components

> NOTE: At the time of writing only Ubuntu 16.04 Xenial Kubernetes repository is available. Replace the below xenial with bionic codename once the Ubuntu 18.04 Kubernetes repository becomes available.

#### Add kubernetes repository
```
sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
```
#### Install Kubeadm
This one command should install kubeadm, kubectl, kubelet.
```
sudo apt-get install -y kubeadm
```
---
## Initialize Kubernete Master Node

Now we are ready to initialize the Kubernetes master node. To do so execute the following linux command on your master node, Take a note of the **kubeadm join** command from the bottom of the output:
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
- **pod-network-cidr** = specify the range of IP addresses for the pod network. We're using the *'flannel virtual network'*. If you want to use another pod network such as kube-router, weave-net or calico, change the range IP address.

 On the **master node**, run the following as a **regular** user:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
---
## Add Worker Nodes to Kubernetes Cluster (MUST BE ROOT)

Use the kubeadm join command retrieved earlier from the Kubernetes master node initialization output to join your Kubernetes cluster:
```
## USE THE JOIN COMMAND FROM YOUR OUTPUT THIS IS A SAMPLE
$ kubeadm join 192.168.100.6:6443 --token 06tl4c.oqn35jzecidg0r0m --discovery-token-ca-cert-hash sha256:c40f5fa0aba6ba311efcdb0e8cb637ae0eb8ce27b7a03d47be6d966142f2204c
```
ON you kubernetes Master, run this command to confirm that the nodes have been added:
```
kubectl get nodes
```
---
## Deploy a  Pod Network
The pod network is utilized for pod communication between nodes, we are going to use flannel:

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
 Use the kubectl command to confirm that everything is up and ready:
```
kubectl get pods --all-namespaces
```
---
# Done!
