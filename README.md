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