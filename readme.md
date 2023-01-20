
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
$masterKey = "w2rMY35Qpinuy5W4rzxmr21w0Pnc7B9TexWTg8X0_4G5AzFuPjNo1w=="

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
docker build .\ --tag la-standard-dotnet:0.0.6-s1 -f .\Dockerfile --no-cache

# workflow not loaded
docker run -d -p 5002:80 -e "WEBSITE_HOSTNAME=localhost" -e "WEBSITE_SITE_NAME=helloworkflow" -e "AzureWebJobsStorage=$storageCN" -e "APPINSIGHTS_INSTRUMENTATIONKEY=556bd798-0734-4dae-a0a0-66d4ed23dff8" -e "AZURE_FUNCTIONS_ENVIRONMENT=Development" -e "AzureFunctionsJobHost__extensionBundle__version=[1.*, 2.0.0)" -e "AzureFunctionsJobHost__extensionBundle__id=Microsoft.Azure.Functions.ExtensionBundle.Workflows" la-standard-dotnet:0.0.6-s1

# not ok
docker run -d -p 5002:80 -e "WEBSITE_HOSTNAME=localhost" -e "WEBSITE_SITE_NAME=helloworkflow" -e "AzureWebJobsStorage=$storageCN" -e "APPINSIGHTS_INSTRUMENTATIONKEY=556bd798-0734-4dae-a0a0-66d4ed23dff8" -e "AZURE_FUNCTIONS_ENVIRONMENT=Development" -e "AzureFunctionsJobHost__extensionBundle__version=[1.*, 2.0.0)" -e "AzureFunctionsJobHost__extensionBundle__id=Microsoft.Azure.Functions.ExtensionBundle.Workflows" -e "AzureFunctionsJobHost__extensions__workflow__settings__Runtime.WorkflowOperationDiscoveryHostMode=true" la-standard-dotnet:0.0.6-s1


docker ps

docker logs ebb2ea97ac6c

docker stop ebb2ea97ac6c

docker exec -it ebb2ea97ac6c bash
ls -a -l
```


## V0 - on kubernetes

```
docker tag la-standard-dotnet:0.0.6-s1 massimocrippa/la-standard-dotnet:0.0.6-s1
docker login -u massimocrippa -p somepassword
docker push massimocrippa/la-standard-dotnet:0.0.6-s1
```

```
# create namespace and shared resources
kubectl create namespace logicapps-v0

# deploy all the content of the dev folder
kubectl --namespace 'logicapps-v0' apply --filename manifests/v0/
```

```
kubectl port-forward logicapps-k8s-v0-84bbf64d96-wt4l5 80:80 -n logicapps-v0
kubectl exec -it curl-pod -n logicapps-v0 -- bin/sh
  
# get the masterkey (azure-webjobs-secrets)
$masterKey = "w2rMY35Qpinuy5W4rzxmr21w0Pnc7B9TexWTg8X0_4G5AzFuPjNo1w=="

# get the LogicApps URL
$sdkWebhook = "http://localhost:80/runtime/webhooks/workflow/api/management/workflows/helloworkflow/triggers/manual/listCallbackUrl?api-version=2020-05-01-preview&code=$masterKey"

# get the LogicApps URL
curl --location --request POST $sdkWebhook --header "Content-Length: 0" -v

# workflow URL	(vscode)
$wfURL="http://localhost/api/helloworkflow/triggers/manual/invoke?api-version=2022-05-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=TKWJDgpW1_bRYF0FZqj--MpwcbC1gx2XH61k5tPkIiI"

curl --location --request POST $wfURL --header 'Content-Type: application/json' --header 'x-format: expand'  --data-raw '{"itemId": 1000}' -v

```