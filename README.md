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