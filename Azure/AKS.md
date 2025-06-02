The following comes from:

https://github.com/HoussemDellai/aks-course/blob/main/02_kubernetes_aks/Readme.md

# Azure AKS Cheatsheet

##### Creating the cluster
```sh
az group create --name rg-aks-cluster --location swedencentral
az aks create -n aks-cluster -g rg-aks-cluster --network-plugin azure --network-plugin-mode overlay --generate-ssh-keys
```

##### Setting the environment

Once the cluster is created, to interact with it using kubectl you will need to set proper environment.

A tip would be to go to the AKS cluster and click on "Connect", that will pop a right-hand side window in which you will find the complete commands with the subscription ID and resource group:

```
az aks get-credentials --resource-group rg-aks-cluster --name aks-cluster
```

##### Show the kube config file
`cat C:\Users\<USER>\.kube\config`

##### Kubernetes cluster nodes details
```sh
kubectl get nodes -o wide
NAME                                STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-71318377-vmss000000   Ready    <none>   7m55s   v1.31.8   10.224.0.6    <none>        Ubuntu 22.04.5 LTS   5.15.0-1088-azure   containerd://1.7.27-1
aks-nodepool1-71318377-vmss000001   Ready    <none>   7m54s   v1.31.8   10.224.0.4    <none>        Ubuntu 22.04.5 LTS   5.15.0-1088-azure   containerd://1.7.27-1
aks-nodepool1-71318377-vmss000002   Ready    <none>   7m55s   v1.31.8   10.224.0.5    <none>        Ubuntu 22.04.5 LTS   5.15.0-1088-azure   containerd://1.7.27-1
```

##### List namespaces
```sh
kubectl get ns
NAME              STATUS   AGE
default           Active   11m
kube-node-lease   Active   11m
kube-public       Active   11m
kube-system       Active   11m
```

##### List services/pods/deployments... of a certain namespace:
```sh
kubectl get svc --namespace kube-system
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kube-dns         ClusterIP   10.0.0.10      <none>        53/UDP,53/TCP   12m
metrics-server   ClusterIP   10.0.177.222   <none>        443/TCP         12m
```

Or just everything from a namespace

```
kubectl get all --namespace kube-system
(long listing of pods, services, daemonsets, deployments and replicasets)
```


##### Deploy Pods from a public registry

```sh
kubectl run nginx --image=nginx  
pod/nginx created 
```

##### List pods

```sh 
kubectl get pods -o wide  
NAME    READY   STATUS    IP           NODE                             
nginx   1/1     Running   10.244.2.3   aks-agentpool-18451317-vmss000001
```

##### Exec into a pod

```
kubectl exec -it nginx -- /bin/bash
root@nginx:/# ls /var/log/
apt  btmp  dpkg.log  faillog  lastlog  nginx  wtmp
root@nginx:/# exit
exit
```

##### Delete a pod

```
kubectl delete pod nginx    
pod "nginx" deleted
```

##### Create a service to expose pods

```sh 
kubectl expose pod nginx --type=LoadBalancer --port=80
service/nginx exposed
```
 
View the Service public IP: `135.116.14.220`.

```sh 
kubectl get svc
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP        16m
nginx        LoadBalancer   10.0.12.164   135.116.14.220   80:30551/TCP   15s
```

#### Deployment

<ins>ReplicaSet</ins> is just about scalability
<ins>Deployment</ins> is about setting variables, mounts and more complete set of features on top of ReplicaSets

##### Create deployment file
`kubectl create deployment nginx-deploy --image=nginx --replicas=3 -o yaml --dry-run=client > nginx-deploy.yaml`

That will create the following contents:
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

##### Apply deployment from file
```
kubectl apply -f .\nginx-deploy.yaml
deployment.apps/nginx-deploy created
```

##### Delete deployment from file
```
kubectl delete -f .\nginx-deploy.yaml
deployment.apps "nginx-deploy" deleted
```

##### List deployments and their pods
```
kubectl get deploy,po -o wide
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx-deploy   3/3     3            3           66s   nginx        nginx    app=nginx-deploy

NAME                                READY   STATUS    RESTARTS   AGE     IP             NODE                                NOMINATED NODE   READINESS GATES
pod/nginx-deploy-5fd7574f9f-2qwws   1/1     Running   0          66s     10.244.2.32    aks-nodepool1-71318377-vmss000001   <none>           <none>
pod/nginx-deploy-5fd7574f9f-w82d5   1/1     Running   0          66s     10.244.1.233   aks-nodepool1-71318377-vmss000000   <none>           <none>
pod/nginx-deploy-5fd7574f9f-xsf4f   1/1     Running   0          66s     10.244.0.216   aks-nodepool1-71318377-vmss000002   <none>           <none>
```

If a pod is killed, another one is automatically replaced:
```
kubectl delete pod nginx-deploy-5fd7574f9f-2qwws     
pod "nginx-deploy-5fd7574f9f-2qwws" deleted

kubectl get deploy,po -o wide
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx-deploy   3/3     3            3           5m35s   nginx        nginx    app=nginx-deploy

NAME                                READY   STATUS    RESTARTS   AGE     IP             NODE                                NOMINATED NODE   READINESS GATES
pod/nginx-deploy-5fd7574f9f-8hht2   1/1     Running   0          6s      10.244.2.123   aks-nodepool1-71318377-vmss000001   <none>           <none>
pod/nginx-deploy-5fd7574f9f-w82d5   1/1     Running   0          5m35s   10.244.1.233   aks-nodepool1-71318377-vmss000000   <none>           <none>
pod/nginx-deploy-5fd7574f9f-xsf4f   1/1     Running   0          5m35s   10.244.0.216   aks-nodepool1-71318377-vmss000002   <none>           <none>
```

<br>
<br>
<br>
<br>
<br>

## 4. Building and deploying container images in ACR

Instead of using a public registry like `Docker Hub`, you can create your own private registry in Azure where you push your own private images.

### 4.1. Creating a container registry in Azure (ACR)

Follow this link to create an `ACR` using the portal: [docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal).
Or this link to create it through the command line: [docs.microsoft.com/en-us/azure/container-registry/container-registry-event-grid-quickstart](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-event-grid-quickstart).

```sh
az acr create --name acraks013579 -g rg-aks-cluster --sku Standard
```

### 4.1. Attach the ACR to the AKS cluster

`AKS` needs to be attached to the `ACR` to be able to pull images from it.

```sh
az aks update --name aks-cluster --resource-group rg-aks-cluster --attach-acr acraks013579
```

### 4.2. Building an image in ACR

You can use `docker build` to build an image in your local machine, assuming you have docker installed.
However, there is another simple option. You can use Azure Container Registry (ACR).
Navigate into the `app-dotnet` folder and run the following command to package the source code, upload it into ACR and build the docker image inside ACR:

```sh 
$acrName="<your acr name>" 
az acr build -t "$acrName.azurecr.io/dotnet-app:1.0.0" -r $acrName ../app-dotnet
```

This will build and push the image to ACR.

### 4.3. Deploying an image from ACR

Deploy the created image in ACR into the AKS cluster and replace image and registry names:

```sh 
kubectl run dotnet-app --image=<your registry id>.azurecr.io/dotnet-app:1.0.0
pod/dotnet-app created
```

Verify the pod deployed successfully.

```sh 
kubectl get pods
# NAME         READY   STATUS    RESTARTS
# dotnet-app   1/1     Running   0       
# nginx        1/1     Running   0       
```

Expose the pod on a public IP address.

```sh 
kubectl expose pod dotnet-app --type=LoadBalancer --port=80
# service/dotnet-app exposed
```

View the created service.

```sh 
kubectl get svc
# NAME         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)     
# dotnet-app   LoadBalancer   10.0.202.46   <pending>       80:31774/TCP
# kubernetes   ClusterIP      10.0.0.1      <none>          443/TCP     
# nginx        LoadBalancer   10.0.147.78   20.61.145.135   80:32640/TCP
```

Note how the service creation is in `Pending` state. 
That is because it takes few seconds to create the public IP address and attach it to the Load Balancer.
Keep watching for the service until it will be created. Use the `-w` or `--watch`.
 
```sh 
kubectl get svc -w
# NAME         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)     
# dotnet-app   LoadBalancer   10.0.202.46   52.142.237.17   80:31774/TCP
# kubernetes   ClusterIP      10.0.0.1      <none>          443/TCP     
# nginx        LoadBalancer   10.0.147.78   20.61.145.135   80:32640/TCP
```

## 5. Creating kubernetes YAML manifest files

Use the kubectl command line to generate a YAML manifest for a Pod.

```sh 
kubectl run nginx-yaml --restart=Never --image=nginx -o yaml --dry-run=client > nginx-pod.yaml
```

Deploy the YAML manifest to AKS:

```sh 
kubectl apply -f .\nginx-pod.yaml
# pod/nginx-yaml created
```

Verify the Pods created by YAML manifest are running.

```sh 
kubectl get pods
# NAME         READY   STATUS    RESTARTS   AGE
# dotnet-app   1/1     Running   0          9m19s
# nginx        1/1     Running   0          21m
# nginx-yaml   1/1     Running   0          9s
```

## Conclusion

You learned in this lab how to build, push and deploy a custom container image into Kubernetes.