# Azure

## Create infrastructure in Azure

### Login into Azure
`az login`

### Create Resource Group
`az group create -n <ResourceGroupName> -l <region>`

Example: 

`az group create -n RG-TM-KubeCluster -l germanywestcentral`

### Upload your SSH Public Key to Azure (used to authenticate SSH later)
`az sshkey create --name “<ssh-key-name>" --public-key "@<path_to_pub_key>.pub" --resource-group <ResourceGroupName>`

Example:

`az sshkey create --name "mySSHKey" --public-key "@~/.ssh/id_rsa.pub" --resource-group RG-TM-KubeCluster`

### Create a VNET
`az network vnet create -g <ResourceGroupName> -n <VNET-Name> --address-prefixes <prefix>/16`

Example: 

`az network vnet create -g RG-TM-KubeCluster -n VNET-TM-KubeCluster --address-prefixes 172.0.0.0/16`

### Create the Master Node VM
`az vm create -n <vm-name-master> -g <ResourceGroupName> --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name <mySSHKey> --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name <dns-name-for-public-ip-master>`

Example: 

`az vm create -n tm-kube-master -g RG-TM-KubeCluster --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name mySSHKey --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name tm-kubeadm-master`

### Create Worker Node VMs
`az vm create -n <vm-name-worker-{n}> -g <ResourceGroupName> --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name <mySSHKey> --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name <dns-name-for-public-ip-worker-{n}>

Example: 

`az vm create -n tm-kube-worker-1 -g RG-TM-KubeCluster --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name mySSHKey --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name tm-kubeadm-worker-1`

# Install CRIO

### SSH into the VMs
```
ssh -l azureuser <fqdns VM>
```  

Example: 

`ssh -l azureuser tm-kubeadm-worker-2.germanywestcentral.cloudapp.azure.com`

### Add Package Repository to apt sources list
```
OS=xUbuntu_18.04
CRIO_VERSION=1.24
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list
```

### Add GPG Keys
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

### Install CRI-O
```
sudo apt-get update
sudo apt-get install cri-o cri-o-runc cri-tools -y
```

### Enable and start CRI-O Dameon
```
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```

### Test if CRI-O is running
`sudo crictl info`

# Install Kubernetes

Since Kubernetes manages the swap itself, you first need to disable swap otherwise Kubernetes may not run. Disable it and install all three core Kubernetes components (kubeadm, kubelet, and kubectl) on all the nodes using the following commands

### Turn of swap
`sudo swapoff -a`

### Install K8s components
Update the apt package index and install packages needed to use the K8s repo:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google signing key
`sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg`

Add the Kubernetes apt repo:
`echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`

Install the K8s components and pin their version:
```
sudo apt-get update
sudo apt install -y kubeadm kubelet kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

If you get the error message `[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist` then enable the kernel module br_netfilter:

`sudo modprobe br_netfilter`

If you get the error message `[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1` then enable ip forwarding by echoing 1 into the file stated in the error message:

`echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward`

Attention: Do not use `sudo echo 1 >  /proc/sys/net/ipv4/ip_forward` as this does not work, as the redirection of the output is not done as sudo!

## Master Node specific tasks

### Initialize Master
`sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans=<fqdns Master-VM>`

Important: If you are using flannel as network provider, then do not change the pod-network-cidr parameter!

This command initialized the K8s control plane and when finished it shows instruction for joining the worker nodes into the cluster:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join <ip>:6443 --token <token> \
	--discovery-token-ca-cert-hash sha256:<hash>

```

### Create kube-config for the azureuser
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install a pod network (flannel)

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

## Worker Nodes specific tasks

### Join Worker Nodes into cluster

Use the command of the output of `kubeadm init` to join the cluster

`sudo kubeadm join <ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

### Check all pods on the cluster

`kubectl get pods --all-namespaces -o wide`

Beispielausgabe
```
NAMESPACE      NAME                                     READY   STATUS    RESTARTS   AGE     IP           NODE               NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-5qd7q                    1/1     Running   0          6m56s   10.0.0.6     tm-kube-worker-2   <none>           <none>
kube-flannel   kube-flannel-ds-cx8q6                    1/1     Running   0          7m18s   10.0.0.5     tm-kube-worker-1   <none>           <none>
kube-flannel   kube-flannel-ds-w9vmm                    1/1     Running   0          16m     10.0.0.4     tm-kube-master     <none>           <none>
kube-system    coredns-565d847f94-hmwlj                 1/1     Running   0          3m41s   10.244.2.2   tm-kube-worker-2   <none>           <none>
kube-system    coredns-565d847f94-pbpvm                 1/1     Running   0          4m9s    10.244.1.2   tm-kube-worker-1   <none>           <none>
kube-system    etcd-tm-kube-master                      1/1     Running   1          17m     10.0.0.4     tm-kube-master     <none>           <none>
kube-system    kube-apiserver-tm-kube-master            1/1     Running   1          17m     10.0.0.4     tm-kube-master     <none>           <none>
```

### Allow access to the K8s API from the Internet (be careful!!!)
`az vm open-port -g RG-TM-KubeCluster -n tm-kube-master --port 6443`

# Minikube on Docker Desktop

## Installation

### Prerequisites

- Docker Desktop is installed

### Install on Linux

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

### Install on macOS

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube

### Install on Windows

PowerShell (as Administrator):
```
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

Add minikube to PATH:
``` 
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}
``` 
## Start the cluster

`minikube start`

## Start and expose the Dashboard

`minikube dashboard`
