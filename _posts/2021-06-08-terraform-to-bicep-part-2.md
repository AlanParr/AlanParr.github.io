---
layout: post
title:  "Terraform to Bicep - Part 2"
date:   2021-06-08 00:00:00 +0000
tags:
  - Terraform
  - Bicep
category: DevOps
---

Following [Part 1](\posts\2021-06-02-terraform-to-bicep-part-1), thought i'd write part 2 before I forgot/lost interest in doing so.

In this part, I am just going to show a few examples of Terraform resources and the Bicep equivalents, including how I worked out what the Bicep equivalent was.

For all of my editing, I am using VS Code with the [HashiCorp Terraform](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform) and [Bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep) extensions.


## Storage Account

### Terraform
```
resource "azurerm_storage_account" "main" {
  name                      = "sa${var.appname}${var.environment}"
  resource_group_name       = azurerm_resource_group.main.name
  location                  = azurerm_resource_group.main.location
  account_kind              = "StorageV2"
  account_tier              = "Standard"
  account_replication_type  = "LRS"
  enable_https_traffic_only = true
  min_tls_version           = "TLS1_2"
}
```

### Bicep
```
resource mainstorage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'sa${appname}${environment}'
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
```

We start our resource with the word `resource` in both. In Bicep, we then put in the name for the resource, this is what other resources will use to reference it. After that, we need to put in the path to the resource type.
One of the really lovely things about the Bicep extension is that it gives you full intellisense for this:

![Bicep resource intellisense](..\\assets\img\bicep2-resource-intellisense.png)
If you don't know what the exact name of the resource is, you can start typing what you want and it gets you fairly close.

Once you hit enter on the resource type, it will offer you a choice of all the API versions

![Bicep resource intellisense versions](..\\assets\img\bicep2-resource-intellisense-versions.png)

This is one feature that I suspect may be a big tick in Bicep's box over time as the Azure API changes. HashiCorp are pretty good at keeping up, but recently I hit an issue where I needed to create a SAS and the version of the API being used by Terraform is not the current and the SAS is not valid for certain uses.

Full disclosure, I haven't actually figured out how to generate a SAS in Bicep yet so it may not have helped me in this case, but assuming these options are kept up to date, it theoretically provides a bit more flexibility.

The downside of having the version in there is that you now have to update it, whereas with Terraform, they take care of that.

So pros and cons to both approaches.

Let's go line by line through the Bicep code.

```
name: 'sa${appname}${environment}'
```
This generates the name of the account, no major difference between this and Terraform except you don't need to prefix the variable with `var.`, which is neither here nor there really.

```
location: resourceGroup().location
```
Here we just take the resource group that we are scoped to, whatever that may be, and set our location to match. We could of course make it different if we needed to, but this provides a nice default.

```
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
```
Here we just set the kind of the account and the SKU. A couple of extra lines than Terraform but again, doesn't really make any odds.

```
properties: {
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
  }
```
One thing I quickly picked up is that for most resources, only properties that are mandatory or are going to be set 99% of the time are present on the base resource.

Lesser used properties hang off of the properties object. Again, full intellisense is provided here which is really helpful from a learning perspective.

![Bicep resource intellisense properties](..\\assets\img\bicep2-resource-intellisense-properties.png)

And that is it for our admittedly very simple storage account example.

There really isn't much difference between the Terraform and Bicep versions, but I am liking the superior intellisense with Bicep and as I already stated, I think the ability to set your API version could be a good differentiator in the future but at the same time it is another thing to keep up to date. I'm not sure what side of the fence it will fall on yet.

The scoping that I wasn't sure about initially has a nice benefit of not having to set the resource group on every resource, which is nice.