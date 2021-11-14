---
layout: post
title:  "Bicep - Create Azure Service Bus with topic and filtered subscription."
date:   2021-11-14 00:00:00 +0000
tags:
  - Bicep
category: DevOps
---

It's been a while since I've done anything with Service Bus so to get myself back in to it and indulge my current enjoyment of Bicep, I thought i'd start with creating the service bus, along with a topic, and a filtered subscription, all in Bicep.

To start with, the main.bicep which is virtually the same as last time you saw it:

```bicep
param appname string = 'testapp'
param environment string = 'staging'
param region string = 'ukwest'

targetScope = 'subscription'

//Create the resource group.
resource sa 'Microsoft.Resources/resourceGroups@2021-01-01' = {
  name: 'rg-${appname}-${environment}'
  location: region
}

//Run the storage module, setting scope to the resource group we just created.
module res './servicebus.bicep' = {
  name: 'resourceDeploy'
  params: {
    appname: appname
    envtype: environment
  }
  scope: resourceGroup(sa.name)
}
```

I've just changed it to call `servicebus.bicep` instead of `storage.bicep`.

The servicebus.bicep consists of the following:

```bicep
targetScope = 'resourceGroup'

param appname string
param envtype string

resource servicebus 'Microsoft.ServiceBus/namespaces@2021-06-01-preview' = {
  name: 'sb${appname}${envtype}'
  location: resourceGroup().location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}
```

Start with defining our parameters that we will take in from the main.bicep and creating the Service Bus namespace. This was pretty straight forward, using the [docs](https://docs.microsoft.com/en-us/azure/templates/microsoft.servicebus/namespaces?tabs=bicep) to get the parameter names.

```bicep
resource orders 'Microsoft.ServiceBus/namespaces/topics@2021-06-01-preview' = {
  name: 'orders'
  parent: servicebus
  properties: {
    defaultMessageTimeToLive: 'P6M' //ISO 8601
    status: 'Active'
  }
}
```

Next we create the topic. I did get stuck here for a bit as it wasn't obvious what the value of parent should be. Eventually, I realised it was the resource object for the `servicebus` itself, which is handy to know as it looks like `parent` is used in various places in Bicep.

The `defaultMessageTimeToLive` is using ISO 8601 duration format which is a fairly simple format to understand, brief explanation is [here](https://www.digi.com/resources/documentation/digidocs/90001437-13/reference/r_iso_8601_duration_format.htm#:~:text=Represents%20a%20duration%20of%20three%20years%2C%20six%20months%2C,duration%20formatupdated%20on%2020%20Sep%202019%2010%3A14%20AM) and there is a calculator [here](https://www.345tool.com/time-util/time-duration-calculator).


```bicep
resource amazonpushlistingrule 'Microsoft.ServiceBus/namespaces/topics/subscriptions/rules@2021-06-01-preview' = {
  name: 'operationtype'
  parent: orderreceived
  properties: {
    filterType: 'CorrelationFilter'
    correlationFilter: {
      properties: {
        'operationtype':'orderreceived'
      }
    }
  }
}
```
Next up is the subscription. Subscriptions can have filters that limit the messages that go in to that subscription from the topic, in this case I'm looking for a property called `operationtype` to have value of `orderreceived`. Only messages where this is true will appear in this subscription.


```bicep
resource catchall 'Microsoft.ServiceBus/namespaces/topics/subscriptions@2021-06-01-preview' = {
  name: 'catchall'
  parent: orders
  properties: {
    deadLetteringOnMessageExpiration: true
    defaultMessageTimeToLive: 'P6M'
    lockDuration: 'PT5M'
    maxDeliveryCount: 20
    status: 'Active'
  }
}
```

Last, but not least, I create a catchall subscription with no filter, so if any messages don't end up where I expect them to, I can look at them in this subscription to see what was wrong.

Then I just deploy using the below Az Cli command
```
az deployment sub create -f .\main.bicep -l ukwest
```


For easy copy/paste, the full content of servicebus.bicep is below:

```bicep
targetScope = 'resourceGroup'

param appname string
param envtype string

resource servicebus 'Microsoft.ServiceBus/namespaces@2021-06-01-preview' = {
  name: 'sb${appname}${envtype}'
  location: resourceGroup().location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}

resource orders 'Microsoft.ServiceBus/namespaces/topics@2021-06-01-preview' = {
  name: 'orders'
  parent: servicebus
  properties: {
    defaultMessageTimeToLive: 'P6M' //ISO 8601
    status: 'Active'
  }
}

resource orderreceived 'Microsoft.ServiceBus/namespaces/topics/subscriptions@2021-06-01-preview' = {
  name: 'orderreceived'
  parent: orders
  properties: {
    deadLetteringOnMessageExpiration: true
    defaultMessageTimeToLive: 'P6M'
    lockDuration: 'PT5M'
    maxDeliveryCount: 20
    status: 'Active'
  }
}

resource amazonpushlistingrule 'Microsoft.ServiceBus/namespaces/topics/subscriptions/rules@2021-06-01-preview' = {
  name: 'operationtype'
  parent: orderreceived
  properties: {
    filterType: 'CorrelationFilter'
    correlationFilter: {
      properties: {
        'operationtype':'orderreceived'
      }
    }
  }
}

resource catchall 'Microsoft.ServiceBus/namespaces/topics/subscriptions@2021-06-01-preview' = {
  name: 'catchall'
  parent: orders
  properties: {
    deadLetteringOnMessageExpiration: true
    defaultMessageTimeToLive: 'P6M'
    lockDuration: 'PT5M'
    maxDeliveryCount: 20
    status: 'Active'
  }
}
```