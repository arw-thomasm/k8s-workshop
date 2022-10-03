# Deploying a Pod

## Prerequisites

- Running Kubernetes (ie. Minikube)
- (Optional) Installed jq (https://stedolan.github.io/jq/)
- Create a namespace for this lab
```
kubectl create namespace lab-1
```

## Lab

1. Change into the directoy labs/lab-1/
2. Open the manifest file (nginx.yml) for your first Pod and take a look at the content
3. Deploy the manifest on your cluster
```
kubectl apply -f nginx.yml -n lab-1
```
4. Check, if the Pod is running and its status:
```
kubectl get pods -n lab-1
kubectl get pod nginx  -o wide -n lab-1
kubectl get pod nginx  -o yaml -n lab-1
kubectl get pod nginx  -o jsonpath="{.status}" -n lab-1 | jq
```

5. Get detailed information about the running Pod
```
kubectl describe pod nginx -n lab-1
```

6. Check if the website is correctly displayed inside the Pod. You need to port-forward the internal port to your localhost
```
kubectl port-forward -n lab-1 pods/nginx 8080:80
```

This opens port 8080 on your local machine and tunnels all requests to this port to the Pod's port 80. Open a browser and connect to http://localhost:8080. You should see the startpage of NGINX.

7. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-1
```
