# Working with Secrets

## Prerequisites

- Running Kubernetes (ie. Minikube)
- Create a namespace for this lab
```
kubectl create namespace lab-7
```

## Lab

1. Change into the directoy labs/lab-7/

2. Deploy the ConfigMap for the NGINX webserver
```
kubectl create configmap nginx-config --from-file configs/nginx/conf.d -n lab-7
```

Take a look at the configuration file (configs/nginx/conf.d/nginx-with-ssl.conf)

- On which port will the server listen on?
- In which directory will the server look for the certificate and key?

3. Deploy the secret, the deployment and the service from deployment.yml
```
kubectl apply -f deployment.yml
```

Take a look at the deployment.yml file

- In which directory will the certificate files from the secret mounted into the Pod?
- What is the full path to the certificate inside the Pod?
- On which port is the container listening?

4. Check if the service is reachable from outside by entering the URL, which is displayed for the service by the command below, into a browser.

**Important**: The command has no idea about the usage of HTTPS for this service, thus you will have to change the http:// prefix to https:// manually!

```
minikube service list
```

- Did you get a certificate warning? Why?

5. (Optional) Cleanup

Cleanup the namespace including all ressources inside that namespace

```
kubectl delete namespaces lab-7
```