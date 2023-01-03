
Framework : .NET6
Azure Functions SDK : 4.1.3



## Storage account config

```
$sid="c1537527-1234-1234-1234-123412341234"
az login
az account set --subscription $sid

$rg="codit-arc-surface"
$storage="wfv0"

az storage account create -n $storage -g $rg -l westeurope --sku Standard_LRS

$storageCN=(az storage account show-connection-string --name $storage --resource-group $rg --query connectionString -o tsv)

$storageCN = "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=wfv0;AccountKey=ABC/9IXD815/cJbeMq+ABC/CwXQMffC+ABC==;"

```

## V0 - on localhost

update the local.setting.json to use Azure storage account (if azurite still not working properly)

```
dotnet build -c release
dotnet publish -c release
func host start

# get the masterkey (azure-webjobs-secrets)
$masterKey = "vYA13Bec8Y82HJr-0SMsJBIck5O3sTF4hoRwPLBa8p6UAzFuQ-5r-A=="

# get the LogicApps URL
$sdkWebhook = "http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/helloworkflow/triggers/manual/listCallbackUrl?api-version=2020-05-01-preview&code=$masterKey"

# get the LogicApps URL
curl --location --request POST $sdkWebhook --header "Content-Length: 0" -v

# workflow URL	(vscode)
$wfURL="http://localhost:7071/api/helloworkflow/triggers/manual/invoke?api-version=2022-05-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=RmsOS3O512qT3Nf4Nb_lREuiglOA1vledP453CnLZto"

# workflow URL	(terminal)
$wfURL="http://localhost:7071/api/helloworkflow/triggers/manual/invoke?api-version=2022-05-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=9k-dgmOe2Cn7MSfStP71sWG2-4dfw1TYaBd9eRqIXc4"

# invoke
curl --location --request POST $wfURL --header 'Content-Type: application/json' --header 'x-format: expand'  --data-raw '{"itemId": 1000}' -v

1..10 | % { curl --location --request POST $wfURL --header 'Content-Type: application/json' --header 'x-format: expand'  --data-raw '{"itemId": 1000}' -v }

```

## V0 - docker on localhost

```
docker build .\ --tag la-standard-dotnet:0.0.6-app -f .\Dockerfile --no-cache

# workflow not loaded
docker run -d -p 5002:80 -e "WEBSITE_HOSTNAME=localhost" -e "WEBSITE_SITE_NAME=helloworkflow" -e "AzureWebJobsStorage=$storageCN" -e "APPINSIGHTS_INSTRUMENTATIONKEY=556bd798-0734-4dae-a0a0-66d4ed23dff8" -e "AZURE_FUNCTIONS_ENVIRONMENT=Development" -e "AzureFunctionsJobHost__extensionBundle__version=[1.*, 2.0.0)" -e "AzureFunctionsJobHost__extensionBundle__id=Microsoft.Azure.Functions.ExtensionBundle.Workflows" la-standard-dotnet:0.0.6-app

# not ok
docker run -d -p 5002:80 -e "WEBSITE_HOSTNAME=localhost" -e "WEBSITE_SITE_NAME=helloworkflow" -e "AzureWebJobsStorage=$storageCN" -e "APPINSIGHTS_INSTRUMENTATIONKEY=556bd798-0734-4dae-a0a0-66d4ed23dff8" -e "AZURE_FUNCTIONS_ENVIRONMENT=Development" -e "AzureFunctionsJobHost__extensionBundle__version=[1.*, 2.0.0)" -e "AzureFunctionsJobHost__extensionBundle__id=Microsoft.Azure.Functions.ExtensionBundle.Workflows" -e "AzureFunctionsJobHost__extensions__workflow__settings__Runtime.WorkflowOperationDiscoveryHostMode=true" la-standard-dotnet:0.0.6-app


docker ps

docker logs ebb2ea97ac6c

docker stop ebb2ea97ac6c

docker exec -it ebb2ea97ac6c bash
ls -a -l
```


## V0 - on kubernetes

```
# create namespace and shared resources
kubectl create namespace wf-v0

# deploy all the content of the dev folder
kubectl --namespace 'wf-v0' apply --filename manifests/v0/
```

```
kubectl port-forward logicapps-k8s-v0-6f9fd4cfbf-26wbz 80:80 -n wf-v0
```