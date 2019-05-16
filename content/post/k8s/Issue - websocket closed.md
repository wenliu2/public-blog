---
title: "Issue: Websocket closed in K8s"
date: 2019-04-02T15:53:24+08:00
categories:
- Technology
- k8s
tags:
- k8s
- nginx-ingress-controller
- websocket
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
Links: https://medium.com/@del680202/%E6%88%91%E5%9C%A8nginx-ingress-reload%E4%B8%8B%E6%8E%A1%E7%9A%84%E5%9D%91-6c0dba53838e


# Change 2 configuration of nginx controller.
---
### 1. enable dynamic config reload flag
This flag will enable the nginx configuration reloading only happens when ingress changed. Other configuration change(like create new pods) will not affect nginx. Without nginx reload,  the websocket connection will not be shutdown.

execute below command:
```
kubectl -n ingress-nginx edit daemonset/nginx-ingress-controller -o yaml
```
make sure the args like below:

```
...
containers:
      - args:
        - /nginx-ingress-controller
        - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
        - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
        - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
        - --annotations-prefix=nginx.ingress.kubernetes.io
        - --enable-dynamic-configurations
...
```

### 2. Change worker shutdown timeout parameter to 3600s (it's 10s by default)

If ingress is changed (add/delete/update ingress), the nginx is reloaded and websoket will be shutdown. `worker-shutdown-timeout` parameter controls how long nginx keeps it before the existing web socket is shutdown.  

```
kubectl -n ingress-nginx edit configmap nginx-configuration

apiVersion: v1
data:
  worker-shutdown-timeout: 3600s
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":null,"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app":"ingress-nginx"},"name":"nginx-configuration","namespace":"ingress-nginx"}}
  creationTimestamp: 2018-11-13T01:22:42Z
  labels:
    app: ingress-nginx
  name: nginx-configuration
  namespace: ingress-nginx
  resourceVersion: "32327432"
  selfLink: /api/v1/namespaces/ingress-nginx/configmaps/nginx-configuration
  uid: 9e62354f-e6e2-11e8-a1cc-74dbd1802aba

```
