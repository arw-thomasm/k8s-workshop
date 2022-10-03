# Troubleshooting a Pod Deployment

## Prerequisites

- Running Kubernetes (ie. Minikube)
- (Optional) Installed jq (https://stedolan.github.io/jq/)
- Create a namespace for this lab
```
kubectl create namespace lab-2
```

## Lab

1. Change into the directoy labs/lab-2/
2. Deploy the manifest on your cluster
```
kubectl apply -f nginx.yml -n lab-2
```
3. Check, if the Pod is running and its status:
```
kubectl get pods -n lab-2
kubectl get pod nginx  -o wide -n lab-2
kubectl get pod nginx  -o yaml -n lab-2
kubectl get pod nginx  -o jsonpath="{.status.containerStatuses}" -n lab-2 | jq
```

4. Get detailed information about the running Pod
```
kubectl describe pod nginx -n lab-2
```

5. Open the manifest file (nginx.yml) and try to find the error in this file, correct it and redeploy the Pod:
```
kubectl apply -f nginx.yml -n lab-2
```

6. Check again the status of the Pod with the commands from step 3

7. Check if the website is correctly displayed inside the Pod. You need to port-forward the internal port to your localhost
```
kubectl port-forward -n lab-2 pods/nginx 8080:80
```

This opens port 8080 on your local machine and tunnels all requests to this port to the Pod's port 80. Open a browser and connect to http://localhost:8080. You should see the startpage of NGINX.

7. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-2
```
