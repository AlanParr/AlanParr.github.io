---
layout: post
title:  "Bicep - Outputting connection string"
date:   2021-08-27 00:00:00 +0000
tags:
  - Bicep
category: DevOps
---

One of the nice touches of Bicep are some of the helpers it provides you, specifically the `environment()` helper. This provides information such as Azure endpoints to save you hard-coding it.

In my recent journey in to deploying SQL Database, I wanted to output the connection string. Naturally, I did what I do with ARM which is build up the connection string myself, as below:

```bicep
output MyConnectionString string = 'Server=tcp:${server.name}.database.windows.net,1433;Initial Catalog=db${appname}${envtype};Persist Security Info=False;User ID=username;Password=password;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;'
```

When I did this, I received this very helpful warning in VS Code, where I have the Bicep extension installed.
![Bicep VS Code warning when hardcoding environment url.](\assets\img\bicep-helper-warning.png)

I checked out the [provided url](https://aka.ms/bicep/linter/no-hardcoded-env-urls) which with a bit of effort lead me to, for my particular use case, `environment().suffixes.sqlServerHostname` which basically gives me the value I was hard-coding but means that if the endpoint changes in the future, Bicep will handle it for me, which is nice.

The end result was the below, which is gloriously lacking hard-coded values I don't control.
```bicep
output MyConnectionString string = 'Server=tcp:${server.name}${environment().suffixes.sqlServerHostname},1433;Initial Catalog=db${appname}${envtype};Persist Security Info=False;User ID=username;Password=password;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;'
```