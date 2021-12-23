#### login into the kubernetes cluster
az aks get-credentials --resource-group decode-kubernetes --name decode-kubernetes-cluster 

#### Integrate an existing ACR with existing AKS clusters by supplying valid values for acr-name or acr-resource-id as below.
az aks update -n decode-kubernetes-cluster -g decode-kubernetes --attach-acr decodekubernetesacr


#### Variables for ingress setup
REGISTRY_NAME=decodekubernetesacr
SOURCE_REGISTRY=k8s.gcr.io
CONTROLLER_IMAGE=ingress-nginx/controller
CONTROLLER_TAG=v1.0.4
PATCH_IMAGE=ingress-nginx/kube-webhook-certgen
PATCH_TAG=v1.1.1
DEFAULTBACKEND_IMAGE=defaultbackend-amd64
DEFAULTBACKEND_TAG=1.5
CERT_MANAGER_REGISTRY=quay.io
CERT_MANAGER_TAG=v1.5.4
CERT_MANAGER_IMAGE_CONTROLLER=jetstack/cert-manager-controller
CERT_MANAGER_IMAGE_WEBHOOK=jetstack/cert-manager-webhook
CERT_MANAGER_IMAGE_CAINJECTOR=jetstack/cert-manager-cainjector

az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$CONTROLLER_IMAGE:$CONTROLLER_TAG --image $CONTROLLER_IMAGE:$CONTROLLER_TAG
az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$PATCH_IMAGE:$PATCH_TAG --image $PATCH_IMAGE:$PATCH_TAG
az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG --image $DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG
az acr import --name $REGISTRY_NAME --source $CERT_MANAGER_REGISTRY/$CERT_MANAGER_IMAGE_CONTROLLER:$CERT_MANAGER_TAG --image $CERT_MANAGER_IMAGE_CONTROLLER:$CERT_MANAGER_TAG
az acr import --name $REGISTRY_NAME --source $CERT_MANAGER_REGISTRY/$CERT_MANAGER_IMAGE_WEBHOOK:$CERT_MANAGER_TAG --image $CERT_MANAGER_IMAGE_WEBHOOK:$CERT_MANAGER_TAG
az acr import --name $REGISTRY_NAME --source $CERT_MANAGER_REGISTRY/$CERT_MANAGER_IMAGE_CAINJECTOR:$CERT_MANAGER_TAG --image $CERT_MANAGER_IMAGE_CAINJECTOR:$CERT_MANAGER_TAG


#### create public ip within the cluster infrastructure resource group ( cluster infra resource group)

az network public-ip create --resource-group MC_decode-kubernetes_decode-kubernetes-cluster_eastus --name decode-kubernetes-cluster-public-lb-static-ip --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv

O/P: 13.82.65.43


#### Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update


#### Method 1: Set the DNS label using the Azure CLI

#### Set variable for ACR location to use for pulling images
ACR_URL=decodekubernetesacr.azurecr.io
#### Public IP address of your ingress controller
STATIC_IP=13.82.65.43
#### Name to associate with public IP address
DNSNAME=collab-azure

#### Use Helm to deploy an NGINX ingress controller
helm upgrade nginx-ingress ingress-nginx/ingress-nginx \
    --namespace decode-services --create-namespace --debug --wait --timeout 30m \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.image.registry=$ACR_URL \
    --set controller.image.image=$CONTROLLER_IMAGE \
    --set controller.image.tag=$CONTROLLER_TAG \
    --set controller.image.digest="" \
    --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.image.registry=$ACR_URL \
    --set controller.admissionWebhooks.patch.image.image=$PATCH_IMAGE \
    --set controller.admissionWebhooks.patch.image.tag=$PATCH_TAG \
    --set controller.admissionWebhooks.patch.image.digest="" \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.image.registry=$ACR_URL \
    --set defaultBackend.image.image=$DEFAULTBACKEND_IMAGE \
    --set defaultBackend.image.tag=$DEFAULTBACKEND_TAG \
    --set defaultBackend.image.digest="" --set controller.service.loadBalancerIP=$STATIC_IP --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$DNS_LABEL


	
	
#### Public IP address of your ingress controller
IP="13.82.65.43"

#### Name to associate with public IP address
DNSNAME="collab-azure"



#### Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

#### Update public ip address with DNS name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME

#### Display the FQDN
az network public-ip show --ids $PUBLICIPID --query "[dnsSettings.fqdn]" --output tsv

o/p: collab-azure.eastus.cloudapp.azure.com


#### Change the Loadbalancer IP with the Azure 

### Install cert-manager

#### Label the decode-services namespace to disable resource validation
kubectl label namespace decode-services cert-manager.io/disable-validation=true

#### Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

#### Update your local Helm chart repository cache
helm repo update

#### Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace decode-services \
  --version $CERT_MANAGER_TAG \
  --set installCRDs=true \
  --set nodeSelector."kubernetes\.io/os"=linux \
  --set image.repository=$ACR_URL/$CERT_MANAGER_IMAGE_CONTROLLER \
  --set image.tag=$CERT_MANAGER_TAG \
  --set webhook.image.repository=$ACR_URL/$CERT_MANAGER_IMAGE_WEBHOOK \
  --set webhook.image.tag=$CERT_MANAGER_TAG \
  --set cainjector.image.repository=$ACR_URL/$CERT_MANAGER_IMAGE_CAINJECTOR \
  --set cainjector.image.tag=$CERT_MANAGER_TAG
  
  				
				

Useful Links:
https://docs.microsoft.com/en-us/azure/aks/ingress-tls?tabs=azure-cli
https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip
https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/