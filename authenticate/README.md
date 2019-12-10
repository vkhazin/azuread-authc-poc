## Known constraints

As stated by Microsoft (https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-desktop-acquire-token):

* Users who need to do MFA won't be able to sign-in (as there is no interaction).
* The Username/Password flow isn't compatible with Conditional Access and multi-factor authentication: As a consequence, if your app runs in an Azure AD tenant where the tenant admin requires multi-factor authentication, you can't use this flow. Many organizations do that.
* It works only for Work and school accounts (**not MSA**).

## Azure Setup

### Configure appication

* Proceed to Azure AD from Azure Portal.
* Register a new application.
* Go to Authentication page and switch **Treat application as a public client** to **Yes** under Advanced settings (it takes about 1 minute for this setting to take effect).
* Go to API permissions and click **Grant admin consent** button.

### Obtain ClientId

![ClientId](readme-img/clientid.png)

## Deployment

### Create resource group and storage account

Replace `<resource_group>`, `<storage_name>` (and `location`, if necessary) with your values: 

    az group create --name <resource_group> --location northeurope
    az storage account create --name <storage_name> --location northeurope --resource-group <resource_group> --sku Standard_LRS
    
### Create functions

Replace `<funcNameAuthenticate>` and `<funcNameHelloWorld>` with your values:

    az functionapp create --resource-group <resource_group> --consumption-plan-location northeurope \
    --name <funcNameAuthenticate> --storage-account <storage_name> --runtime dotnet

    az functionapp create --resource-group <resource_group> --consumption-plan-location northeurope \
    --name <funcNameHelloWorld> --storage-account <storage_name> --runtime dotnet

### Settings

Settings are required for `authenticate` function only:

* Proceed to Function App from Azure Portal
* Select `<funcNameAuthenticate>`
* Click **Configuration** under Configured features on the Overview tab
* Use **New Application Setting** button to add `ClientId` and `DirectoryName` settings

### Deploy

Execute from each function folders:

```
func azure functionapp publish <funcNameAuthenticate>
```

```
func azure functionapp publish <funcNameHelloWorld>
```

## Running Local

### local.settings.json

Create `local.settings.json` files in the function folders.
For `authenticate` function settings file should include ClientId and DirectoryName:

    {
      "IsEncrypted": false,
      "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "ClientId": "00000000-aaaa-bbbb-cccc-dddddddddddd",
        "DirectoryName": "zzzzzz.onmicrosoft.com"
      }
    }
    
For `helloWorld` function:

    {
      "IsEncrypted": false,
      "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet"
      }
    }

### Run

    cd authenticate
    func start --build

To run `helloWorld` at the same time use different port:
    
    cd ../helloWorld
    func start --build -p 7072

## Sample funcs

authenticate: https://azureavcauth.azurewebsites.net/api/authenticate?code=kfz4wEkaSkQvKGFnik5rC1B5VJZ3QDxzrjtfZzitLIhi2uBbIMiiIw==

helloWorld: https://azureavchello.azurewebsites.net/api/helloworld?code=20kDE3lBP6cB4Hr1TruohTzxKRrwo5JGM3aEOdoEj5Mg3uJniRPmGA==

Sample credentials:

```
{
   "username": "chris@alexeybusyginhotmail.onmicrosoft.com",
   "password": "Tupa2441Tupa2441"
}
```
