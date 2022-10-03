# Deploying an NGINX ingress controller

## Prerequisites

- Running Kubernetes (ie. Minikube)
- You need administrative rights on the machine running Minikube. You must be able to bind to priviledged ports on the host
- Enable Minikube NGINX Ingress Controller
```
minikube addons enable ingress
```
- Create a namespace for this lab
```
kubectl create namespace lab-8
```

## Lab


git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.3.1

cd kubernetes-ingress/deployments

kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f common/ingress-class.yaml

kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml

kubectl apply -f deployment/nginx-ingress.yaml

kubectl create -f service/nodeport.yaml





kubectl create secret docker-registry regcred --docker-server=private-registry.nginx.com --docker-username=<JWT Token> --docker-password=none -n nginx-ingress

kubectl get secret regcred --output=yaml -n nginx-ingress









kubectl create secret docker-registry regcred --docker-server=private-registry.nginx.com --docker-username=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InRyaWFsIn0.eyJpc3MiOiJuZ2lueCBpc3N1ZXIiLCJpYXQiOjE2NjQ4MTM4ODIsImp0aSI6Ijg5NDUiLCJzdWIiOiJUMDAwMTI1ODc2IiwiZXhwIjoxNjY3NDA1ODgyfQ.lBULpzPFyyXzmlpgILRClMbLGr8qM7oO51CelHcBvTeV_NC70vAqiIauPvHwBU46eqS3vy_kzdTwbz6caAn4hWKy2qk5aTuoIKWxJifeFntCUaWYJoARwwTSLZ5iI1kUG5SpUB7n-Dj2Nylro1L8kqsgV8-UDXhcxdLV14qBhPsuZmxPEx-TlvELmqIlJCWcVW0KF2yZzYccHtC_QiLMuAAB9nu3XoGJDCYj5Jglw5FWPrjIoGuPPXOctydNoq7Xh2_y0mh15Q--wfiGKpaHb9IXnsQyuY3L4vuUM8YUY39KJ1JngN5Q9oDHNjlXjBv06Ig_Uu9xYaAAETzxE63cg4Iqx7Z4iW-9CSk2MvKhLFi70KH2JGX6lXaahouQYIt_GN56whV2gGnfoIL30FwtBH49UU70tzgvfWDxHgF9AHLIAmXnls64Z438kROKaX6FRDBxqOz8YEiVjI7QPnRDF-ok9srHCPvrRtId-iVqD7-aO0Z_solF0ox2UG4uoOEq --docker-password=none -n nginx-ingress



