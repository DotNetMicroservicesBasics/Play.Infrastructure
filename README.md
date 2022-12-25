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