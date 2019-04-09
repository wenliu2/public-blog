---
title: "Install k8s"
date: 2019-04-01T09:26:08+08:00
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
## Prerequisite
---
1. Install Ubuntu16.04
2. Install docker


## Install kubeadm
---
```
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

## Creating a single master cluster with kubeadm
---
#### Initializing your master
Run as root(use calico as network addon):
`kubeadm init --pod-network-cidr=192.168.0.0/16`
The output looks like:
```
[init] Using Kubernetes version: vX.Y.Z
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubeadm-master localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 31.501735 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-X.Y" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm-master" as an annotation
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubeadm-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: <token>
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

If you are the root, run below:
`export KUBECONFIG=/etc/kubernetes/admin.conf`


#### Install calico network add-on
```
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

#### Check all run well
```
kubectl get pods -n kube-system

NAME                                       READY   STATUS    RESTARTS   AGE
calico-node-lq7tx                          2/2     Running   0          24s
coredns-fb8b8dccf-8v8ww                    1/1     Running   0          2m50s
coredns-fb8b8dccf-f6pwr                    1/1     Running   0          2m50s
etcd-r1-k8s01-2664920                      1/1     Running   1          35m
kube-apiserver-r1-k8s01-2664920            1/1     Running   1          35m
kube-controller-manager-r1-k8s01-2664920   1/1     Running   1          35m
kube-proxy-wbxjq                           1/1     Running   1          36m
kube-scheduler-r1-k8s01-2664920            1/1     Running   1          35m
```

If the coredns pods showing error, fix it with below steps:   
1. open and config the configmap of coredns `kubectl edit cm coredns -n kube-system`   
2. delete 'loop', save and exit   
3. restart coredns pods by:
`
kubectl get pods -n kube-system -oname |grep coredns |xargs kubectl delete -n kube-system
`   
4. wait the coredns restart and check the output. It may take 2~3 minutes.


## Joining your nodes
---
The nodes are where your workloads (containers and pods, etc) run. To add new nodes to your cluster do the following for each machine:

* SSH to the machine
* Become root (e.g. sudo su -)
* Run the command that was output by kubeadm init. For example:
```kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>```   

If you do not have the token, you can get it by running the following command on the master node:

`kubeadm token list`
The output is similar to this:
```
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```                                                                                           
By default, tokens expire after 24 hours. If you are joining a node to the cluster after the current token has expired, you can create a new token by running the following command on the master node:

`kubeadm token create`
The output is similar to this:
```
5didvk.d09sbcov8ph2amjw
```
If you don’t have the value of --discovery-token-ca-cert-hash, you can get it by running the following command chain on the master node:
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```
The output is similar to this:
```
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```
Note: To specify an IPv6 tuple for <master-ip>:<master-port>, IPv6 address must be enclosed in square brackets, for example: [fd00::101]:2073.
The output should look something like:
```
[preflight] Running pre-flight checks

... (log output of join workflow) ...
```

Node join complete:   

* Certificate signing request sent to master and response received.
* Kubelet informed of new secure connection details.

Run `kubectl get nodes` on the master to see this machine join.
A few seconds later, you should notice this node in the output from kubectl get nodes when run on the master.