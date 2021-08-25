---
layout: post
title:  "Terraform to Bicep - Part 1"
date:   2021-06-02 00:00:00 +0000
tags:
  - Terraform
  - Bicep
category: DevOps
---

In what will hopefully be a multi-part series of posts, I'll go over Terraform vs Bicep and cover translating a Terraform file to Bicep and cover the challenges I experience doing this and my thoughts as I go.

## What is Terraform?
[Terraform](https://www.terraform.io/) is a product from HashiCorp that provides a common way to build infrastructure and manipulate systems. It uses HashiCorp Common Language (HCL) and has a provider model that allows you to write scripts against many services, such as cloud providers (Azure, AWS, GCP) but also services like Akamai, VMware VSphere, Kubernetes, GitHub, LaunchDarkly, and [many more](https://registry.terraform.io/browse/providers).

## What is Bicep?
[Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) is a new language from Microsoft which sits on top of Azure Resource Manager (ARM) and provides a DSL that is much easier to author than ARM's traditional JSON files.

## Why Bicep?
I've been using Terraform quite a lot in the last year and have really enjoyed using it. Prior to that I authored the ARM JSON files manually, which are not the easiest things to write. They take a long time to write and are tricky things to debug.
While I do like Terraform and it mostly keeps up with ARM, there are 2 things that prompted me to look in to Bicep (beyond the fact it is new and shiny).

1. Terraform needs you to maintain a state file, which it uses to record the previous state of the resources it deployed which it can then check against next time it is run. Its not a big deal but when using Terraform in CI/CD environments, its another thing you need to take in to account and find somewhere to keep it. I generally use Azure Blob Storage to store it, which necessitates an account and corporate security requires a key vault to store the credentials for the storage account, so it just adds a bit of extra work.
2. Currently, Terraform uses an older version of the SAS API which means that under certain circumstances, it produces SAS tokens that aren't usable, such as when trying to configure Blob export of logs from an AppService.

So that is why I wanted to look in to Bicep.

I won't cover installing Terraform or Bicep as this is covered elsewhere in their respective docs.

## Syntax comparison
Syntactically, Terraform and Bicep aren't that different.

The below snippets both create a resource group using passed in parameters `appname` and `environment`

### Terraform
```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-${var.appname}-${var.environment}"
  location = var.primaryregion
}
```

### Bicep
```bicep
targetScope = 'subscription'

resource sa 'Microsoft.Resources/resourceGroups@2021-01-01' = {
  name: 'rg-${appname}-${environment}'
  location: region
}
```

The differences comes when you need to deploy a resource group then deploy resources in to that group.

In Terraform, you just keep creating resources and reference the group you just created.

```hcl
resource "azurerm_application_insights" "main" {
  name                = "appi-${var.appname}-${var.environment}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  application_type = "web"
}
```

However, in Bicep, each file has a scope which is one of `managementGroup`,`resourceGroup`,`subscription`, or `tenant`. As a result, you can't create resources for a resource group in the same file you create the group.

You must break the additional resources out in to a module, as below:

```bicep
module res './resources.bicep' = {
  name: 'resourceDeploy'
  params: {
    appname: appname
    environment: environment
    region: region
  }
  scope: resourceGroup(sa.name)
}
```

This references another file called resources.bicep. Note how it sets the scope of that module to the resourceGroup we just created.


## First Impressions
I like that there is no state file in Bicep.

While I initially was disappointed at having to break out the resources in to a separate module, the idea is growing on me as it encourages modularity. You can do modules in Terraform but it is easy to start with a simple file and end up with a single very long file. Having modules there from the start keeps the idea in the developer's consciousness.

The addition of the API version intrigues me, seems like this may come in useful in the future.


I still slightly prefer the Terraform syntax, for some uses it is more terse than Bicep.

## Next
Assuming I get around to writing part 2, I intend to go through creating resources in more detail.

For now, i've put the working script in a [GitHub repository](https://github.com/AlanParr/terraform-vs-bicep).

