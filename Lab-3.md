# Deploying an NGINX Pod with custom configuration

## Prerequisites

- Running Kubernetes (ie. Minikube)
- (Optional) Installed jq (https://stedolan.github.io/jq/)
- Create a namespace for this lab
```
kubectl create namespace lab-3
```

## Lab

1. Change into the directoy labs/lab-3/

2. Deploy the manifest on your cluster
```
kubectl apply -f nginx.yml -n lab-3
```

3. Check, if the Pod is running and its status:
```
kubectl get pods -n lab-3
```

4. Get detailed information about the running Pod. Can you see, that the Pod now listens on port 8080?
```
kubectl describe pod nginx -n lab-3
```

5. Open the manifest file (nginx.yml) and take a look at the content
- You'll see a new ressource named ConfigMap

6. Check if the website is correctly displayed inside the Pod. You need to port-forward the internal port to your localhost
```
kubectl port-forward -n lab-3 pods/nginx 8080:8080
```

This opens port 8080 on your local machine and tunnels all requests to this port to the Pod's port 8080. Open a browser and connect to http://localhost:8080. You should see a message from NGINX.

7. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-3
```
