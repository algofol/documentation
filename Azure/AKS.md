The following comes from:

https://github.com/HoussemDellai/aks-course/blob/main/02_kubernetes_aks/Readme.md

# Azure AKS Cheatsheet

Creating the cluster
```sh
az group create --name rg-aks-cluster --location swedencentral
az aks create -n aks-cluster -g rg-aks-cluster --network-plugin azure --network-plugin-mode overlay
```

Setting the environment
```
az account set --subscription <SUBSCRIPTION_ID>
az aks get-credentials --resource-group <RG_NAME> --name <AKS_CLUSTER_NAME>
```

Show the kube config file
`cat C:\Users\<USER>\.kube\config`

## Kubernetes cluster details
`kubectl get nodes -o wide`


# Deploying Applications into Kubernetes/AKS

## Prerequisites

You will need the following tools:

1. Kubectl CLI: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)
2. AKS (or any Kubernetes) cluster
3. Owner or Contributor of an Azure subscription: [http://azure.com/free](http://azure.com/free)

## 1. Setting up the environment

### Creating an AKS cluster in Azure

```sh
az group create --name rg-aks-cluster --location swedencentral
az aks create -n aks-cluster -g rg-aks-cluster --network-plugin azure --network-plugin-mode overlay
```

Next, the kubectl CLI will be used to deploy applications to the cluster.
This command needs to be connected to AKS.
To do that we use the following command:

```sh
az aks get-credentials --resource-group rg-aks-cluster --name aks-cluster
```

Check that the connection was successfull by listing the nodes inside the cluster:

```sh
kubectl get nodes
--------------------
NAME                       STATUS   ROLES   VERSION
aks-nodepool1-31718369-0   Ready    agent   v1.12.8
```

## 2. Deploying Pods from a public registry

```sh
kubectl run nginx --image=nginx  
pod/nginx created 
```

List the created pod, view its private IP address of the Pod and its host node.

```sh 
kubectl get pods -o wide  
NAME    READY   STATUS    IP           NODE                             
nginx   1/1     Running   10.244.2.3   aks-agentpool-18451317-vmss000001
```

## 3. Creating a service to expose pods

Now you have a pod running in Kubernetes and you want to expose it on internet through a public IP address.

>In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster.

Create a Service object to expose the Pod through public IP and Load Balancer.

```sh 
kubectl expose pod nginx --type=LoadBalancer --port=80
# service/nginx exposed
```
 
View the Service public IP: `20.61.145.135`.

```sh 
kubectl get svc
# NAME         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)     
# kubernetes   ClusterIP      10.0.0.1      <none>          443/TCP     
# nginx        LoadBalancer   10.0.147.78   20.61.145.135   80:32640/TCP
```

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