---
layout: post
title:  "Going Cloudy Part 6 - Monitoring and Load Balancing"
date: 2013-04-28 21:08:00 +0000
categories: azure cloud migraton
---

## The Monitoring Project
When using the traffic manager, it needs an endpoint on your service to hit in order to determine whether or not it is responding. This endpoint must be open (no authentication) and must be a path on your service. As you may have noticed, in the ServiceDefinition, I instructed my Site, Web Service and REST Services to only respond on port 80 or 443 to a specific host header, one which the traffic manager cannot provide as it will be accessing the instance directly, in other words via it’s cloudapp.net address.

The simple way to solve this would be to make one of the services also respond on an endpoint without a Host Header and give it an unauthenticated ActionResult somewehere that the Traffic Manager could access.

Now I’m no security expert, but I do my best, and I didn’t want any of my core services hosting an unauthenticated endpoint and I want to make sure that they are only accessible by their public urls. Therefore, I created a project that is just for monitoring the health of my services. Initially, this will just service the traffic manager. In the long term, it’s a convenient place to put any generic monitoring functionality.

In order to service the Traffic Manager, it just has a GET action on the Home controller which does a quick database connectivity check and returns a 200 if everything is okay. Before go live, this will be extended to check all the main services are responding.

## Using the Azure Traffic Manager
As previously mentioned, I need to have service instances in both the EU and the US. I didn’t want users to have to decide which one they went to by going to eu.domain.com or us.domain.com, that’s just a bad user experience in my book.

The Azure Traffic Manager provides you with a load balancer that you can use between Cloud Services in the same or different regions. I am using it in the performance mode, which routes the user to the nearest (and presumably fastest) service to them.

The Traffic Manager uses the aforementioned Monitoring project to determine if the service is unavailable. It regularly hits the health endpoint and if it does not receive a 200 within 5 seconds, it considers the instance to be down. When the instance is determined to be down, on the next DNS refresh, the records will be updated to point to the next service on the list. There will still be some down time while all this happens, but it will most likely only be a couple of minutes.
