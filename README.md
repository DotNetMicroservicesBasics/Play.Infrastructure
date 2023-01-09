# Play.Infrastructure
Play Economy Infrastructure components

## Add the GitGub package source
```powershell
$owner="DotNetMicroservicesBasics"
$gh_pat="PAT HERE"

dotnet nuget add source --username yasserMokh --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group
```powershell
$appname="playeconomy"
az group create --name $appname --location eastus
```

## Ceating the Cosmos DB account
```powershell
$dbname="playeconomycosmosdb"
az cosmosdb create --name $dbname --resource-group $appname --kind MongoDB --enable-free-tier 
```

## Ceating the Azure Service Bus Namespace
```poweshell
$sbname="playeconomyservicebusnamespace"
az servicebus namespace create --name $sbname --resource-group $appname --sku Standard
```

## Ceating the Azure Container Registry
```poweshell
$acrname="playeconomyazurecontainerregistry"
az acr create --name $acrname --resource-group $appname --sku Basic
```

## Creating the Azure Kubernetes Service (AKS) cluster
```powershell
az extension add --name aks-preview
az feature register --name "EnableWorkloadIdentityPreview" --namespace "Microsoft.ContainerService"

# Wait for this command to reach "Registered" state
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableWorkloadIdentityPreview')].{Name: name, State: properties.state}"


az provider register --namespace Microsoft.ContainerService

$aksname="playeconomyakscluster"
az aks create -n $aksname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $acrname --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys

az aks get-credentials --name $aksname --resource-group $appname

# Test Creation & Connection Success
kubectl version
kubectl cluster-info
```

## Creating the Azure Key Vault
```powershell
$kvname="playeconomyazurekeyvault"
az keyvault create -n $kvname -g $appname

# add a secret to your newly created key vault
az keyvault secret set --vault-name $kvname --name "[KEY HERE]" --value "[VALUE HERE]"
```

## Installing Emissary-ingress
```powershell
helm repo add datawire https://app.getambassador.io
helm repo update


kubectl apply -f https://app.getambassador.io/yaml/emissary/3.3.1/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

$emnamespace="emissary"
kubectl create namespace $emnamespace
$dnsname="playeconomygateway"
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$dnsname --namespace $emnamespace  
kubectl rollout status deployment/emissary-ingress -n $emnamespace -w

#Check pods & service
kubectl get pods -n $emnamespace
kubectl get service emissary-ingress -n $emnamespace
```

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $emnamespace

kubectl apply -f .\emissary-ingress\mappings.yaml -n $emnamespace
```

## Installing cert-manager
```powershell
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager --version v1.10.1 --set installCRDs=true --namespace $emnamespace
```

## Create the cluster issuer
```powershell
kubectl apply -f .\cert-manager\cluster-issuer.yaml -n $emnamespace

# Check status
kubectl get clusterissuer -n $emnamespace

kubectl apply -f .\cert-manager\acme-challenge.yaml -n $emnamespace
```

## Create the tls certificate
```powershell
kubectl apply -f .\emissary-ingress\tls-certificate.yaml -n $emnamespace

kubectl get certificate -n $emnamespace

kubectl describe certificate -n $emnamespace

kubectl get secret -n $emnamespace

kubectl get secret playeconomy-tls -n $emnamespace -o yaml
```

## Enabling TLS & HTTPS
```powershell
kubectl apply -f .\emissary-ingress\host.yaml -n $emnamespace
```

## Package & Publich the microservice Helm Chart
```powershell
helm package .\helm\microservice

$helmUser=[guid]::Empty.Guid
$helmPassword=az acr login --name $acrname --expose-token --query accessToken -o tsv

helm registry login "$acrname.azurecr.io" --username $helmUser --password $helmPassword

helm push microservice-0.1.2.tgz oci://$acrname.azurecr.io/helm
```

## Create Github service principal
```powershell
$appID= az ad sp create-for-rbac -n "Github" --query appId -o tsv

az role assignment create --assignee $appID --role "AcrPush" --resource-group $appname
az role assignment create --assignee $appID --role "Azure Kubernetes Service Cluster User Role" --resource-group $appname
az role assignment create --assignee $appID --role "Azure Kubernetes Service Contributor Role" --resource-group $appname
```

## Deploy Seq to AKS
```powershell
$obsvnamesapce="observability"
helm repo add datalust https://helm.datalust.co
helm repo update

helm install seq datalust/seq -n $obsvnamesapce --create-namespace
```

## Deploy Jaeger to AKS
```powershell
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm repo update

helm upgrade jaeger jaegertracing/jaeger --values .\jaeger\values.yaml -n $obsvnamesapce --install 
```