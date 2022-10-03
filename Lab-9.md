# (NOT WORKING) Deploying an NGINX ingress controller (NGINX Ingress Controller maintained by NGINX)


## Prerequisites

- Running Kubernetes (ie. Minikube)
- You need administrative rights on the machine running Minikube. You must be able to bind to priviledged ports on the host
- Create a namespace for this lab
```
kubectl create namespace lab-8
```

## Lab


1. Deploy the coffee and tea deployment
```
kubectl -n lab-8 apply -f coffee-deployment.yml
kubectl -n lab-8 apply -f tea-deployment.yml
```

2. Verify if the pods are up and running
```
kubectl -n lab-8 get pods
```

- How many Pods are running in the lab-8 namespace?

3. Clone the NGINX Kubernetes Ingress Controller Repository to your local machine and change into the repo
```
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.3.1
cd kubernetes-ingress/deployments
```

4. Deploy the NGINX Kubernetes Ingress Controller

cd kubernetes-ingress/deployments

```
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f common/ingress-class.yaml
```
```
kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
```
```
kubectl apply -f deployment/nginx-ingress.yaml
```
```
kubectl create -f service/nodeport.yaml
```
```
kubectl create secret docker-registry regcred --docker-server=private-registry.nginx.com --docker-username=<JWT Token> --docker-password=none -n nginx-ingress
kubectl get secret regcred --output=yaml -n nginx-ingress
```


98. Deploy the Ingress rules
```
kubectl -n lab-8 apply -f ingress.yml
```

99. Open a browser to check if the ingress ressource works. You should be able reach the two services via the following URLs:

- http://localhost/coffee
- http://localhost/tea

- What is the server name of the Pod you are connected to?
- Does the server name change, when you refresh the page?

