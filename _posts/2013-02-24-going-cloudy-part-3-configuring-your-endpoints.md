---
layout: post
title:  "Going Cloudy Part 3 - Configuring your endpoints"
date: 2013-02-24 20:27:00 +0000
categories: azure cloud migration
---

In my previous post, I outlined the new architecture. You may have noticed how we no longer have urls containing relative paths. This is because all of the components will be hosted on the same web role and I don’t want to have to deal with complicated startup scripts to configure IIS, therefore I want to try and use the Azure-provided methods as much as possible. When deploying multiple sites to a single web role in Azure, there are essentially two choices for setting it up.

* Virtual directories
* Sites

Using virtual directories involves nesting all of the secondary services in the hierarchy of the main web role project (in this case the website). The up side of this is that I could retain my current url structure. The main down side of this is the inheritance of web.config settings. I initially prototyped using this set up and spent a lot of time overriding entries from the main web role config in the web.configs of the web and rest services to prevent missing assembly and conflicting configuration errors.

An example of this is the following snippet from the assemblies section in the web.config of the main website.

{% highlight xml %}
          <add assembly="System.Web.Helpers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
          <add assembly="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35, processorArchitecture=MSIL" />
          <add assembly="System.Web.Abstractions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
          <add assembly="System.Web.Routing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
          <add assembly="System.Web.WebPages, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
          <add assembly="System.Data.Linq, Version=4.0.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />
{% endhighlight %}

Looks harmless enough right? All of these assembly imports will get inherited down to the monitoring site’s child projects, the ASMX web service, and the Site and REST service. The latter 2 are also ASP.NET MVC so it’s not a problem. However, the asmx web service has none of the system.web assemblies in it, causing runtime errors post-deployment.

There were other items that caused me problems but this is the most obvious example. In the end, I decided the virtual directory set up was too brittle.

Using sites has the benefit of keeping all the individual applications on the server separate, so a failure or change in one won’t affect another. Because the applications are separate, we can’t use paths to distinguish which application we want when our request hits the server, so instead we use Host Headers.

The Host Header indicates the url that the user followed in order to get to us. By using subdomains for each service, we can tell from the host header which service the user is attempting to access.

The ServiceDefinition.csdef for the project looks like this.

{% highlight xml %}
  <WebRole name="domain.Monitoring" vmsize="ExtraSmall">
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint3" />
        </Bindings>
      </Site>
      <Site name="Site" physicalDirectory="../../../../../AzPublish/Site-Release">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint1" hostHeader="domain.com" />
          <Binding name="Endpoint2" endpointName="Endpoint2" hostHeader="domain.com" />
        </Bindings>
      </Site>
      <Site name="Gateway" physicalDirectory="../../../../../AzPublish/webservice-Release">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint1" hostHeader="webservice.domain.com" />
          <Binding name="Endpoint2" endpointName="Endpoint2" hostHeader="webservice.domain.com" />
        </Bindings>
      </Site>
      <Site name="RESTService" physicalDirectory="../../../../../AzPublish/Rest-Release">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint1" hostHeader="rest.domain.com" />
          <Binding name="Endpoint2" endpointName="Endpoint2" hostHeader="rest.domain.com" />
        </Bindings>
      </Site>
    </Sites>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="http" port="80" />
      <InputEndpoint name="Endpoint2" protocol="https" port="443" certificate="DomainCertificate" />
      <InputEndpoint name="Endpoint3" protocol="http" port="22202" />
    </Endpoints>
{% endhighlight %}

The Binding elements within each site define the endpoints that this site can be reached on and the hostHeader attribute maps this application to the url the user followed to get to us.

Notice how the domain.Monitoring project has no domain header. This means that this project will reply on endpoint3 (mapped to port 22202) for all traffic. Having it on a separate port means there is no chance of it ever grabbing traffic from the main projects.

You’ll notice that the Monitoring project is the main project in this deployment (I’ll go in to the monitoring project when I cover the traffic manager). The reason for this is that I had some initial trouble getting the host header set up to work with the Azure Traffic Manager and thought doing this may work. That later turned out to be incorrect but I see no harm in keeping the monitoring project as the main one here. Feel free to let me know if there are some caveats to doing this that I am not aware of.

Only the main project in the web role is compiled when publishing, so the ServiceDefinition needs to be pointed to a location containing the compiled project. In order to achieve this, I use file system deployment to publish the subprojects to the AzPublish folder. It’s not ideal and I intend to get all this scripted in Powershell before we go live. When I do that, I will of course publish the scripts on this blog. Also, if there is a better way, please use the comments.
