---
title: "Monitor Prometheus Misc"
date: 2019-04-12T08:27:11+08:00
categories:
- technology
- k8s
tags:
- k8s
- prometheus
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
* check Prometheus configuration
```
get prometheus -n ns-monitor monitor-prometheus-operato-prometheus -o yaml
```

* check operator log
```
kubectl logs -n ns-monitor -f deployment/monitor-prometheus-operato-operator
```

* check operator yaml
```
kubectl -n ns-monitor get deployment/monitor-prometheus-operato-operator -o yaml
```

* proxy prometheus
```
kubectl -n ns-monitor port-forward svc/monitor-prometheus-operato-prometheus 9090
```