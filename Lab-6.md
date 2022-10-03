# Different types of deployments - Daemonset and StatefulSet

## Prerequisites

- Running Kubernetes (ie. Minikube)
- Create a namespace for this lab
```
kubectl create namespace lab-6
```

## Lab
1. Change into the directoy labs/lab-6/

2. Deploy the DaemonSet manifest on your cluster
```
kubectl apply -f daemonset.yml -n lab-6
```
- How many pods where started? (Hint: kubectl get pods -n lab-6)
- Why was the number of pods exactly as the number you found out? (Hint: Minikube is just a 1-node cluster)
- How many pods would have been started, if your cluster had 3 worker nodes?

3. Take a look at the DaemonSet details:
```
kubectl -n lab-6 describe daemonsets.apps fluentd-elasticsearch
```
You will see, that a DaemonSet ensures, that at least one copy of the deployment runs on every node of the cluster (in our case we have just one node):

```
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
```

- What are typical services which would be running as DaemonSet on a cluster?

4. Deploy the StatefulSet manifest on your cluster
```
kubectl apply -f statefulset.yml -n lab-6
```

Repeat checking the Pods, which are started in your lab namespace:
```
kubectl -n lab-6 get pods
```
- What did you observe?
- How are the Pods named?

Kill the first pod of the Stateful set:
```
kubectl -n lab-6 delete pod database-0; kubectl -n lab-6 get pods
```

- What happened to database-0? Was it recreated with the same name (database-0)? 
- Do you remember what happened to the deleted Pod of a ReplicaSet in Lab 5?
- What are typical services which would be running as StatefulSet on a cluster?

5. Examine the StatefulSet manifest (statefulset.yml)

- Will data written to the database be lost when a pod dies?

6. Get a list of persistent volume claims in your lab-6 namespace:
```
kubectl -n lab-6 get persistentvolumeclaims
```
- How many persistent volume claims do you see?
- Why do you see exactly this number of PVC's?

7. Take a closer look to the PVC of the first Pod (database-0):
```
kubectl -n lab-6 describe persistentvolumeclaims data-database-0
```
- What is the storage class, K8s used to create the persistent volumes? (Hint: Look for StorageClass:)
- What is the capacity of the persistent volume? (Hint: Look for Capacity:)
- What is the name of the persistent volume Kubernetes created for this Pod? (Hint: Look for Volume:)

**Note**: Remember the name of the persistent volume for later steps!
```
kubectl -n lab-6 get persistentvolumeclaims data-database-0 -o custom-columns=NAME:.spec.volumeName
```

8. Check the persistent volume Kubernetes created for database-0
```
kubectl get persistentvolume <name of the persistent volume from step 7>
```
- What is the reclaim policy for this PV?

9. Delete the StatefulSet and check if all Pods where deleted as well
```
kubectl -n lab-6 delete statefulsets.apps database
kubectl -n lab-6 get pods
```

10. Check if the persistent volume created for the former database-0 Pod still exists
```
kubectl get persistentvolume <name of the persistent volume from step 7>
```
- Why does the pv still exists although the reclaim policy was set to "delete"?
- What is the status of the PV?

11. Check if there are any persistent volume claims left in the lab-6 namespace
```
kubectl get persistentvolumeclaims -n lab-6
```
- What is the status of the PVCs?

12. Re-Deploy the StatefulSet and check if all Pods are up and running again after a few seconds
```
kubectl apply -f statefulset.yml -n lab-6
kubectl get pods -n lab-6
```

13. Check the persistent volume which is mounted to database-0 Pod
```
kubectl -n lab-6 get persistentvolumeclaims data-database-0 -o custom-columns=NAME:.spec.volumeName
```
- The PV should be the same as before!

14. Delete the StatefulSet again. This time delete the PVCs as well
```
kubectl -n lab-6 delete statefulsets.apps database
kubectl -n lab-6 delete persistentvolumeclaims data-database-{0..2}
```

15. Check the status of the PVCs and PVs
```
kubectl -n lab-6 get persistentvolumeclaims
kubectl get persistentvolumes
```
- What is the status of the PVCs?
- What is the status of the PVs?

16. Create the "retain" storage class
```
kubectl apply -f storageclass-retain.yml
```

17. Create a StatefulSet with this new storage class
```
kubectl apply -f statefulset-retain.yml  -n lab-6
```

18. Check the name of the PV created for the first pod (database-0)
```
kubectl -n lab-6 get persistentvolumeclaims data-database-0 -o custom-columns=NAME:.spec.volumeName
```

19. Delete the StatefulSet including the PVCs
```
kubectl -n lab-6 delete statefulsets.apps database
kubectl -n lab-6 delete persistentvolumeclaims data-database-{0..2}
```

20. Check if the PVCs were deleted
```
kubectl -n lab-6 get persistentvolumeclaims
```

21. Check the status of the PVs
```
kubectl -n lab-6 get persistentvolumeclaims
```
- What is the status of the persistent volumes?

22. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-6
```

Cleanup persistent volumes as well:
```
for pv in $(kubectl get persistentvolumes | grep Released | awk '{print $1;}'); do kubectl delete persistentvolume $pv; done
```