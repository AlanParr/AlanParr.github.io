---
layout: post
title:  "Bicep - Create a storage account and retrieve SAS tokens"
date:   2021-06-10 00:00:00 +0000
tags:
  - Bicep
category: DevOps
---

First thing first, I am really enjoying using Bicep. The intellisense is the real difference between that and something like Terraform, it makes discovery so much easier and creates a real flow to building Bicep files.

The ability to get an ARM JSON file out of the Bicep script is also proving really useful when something isn't working to figure out what I am doing wrong.


I thought I'd just share a script i've been working on which creates a storage account, with a container, and then outputs 2 SAS tokens, one which grants read-only access to the entire account and one that grants upload to only a specific container.


On to the code:

main.bicep
```bicep
param appname string = 'testapp'
param environment string = 'preprod'
param region string = 'ukwest'

targetScope = 'subscription'

//Create the resource group.
resource sa 'Microsoft.Resources/resourceGroups@2021-01-01' = {
  name: 'rg-${appname}-${environment}'
  location: region
}

//Run the storage module, setting scope to the resource group we just created.
module res './storage.bicep' = {
  name: 'resourceDeploy'
  params: {
    appname: appname
    envtype: environment
  }
  scope: resourceGroup(sa.name)
}

//The below both feel a bit dirty manually building the url. 
//Please let me know if there is a better way to do this.

//Create the full url for our account download SAS.
output blobDownloadSAS string = '${res.outputs.blobEndpoint}/?${res.outputs.allBlobDownloadSAS}'

//Create the full url for our container upload SAS.
output myContainerUploadSAS string = '${res.outputs.myContainerBlobEndpoint}?${res.outputs.myContainerUploadSAS}'
```

storage.bicep
```bicep
targetScope = 'resourceGroup'

param appname string
param envtype string

//storage account
resource mainstorage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'sa${appname}${envtype}'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
  properties: {
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
  }
}

//create container
resource mainstoragecontainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-02-01' = {
  name: '${mainstorage.name}/default/mycontainer'
  dependsOn: [
    mainstorage
  ]
}

output blobEndpoint string = 'https://satestapppreprod.blob.${environment().suffixes.storage}'
output myContainerBlobEndpoint string = 'https://satestapppreprod.blob.${environment().suffixes.storage}/mycontainer'

//SAS to download all blobs in account
output allBlobDownloadSAS string = listAccountSAS(mainstorage.name, '2021-04-01', {
  signedProtocol: 'https'
  signedResourceTypes: 'sco'
  signedPermission: 'rl'
  signedServices: 'b'
  signedExpiry: '2022-07-01T00:00:00Z'
}).accountSasToken

//SAS to upload blobs to just the mycontainer container.
output myContainerUploadSAS string = listServiceSAS(mainstorage.name,'2021-04-01', {
  canonicalizedResource: '/blob/${mainstorage.name}/mycontainer'
  signedResource: 'c'
  signedProtocol: 'https'
  signedPermission: 'rwl'
  signedServices: 'b'
  signedExpiry: '2022-07-01T00:00:00Z'
}).serviceSasToken
```

We can run this with the Az CLI:
```
az deployment sub create -f .\main.bicep -l ukwest
```

In the output, we get our SAS tokens:
```json
"outputs": {
      "blobDownloadSAS": {
        "type": "String",
        "value": "https://satestapppreprod.blob.core.windows.net/?sv=2015-04-05&ss=b&srt=sco&sp=rl&se=2022-07-01T00%3A00%3A00.0000000Z&spr=https&sig=REDACTED"
      },
      "myContainerUploadSAS": {
        "type": "String",
        "value": "https://satestapppreprod.blob.core.windows.net/mycontainer?sv=2015-04-05&sr=c&spr=https&se=2022-07-01T00%3A00%3A00.0000000Z&sp=rwl&sig=REDACTED"
      }
    }
```

I tested these in Azure Storage Explorer and got an error when trying to upload with the first SAS token (as expected) but it worked with second (also as expected).

One thing that caused some confusion when creating the SAS configuration in the Bicep file was that it appears the order of the attributes in `signedPermission` matters as I originally had it set to `rlw` and it didn't work, but changing to `rwl` did work.

I figured it out by generating the SAS manually in Azure Storage Explorer and comparing them to find the differences.

Hope the above is useful, will try to post more useful bits I find as I work with Bicep more.