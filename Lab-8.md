# Deploying an NGINX ingress controller (NGINX Ingress Controller maintained by the Kubernetes project)

## Prerequisites

- Running Kubernetes (ie. Minikube)
- You need administrative rights on the machine running Minikube. You must be able to bind to priviledged ports on the host
- Enable Minikube NGINX Ingress Controller, tunnel ingress controller ports to your local machine (leave the terminal open)
```
minikube addons enable ingress
minikube tunnel
```
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

3. Deploy the Ingress rules
```
kubectl -n lab-8 apply -f ingress.yml
```

4. Open a browser to check if the ingress ressource works. You should be able reach the two services via the following URLs:

- http://localhost/coffee
- http://localhost/tea

- What is the server name of the Pod you are connected to?
- Does the server name change, when you refresh the page?

5. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-8
```