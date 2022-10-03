# Deploying an NGINX Pod with custom configuration - Deploy an existing configuration file

## Prerequisites

- Running Kubernetes (ie. Minikube)
- (Optional) Installed jq (https://stedolan.github.io/jq/)
- Create a namespace for this lab
```
kubectl create namespace lab-4
```

## Lab

1. Change into the directoy labs/lab-4/

2. Create a ConfigMap ressource with the content of an NGINX configuration file
```
kubectl create configmap nginx-config --from-file configs/nginx/conf.d -n lab-4
```

3. Deploy the Pod manifest on your cluster
```
kubectl apply -f nginx.yml -n lab-4
```

3. Check, if the Pod is running and its status:
```
kubectl get pods -n lab-4
```

4. Get detailed information about the running Pod. Can you see, that the Pod now listens on port 8080?
```
kubectl describe pod nginx -n lab-4
```

5. Open the manifest file (nginx.yml) and take a look at the differences to Lab-3
- Now, we are mounting an existing ConfigMap into your Pod

6. Check if the website is correctly displayed inside the Pod. You need to port-forward the internal port to your localhost
```
kubectl port-forward -n lab-4 pods/nginx 8080:8080
```

This opens port 8080 on your local machine and tunnels all requests to this port to the Pod's port 8080. Open a browser and connect to http://localhost:8080. You should see a message from NGINX.

7. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-4
```
