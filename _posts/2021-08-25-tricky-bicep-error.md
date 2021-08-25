---
layout: post
title:  "Bicep - A root resource type must have segment length one greater than its resource name"
date:   2021-08-25 00:00:00 +0000
tags:
  - Bicep
category: DevOps
---

I was working on a Bicep script that would deploy an Azure SQL database server and a database.

Below is an excerpt of the file with the relevant pieces:

```bicep
resource server 'Microsoft.Sql/servers@2021-02-01-preview' = {
  name: 'server${appname}${envtype}'
  location: resourceGroup().location
  properties:{
    administratorLogin: 'adminuser'
    administratorLoginPassword: 'passwordhere'
  }
}

resource database 'Microsoft.Sql/servers/databases@2021-02-01-preview' = {
  name: 'db${appname}${envtype}'
  location: resourceGroup().location
  sku: {
    name: 'GP_S_Gen5'
    tier: 'GeneralPurpose'
    family: 'Gen5'
    capacity: 1
  }
  dependsOn: [
    server
  ]
}
```

When I ran the script, I got the following baffling error:

```json
{
  "error": {
    "code": "InvalidTemplate",
    "message": "Deployment template validation failed: 'The template resource 'dbcssccptest' for type 'Microsoft.Sql/servers/databases' at line '1' and column '1162' has incorrect segment lengths. A nested resource type must have identical number of segments as its resource name. A root resource type must have segment length one greater than its resource name. Please see https://aka.ms/arm-template/#resources for usage details.'.",
    "additionalInfo": [
      {
        "type": "TemplateViolation",
        "info": {
          "lineNumber": 1,
          "linePosition": 1162,
          "path": "properties.template.resources[1].type"
        }
      }
    ]
  }
}
```

The key piece of information being
```
A nested resource type must have identical number of segments as its resource name. A root resource type must have segment length one greater than its resource name.
```

I followed the link and, frankly, it was no use at all.
After over an hour of experimentation, and googling, I manage to find a Stack Overflow post which unfortunately I didn't save the url for, which gave me the answer.

The reason was staring me in the face the whole time as another thing I was trying to figure out was how to link the database to the server.

As it turns out, the name of the database must be prefixed with the name of the server.

So
```bicep
name: 'db${appname}${envtype}'
```

should be 

```bicep
name: 'server${appname}${envtype}/db${appname}${envtype}'
```

Once I knew the solution, the error message started to make a bit more sense, although a more decisive error message would've been nice.

Hopefully this blog post will save someone else (or future me) from wrestling with the same problem.