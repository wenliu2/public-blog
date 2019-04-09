---
title: "Verify DNS Is Working"
date: 2019-04-01T13:55:35+08:00
categories:
- Technology
- k8s
tags:
- k8s
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

*1.* On the Kubernetes master (if you used an HA deployment, select the primary master), set up a test environment by running the following command:kubectl create -f ./busybox.v1.yaml   
```c
  apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: busybox
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: busybox
    template:
      metadata:
        labels:
          app: busybox
      spec:
        containers:
        - name: busybox
          image: busybox:1.28.3
          command:
            - sleep
            - "3600"
          imagePullPolicy: IfNotPresent
        restartPolicy: Always
```

*2.* Verify that the test pod is running by executing the following command:

```
kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
busybox-766c978cd5-wrwgm   1/1     Running   0          5m34s
```

Verify that DNS is working correctly by running the following command:
`kubectl exec -ti busybox-766c978cd5-wrwgm -- nslookup kubernetes.default`

If DNS is working correctly, the command returns a response like the following example: 
```
Server: 10.96.0.10 
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local 
Name: kubernetes.default 
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

Errors such as the following message indicate a problem with DNS: 
```
Server: 10.0.0.10 
Address 1: 10.0.0.10
nslookup: can't resolve 'kubernetes.default'
```

If you encounter a DNS problem, you must correct it.
If the nslookup command fails, execute the following command on all Kubernetes nodes in your cluster (including all masters and worker nodes):
```
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

Then repeat step 3 to verify that the DNS is now working correctly. If DNS problems persist, recreate the core-dns pods on the Kubernetes master by running the following command (in an HA deployment, execute this command on the primary Kubernetes master):
```
kubectl delete pod -n kube-system  -l k8s-app=kube-dns
```

Remember that you must repeat step 3 to verify that the DNS is now working correctly.

For additional troubleshooting tips, see the Debugging DNS Resolution section of the Kubernetes.

When the DNS is working correctly, delete the test environment by running the following command:
`kubectl delete -f ./busybox.v1.yaml`   
This command returns the following response to confirm that the test environment was deleted: pod "busybox" deleted