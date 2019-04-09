---
title: "Service Mesh vs API Gateway"
date: 2019-03-21T16:22:11+08:00
categories:
- technology
- service
tags:
- service mesh
- api gateway
- architecture
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

# 1. What's service mesh?
---
service mesh ...


# 2. What's API Gateway
---
#### API Gateway features
The key objective of using API Gateway is to expose your (micro) services as managed APIs. So, the API or Edge services that we develop at the API Gateway layer serves a specific business functionality.

* API/Edge services call the downstream (composite and atomic) microservices and contain the business logic that creates compositions/mashups of multiple downstream services.   
* API/Edge services also need to call the downstream services on the resilient manner and apply various stability patterns such as Circuit Breakers, Timeouts, Load Balancing/Failover. Therefore most of the API Gateway solutions out there have these features built in.
* API Gateways also come inbuilt support for service discovery, analytics(observability: Metrics, monitoring, distributed logging, distributed tracing.) and security.
* API Gateways closely work with several other components of the API Management ecosystem, such as API marketplace/store, API publishing portal.


# 3. Key difference
---
The key differentiators between API Gateways and service mesh is that API Gateways is a key part of exposing API/Edge services where service mesh is merely an inter-service communication infrastructure which doesn’t have any business notion of your solution.