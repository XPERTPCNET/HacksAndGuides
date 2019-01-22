# Kubernetes-the-Hard-Way-Steps

## Ubuntu 18.04 (bionic beaver)

#### Nodes:
- Kube01 (controller)
- Kube02 (controller)
- Kube03, (worker)
- Kube04 (worker)
- Kube05.(load Balancer)

# Steps:
After I setup VM's and DNS for the nodes, i proceeded with the following:

## INSTALL TOOLS FOR KUBERNETES
> We're going to use the pacakage manager for debian.
#### Install  cfssl
```
sudo apt-get install golang-cfssl -y
cfssl version
```
#### Install kubectl
```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl
```
---
##  CA AND KUBERNETES CERTIFICATES

### Provision the Certificate Authority
```
cd ~/
mkdir kthw
cd kthw/
```
Use this command to generate the certificate authority. Include the opening and closing curly braces to run this entire block as a single command.
```
{

cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
Generated files:
> ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem

### Generate Client Certificates

- Admin Client certificate:
```
cd ~/kthw
```
```
{

cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```
Generated files:
> admin.csr  admin-csr.json  admin-key.pem  admin.pem 


### Kubelet Client certificates
 Be sure to enter your actual cloud server values for all four of the variables at the top:
```
WORKER0_HOST=<Public hostname of your first worker node cloud server>
WORKER0_IP=<Private IP of your first worker node cloud server>
WORKER1_HOST=<Public hostname of your second worker node cloud server>
WORKER1_IP=<Private IP of your second worker node cloud server>
```
My values for the variables:
>WORKER0_HOST=kube03.home.lab
WORKER0_IP=192.168.123.213
WORKER1_HOST=kube04.home.lab
WORKER1_IP=192.168.123.214

```
cd ~/kthw

{
cat > ${WORKER0_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER0_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

cat > ${WORKER1_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

}
```
Generated files:
> kube03.home.lab.csr kube03.home.lab-csr.json kube03.home.lab-key.pem kube03.home.lab.pem
> kube04.home.lab.csr kube04.home.lab-csr.json kube04.home.lab-key.pem kube04.home.lab.pem
  
- Controller Manager Client certificate:
```
cd ~/kthw

{

cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager kube-controller-manager.csr 

}
```
Generated files:
> kube-controller-manager-key.pem kube-controller-manager.pem kube-controller-manager.csr kube-controller-manager.csr.json

- Kube Proxy Client Certificate
```
{

cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Me;lbourne",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```
Generated files:
> kube-proxy-csr.json kube-proxy.csr kube-proxy-key.pem kube-proxy.pem

- Kube Scheduler Client Certificate:
```
cd ~/kthw

{

cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```
Resutling files:
> kube-scheduler-key.pem kube-scheduler.pem kube-scheduler-csr.json kube-scheduler-csr

### Generate Kubernetes API Certficate
```
cd ~/kthw
CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default
```
>CERT_HOSTNAME=10.32.0.1,192.168.123.211,kube01.home.lab,192.168.123.212,kube02.home.lab,192.168.123.215,kube05.home.lab,127.0.0.1,localhost,kubernetes.default
```
cd ~/kthw

{

cat > kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```
Generated files:
> kubernetes-key.pem kubernetes.pem.

### Generate the Service Account Key Pair
```

cd ~/kthw

{

cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Melbourne",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Florida"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```
Generated files:
>service-account-key.pem service-account.pem

### Distribuite the Certificate Files
Move certificate files to the worker nodes:
```
scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem user@<worker 1 public IP>:~/
scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem user@<worker 2 public IP>:~/
``` 
Move certificate files to the controller nodes:
```
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 1 public IP>:~/
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 2 public IP>:~/
```
---
## KUBERNETES CONFIGURATION FILES FOR AUTHENTICATION

### Generating Kubeconfigs for the Cluster
- Generate a kubelet kubeconfig for each worker node:
```
for instance in <worker 1 hostname> <worker 2 hostname>; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
- Generate a kube-proxy kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```
- Generate a kube-controller-manager kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```
- Generate a kube-scheduler kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
- Generate an admin kubeconfig:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
### Distributing Kubeconfig files
- Move kubeconfig files to the worker nodes:
```
scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/
```
- Move kubeconfig files to the controller nodes:
```
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/
```

### Generating the Data Encryption Config
- Generate the Kubernetes Data encrpytion config file containing the encrpytion key:
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

- Copy the file to both controller servers:
```
scp encryption-config.yaml user@<controller 1 public ip>:~/
scp encryption-config.yaml user@<controller 2 public ip>:~/
```
---
## THE KUBERNETES CONTROLLERS
### Install Etcd Cluster

> Run these commands on BOTH controller nodes to create the cluster:
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```
- Set up the following environment variables. Be sure you replace all of the <placeholder values> with their corresponding real values:
```
ETCD_NAME=<cloud server hostname>
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
INITIAL_CLUSTER=<controller 1 hostname>=https://<controller 1 private ip>:2380,<controller 2 hostname>=https://<controller 2 private ip>:2380
```
- Create the systemd unit file for etcd using this command. Note that this command uses the environment variables that were set earlier:

```
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

- Start and enable the etcd service:

```
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
You can verify that the etcd service started up successfully like so:

sudo systemctl status etcd
Use this command to verify that etcd is working correctly. The output should list your two etcd nodes:

sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
  
  ```
 
  ### Installing Kubernetes Control Plane Binaries
  ```
  sudo mkdir -p /etc/kubernetes/config

wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl"

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
  ```
  ### Setting up the Kubernetes API Server
  
You can configure the Kubernetes API server like so:
```
sudo mkdir -p /var/lib/kubernetes/

sudo cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem \
  encryption-config.yaml /var/lib/kubernetes/
```
Set some environment variables that will be used to create the systemd unit file. Make sure you replace the placeholders with their actual values:
```
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
CONTROLLER0_IP=<private ip of controller 0>
CONTROLLER1_IP=<private ip of controller 1>
```
Generate the kube-apiserver unit file for systemd:
```
cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2 \\
  --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
### Setting up the Kubernetes Controller Manager 
You can configure the Kubernetes Controller Manager like so:
```
sudo cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Generate the kube-controller-manager systemd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
### Setting up the Kubernetes Scheduler

Copy kube-scheduler.kubeconfig into the proper location:
```
sudo cp kube-scheduler.kubeconfig /var/lib/kubernetes/
Generate the kube-scheduler yaml config file.

cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

Create the kube-scheduler systemd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
Start and enable all of the services:
```
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```
t's a good idea to verify that everything is working correctly so far: Make sure all the services are active (running):
```
sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
```
Use kubectl to check componentstatuses:

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
You should get output that looks like this:
```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```
### Enable HTTP Health Checks (optional)

You can set up a basic nginx proxy for the healthz endpoint by first installing nginx"
```
sudo apt-get install -y nginx
```

Create an nginx configuration for the health check proxy:
```
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```
Set up the proxy configuration so that it is loaded by nginx:
```
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl enable nginx
```
You can verify that everything is working like so:
```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```
> You should receive a 200 OK response.
 
### Set up RBAC for Kubelet Authorization

Create a role with the necessary permissions:
```
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```
Bind the role to the kubernetes user:
```
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```
---
## THE LOAD BALANCER
### Setting up a Kube API Frontend Load Balancer

Here are the commands you can use to set up the nginx load balancer. Run these on the server that you have designated as your load balancer server:
```
sudo apt-get install -y nginx
sudo systemctl enable nginx
sudo mkdir -p /etc/nginx/tcpconf.d
sudo vi /etc/nginx/nginx.conf
```
Add the following to the end of nginx.conf:
```
include /etc/nginx/tcpconf.d/*;
````
Set up some environment variables for the lead balancer config file:
```
CONTROLLER0_IP=<controller 0 private ip>
CONTROLLER1_IP=<controller 1 private ip>
```
Create the load balancer nginx config file:
```
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server $CONTROLLER0_IP:6443;
        server $CONTROLLER1_IP:6443;
    }

    server {
        listen 6443;
        listen 443;
        proxy_pass kubernetes;
    }
}
EOF
```
Reload the nginx configuration:
```
sudo nginx -s reload
```
You can verify that the load balancer is working like so:
```
curl -k https://localhost:6443/version
```
You should get back some json containing version information for your Kubernetes cluster.

## KUBERNETES WORKERS

### Installing Worker Node Binaries

```
sudo apt-get -y install socat conntrack ipset

wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet

sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

chmod +x kubectl kube-proxy kubelet runc.amd64 runsc

sudo mv runc.amd64 runc

sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/

sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/

sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/

sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
```

###  Configuring Containerd

You can configure the containerd service like so. Run these commands on both worker nodes:
```
sudo mkdir -p /etc/containerd/
```
Create the containerd config.taml:
```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```
Create the containerd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
### Configuring Kubelet

Set a HOSTNAME environment variable that will be used to generate your config files. Make sure you set the HOSTNAME appropriately for each worker node:
```
HOSTNAME=$(hostname)

sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```
Create the kubelet config file:
```
cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS: 
  - "10.32.0.10"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

Create the kubelet unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2 \\
  --hostname-override=${HOSTNAME} \\
  --allow-privileged=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configuring Kube-Proxy
You can configure the kube-proxy service like so. Run these commands on both worker nodes:
```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```
Create the kube-proxy config file:
```
cat << EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```
Create the kube-proxy unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
``````
Now you are ready to start up the worker node services! Run these:
```
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```
Check the status of each service to make sure they are all active (running) on both worker nodes:
```
sudo systemctl status containerd kubelet kube-proxy
```
Finally, verify that both workers have registered themselves with the cluster. Log in to one of your control nodes and run this:
```
kubectl get nodes
```
You should see the hostnames for both worker nodes listed. Note that it is expected for them to be in the NotReady state at this point.

---

## CLUSTER NETWORK
### Installing Weave Net
You can configure Weave Net like this:

First, log in to both worker nodes and enable IP forwarding:
```
sudo sysctl net.ipv4.conf.all.forwarding=1
echo "net.ipv4.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
```

The remaining commands can be done using kubectl. To connect with kubectl, you can either log in to one of the control nodes and run kubectl there or open an SSH tunnel for port 6443 to the load balancer server and use kubectl locally.

You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:
```
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```
#### Install Weave Net:

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.200.0.0/16"
```
Now Weave Net is installed, but we need to test our network to make sure everything is working.

- First, make sure the Weave Net pods are up and running:
```
kubectl get pods -n kube-system
```
This should return two Weave Net pods, and look something like this:
```
NAME              READY     STATUS    RESTARTS   AGE
weave-net-m69xq   2/2       Running   0          11s
weave-net-vmb2n   2/2       Running   0          11s
```

- Next, we want to test that pods can connect to each other and that they can connect to services. We will set up two Nginx pods and a service for those two pods. Then, we will create a busybox pod and use it to test connectivity to both Nginx pods and the service.

Create an Nginx deployment with 2 replicas:
```
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      run: nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```
Next, create a service for that deployment so that we can test connectivity to services as well:
```
kubectl expose deployment/nginx
```
Now let's start up another pod. We will use this pod to test our networking. We will test whether we can connect to the other pods and services from this pod.
```
kubectl run busybox --image=radial/busyboxplus:curl --command -- sleep 3600

POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```
Now let's get the IP addresses of our two Nginx pods:
```
kubectl get ep nginx
```
There should be two IP addresses listed under ENDPOINTS, for example:
```
NAME      ENDPOINTS                       AGE
nginx     10.200.0.2:80,10.200.128.1:80   50m
```
Now let's make sure the busybox pod can connect to the Nginx pods on both of those IP addresses.
```
kubectl exec $POD_NAME -- curl <first nginx pod IP address>
kubectl exec $POD_NAME -- curl <second nginx pod IP address>
```
Both commands should return some HTML with the title "Welcome to Nginx!" This means that we can successfully connect to other pods.

Now let's verify that we can connect to services.
```
kubectl get svc
```
This should display the IP address for our Nginx service. For example, in this case, the IP is 10.32.0.54:
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   1h
nginx        ClusterIP   10.32.0.54   <none>        80/TCP    53m
```
Let's see if we can access the service from the busybox pod!
```
kubectl exec $POD_NAME -- curl <nginx service IP address>
```
>This should also return HTML with the title "Welcome to Nginx!"

### Cleanup:
Now that we have networking set up in the cluster, we need to clean up the objects that were created in order to test the networking.
```
kubectl delete deployment busybox
kubectl delete deployment nginx
kubectl delete svc nginx
```

### Deploy Kube-DNS to the Cluster
You can open the SSH tunnel by running this in a separate terminal. Leave the session open while you are working to keep the tunnel active:

ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
You can install kube-dns like so:

kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yaml
Verify that the kube-dns pod starts up correctly:

kubectl get pods -l k8s-app=kube-dns -n kube-system
You should get output showing the kube-dns pod. It should look something like this:

NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-598d7bf7d4-spbmj   3/3       Running   0          36s
Make sure that 3/3 containers are ready, and that the pod has a status of Running. It may take a moment for the pod to be fully up and running, so if READY is not 3/3 at first, check again after a few moments.

Now let's test our kube-dns installation by doing a DNS lookup from within a pod. First, we need to start up a pod that we can use for testing:

kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
Next, run an nslookup from inside the busybox container:

kubectl exec -ti $POD_NAME -- nslookup kubernetes
You should get output that looks something like this:

Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
If nslookup succeeds, then your kube-dns installation is working!



Once you are done, it's probably a good idea to clean up the the objects that were created for testing:

kubectl delete deployment busybox

---
---
## ERRORS:
####  Kubelet service status not active "code=exited status=255"
```
# Turn off memory swap:
sudo swapoff -a

# Remove references from ftab
sudo nano /etc/ftab
```
#### NGINX bucket size error
> edit /etc/nginx/nginx/conf 
```
 # Look for lhttp section:
 http  {
server_names_hash_bucket_size 64
```