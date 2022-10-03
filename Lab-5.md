# Create a deployment of an application make  and scale the deployment

## Prerequisites

- Running Kubernetes (ie. Minikube)
- Create a namespace for this lab
```
kubectl create namespace lab-5
```

## Lab
1. Change into the directoy labs/lab-5/

2. Deploy the manifest on your cluster
```
kubectl apply -f coffee-deployment.yml -n lab-5
```

3. Check the deployment
```
kubectl -n lab-5 get deployments.apps coffee
```

- How many pods are running?

4. Get the list of pods running:
```
kubectl get pods -n lab-5
```
- How are pods named? 
- How many pods are running?

5. Get detailed information about the running Pod
```
kubectl describe deployment coffee -n lab-5
```
- On which port are the containers listening?
- Did the deployment create a ReplicaSet?
- How many replicas are running?

6. Scale the deployment up to 5 pods
```
kubectl -n lab-5 scale deployment coffee --replicas 5
```

7. Check if the scaling did work:
```
kubectl get deployment coffee -n lab-5
kubectl get pods -n lab-5
```

8. Open the manifest file (coffee-deployment.yml) and verify if the deployment is reachable from outside the cluster.

*Note*: We configured the service as type NodePort, which binds the service to a port within the range 30000-32767 on each node of the cluster (Minikube has just one node ;-)).

*Important*: Because of how Minikube works inside Docker the actual address and port which is reachable from outside must be determined by issuing the command

``` 
minikube service list
``` 

Check if the service is reachable from outside by entering the URL, which is displayed for the service by the command above, into a browser.

Note the pod name (Server name:) on the website (coffee-XXXXXXXX-xxxxx), you'll need it in the next step

9. Delete the pod from above and check immediately whats happening to the pods by issuing the following two commands at once:
```
kubectl delete pod coffee-XXXXXXXX-xxxxx -n lab-5; kubectl get pods -n lab-5
```

Refresh the browser page. What happened to the Server name?

9. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-5
```