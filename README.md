# Create infrastructure in Azure

### Login into Azure
`az login`

### Create Resource Group
`az group create -n <ResourceGroupName> -l <region>`

Example: `az group create -n RG-TM-KubeCluster -l germanywestcentral`

### Upload your SSH Public Key to Azure (used to authenticate SSH later)
`az sshkey create --name “<ssh-key-name>" --public-key "@<path_to_pub_key>.pub" --resource-group <ResourceGroupName>`

Example: `az sshkey create --name "mySSHKey" --public-key "@~/.ssh/id_rsa.pub" --resource-group RG-TM-KubeCluster`

### Create a VNET
`az network vnet create -g <ResourceGroupName> -n <VNET-Name> --address-prefixes <prefix>/16`

Example: `az network vnet create -g RG-TM-KubeCluster -n VNET-TM-KubeCluster --address-prefixes 172.0.0.0/16`

### Create the Master Node VM
`az vm create -n <vm-name-master> -g <ResourceGroupName> --vnet-name <VNET-Name> --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name <mySSHKey> --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name <dns-name-for-public-ip-master>`

Example: `az vm create -n tm-kube-master -g RG-TM-KubeCluster --vnet-name VNET-TM-KubeCluster --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name mySSHKey --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name tm-kubeadm-master`

### Create Worker Node VMs
`az vm create -n <vm-name-worker-{n}> -g <ResourceGroupName> --vnet-name <VNET-Name> --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name <mySSHKey> --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name <dns-name-for-public-ip-worker-{n}>

Example: `az vm create -n tm-kube-worker-1 -g RG-TM-KubeCluster --vnet-name VNET-TM-KubeCluster --image ubuntults --size Standard_DS2_v2 --data-disk-sizes-gb 10 --generate-ssh-keys --ssh-key-name mySSHKey --admin-username azureuser --public-ip-sku Standard --public-ip-address-dns-name tm-kubeadm-worker-1`
