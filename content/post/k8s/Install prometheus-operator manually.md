---
title: "Install prometheus-operator Manually"
date: 2019-04-02T13:40:34+08:00
categories:
- Technology
- k8s
tags:
- k8s
- prometheus
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

* A k8s is ready, but it is behind corp firewall and has no internet access.   
* Can't install prometheus-operator with helm automatically.   
* This guide explains how to install prometheus-operator in a k8s cluster without internet access.   

*1.* On the desktop with internet access, install helm.

*2.* Fetch chart source
```
mkdir $PWD/charts
cd charts
helm fetch --untar stable/prometheus-operator
```
This will fetch chart files of prometheus-operator and save them into prometheus-operator directory in current working dir.

*3.* Generate yaml with template command .  
```
helm template --name monitor --namespace ns-monitor prometheus-operator > prometheus-operator.yaml
```

*4.* Manually change images in yaml file.

  * Create repository: ecr.vip.ebayc3.com/k8smonitor
  * Create robot account and grant RW permission of k8smonitor repo
  * Docker login with robot account just created
  * Run `base64 $HOME/.docker/config.json`, copy the output base64 string
  * Create a secret yaml file .  
    
    ```
    apiVersion: v1
    kind: Secret
    metadata:
      name: wenliu2ecrkey
      namespace: ns-monitor
    data:
      .dockerconfigjson: "PASTE the base64 string in the previous step here."
    type: kubernetes.io/dockerconfigjson
    ```
  * Run below commands in Desktop console, these steps are pull docker image, change its tag and upload to local registry.  

    ```
    docker pull quay.io/prometheus/node-exporter:v0.17.0 && docker pull kiwigrid/k8s-sidecar:0.0.13 && docker pull grafana/grafana:6.0.2 && docker pull k8s.gcr.io/kube-state-metrics:v1.5.0 && docker pull quay.io/coreos/prometheus-operator:v0.29.0


    docker tag quay.io/prometheus/node-exporter:v0.17.0 ecr.vip.ebayc3.com/k8smonitor/node-exporter:v0.17.0
    docker tag kiwigrid/k8s-sidecar:0.0.13 ecr.vip.ebayc3.com/k8smonitor/k8s-sidecar:0.0.13
    docker tag grafana/grafana:6.0.2 ecr.vip.ebayc3.com/k8smonitor/grafana:6.0.2
    docker tag k8s.gcr.io/kube-state-metrics:v1.5.0 ecr.vip.ebayc3.com/k8smonitor/kube-state-metrics:v1.5.0
    docker tag quay.io/coreos/prometheus-operator:v0.29.0 ecr.vip.ebayc3.com/k8smonitor/prometheus-operator:v0.29.0



    docker push ecr.vip.ebayc3.com/k8smonitor/node-exporter:v0.17.0
    docker push ecr.vip.ebayc3.com/k8smonitor/k8s-sidecar:0.0.13
    docker push ecr.vip.ebayc3.com/k8smonitor/grafana:6.0.2
    docker push ecr.vip.ebayc3.com/k8smonitor/kube-state-metrics:v1.5.0
    docker push ecr.vip.ebayc3.com/k8smonitor/prometheus-operator:v0.29.0
    ```

  * Change prometheus-operator.yaml, to replace external image with image in local registry.
  * Change prometheus-operator.yaml, specify to use "wenliu2ecrkey" to pull images.

    ```
    spec: 
      ...

      imagePullSecrets:
        - name: wenliu2ecrkey
      
      containers:
        ...
      ...
    ```
  * Create a namespace `ns-monitor` by running `kubectl apply -f ns-monitor.yaml `

    ```
    apiVersion: v1
    kind: Namespace
    metadata:
      name: ns-monitor
    ```
  * Run `kubectl apply -f prometheus-operator.yaml -n ns-monitor`
  * Some Services are created under kube-system namespace, for these service, we should run them again

    ```
    kubectl apply -f po-kube-system.yaml -n kube-system

    ==== content of po-kube-system.yaml ===
    ---
    # Source: prometheus-operator/templates/exporters/core-dns/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: monitor-prometheus-operato-coredns
      labels:
        app: prometheus-operator-coredns
        jobLabel: coredns
        
        chart: prometheus-operator-5.0.3
        release: "monitor"
        heritage: "Tiller"
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 9153
          protocol: TCP
          targetPort: 9153
      selector:
        
        k8s-app: kube-dns
    ---
    # Source: prometheus-operator/templates/exporters/kube-controller-manager/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: monitor-prometheus-operato-kube-controller-manager
      labels:
        app: prometheus-operator-kube-controller-manager
        jobLabel: kube-controller-manager
        
        chart: prometheus-operator-5.0.3
        release: "monitor"
        heritage: "Tiller"
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 10252
          protocol: TCP
          targetPort: 10252
      selector:
        
        component: kube-controller-manager
      type: ClusterIP
    ---
    # Source: prometheus-operator/templates/exporters/kube-etcd/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: monitor-prometheus-operato-kube-etcd
      labels:
        app: prometheus-operator-kube-etcd
        jobLabel: kube-etcd
        
        chart: prometheus-operator-5.0.3
        release: "monitor"
        heritage: "Tiller"
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 2379
          protocol: TCP
          targetPort: 2379
      selector:
          
        component: etcd
      type: ClusterIP
    ---
    # Source: prometheus-operator/templates/exporters/kube-scheduler/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: monitor-prometheus-operato-kube-scheduler
      labels:
        app: prometheus-operator-kube-scheduler
        jobLabel: kube-scheduler
        
        chart: prometheus-operator-5.0.3
        release: "monitor"
        heritage: "Tiller"
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 10251
          protocol: TCP
          targetPort: 10251
      selector:
          
        component: kube-scheduler
      type: ClusterIP
    ```  

  * Test, run below command and access localhost:3000, login with admin/prom-operator.

    ```
    kubectl port-forward pod/monitor-grafana-54bf574bfc-zmr6x 3000 3000 -n ns-monitor
    ```
