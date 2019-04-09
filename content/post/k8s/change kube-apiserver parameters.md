---
title: "How to Change kube-apiserver Parameters"
date: 2019-04-01T15:15:05+08:00
categories:
- Technology
- k8s
tags:
- k8s
- kube-apiserver
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
(for k8s cluster provisioned with kubeadm)   
SSH to k8s master node, edit file: /etc/kubernetes/manifests/kube-apiserver.json.    
kube-apiserver pod will restart automatically after saving.
