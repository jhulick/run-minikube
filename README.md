# Run minikube

## Run minikube in macOS Mojave (10.14.2)

### install minikube

Please follow this [tutorial](https://kubernetes.io/docs/tutorials/hello-minikube/)

```
$ minikube version
```
  > minikube version: v0.30.0

### install vm driver

I used [hyperkit](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver)

### 1. Prepare docker images dependended and Start minikube

Run `docker pull` the following images on `Host` terminal:

```
	k8s.gcr.io/coredns:1.2.2
	k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
	k8s.gcr.io/kube-proxy-amd64:v1.10.0
	k8s.gcr.io/kube-apiserver-amd64:v1.10.0
	k8s.gcr.io/kube-controller-manager-amd64:v1.10.0
	k8s.gcr.io/kube-scheduler-amd64:v1.10.0
	k8s.gcr.io/etcd-amd64:3.1.12
	k8s.gcr.io/kube-addon-manager:v8.6
	k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8
	k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8
	k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8
	k8s.gcr.io/pause-amd64:3.1
	gcr.io/k8s-minikube/storage-provisioner:v1.8.1
```

Save the images as `k8s.zip`:
```
$ docker save \
	k8s.gcr.io/coredns:1.2.2 \
	k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0 \
	k8s.gcr.io/kube-proxy-amd64:v1.10.0 \
	k8s.gcr.io/kube-apiserver-amd64:v1.10.0 \
	k8s.gcr.io/kube-controller-manager-amd64:v1.10.0 \
	k8s.gcr.io/kube-scheduler-amd64:v1.10.0 \
	k8s.gcr.io/etcd-amd64:3.1.12 \
	k8s.gcr.io/kube-addon-manager:v8.6 \
	k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8 \
	k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8 \
	k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8 \
	k8s.gcr.io/pause-amd64:3.1 \
	gcr.io/k8s-minikube/storage-provisioner:v1.8.1 \
	| gzip -c > k8s.zip
```

After then, run the following commands:
```
$ sudo rm /var/db/dhcpd_leases # before you remove it, try 'cat /var/db/dhcpd_leases'
$ minikube stop
$ minikube delete
$ minikube start --vm-driver=hyperkit -v=9
$ export NO_PROXY=$no_proxy,$(minikube ip)
$ minikube start --vm-driver=hyperkit -v=9
```
If you see the waiting for ssh... the installing cfssl (with brew) and removing ~/.minikube, starting over.

### 2. When terminal prints `Starting cluster components...`
We will start a simple HTTP server in `Host` by:

```
$ python -m SimpleHTTPServer
```

to serve `k8s.zip` which will be downloaded in `vm`.

### 3. Login the `vm`

```
$ ssh docker@$(minikube ip) # using password "tcuser"
```

The `/mnt/vda1` has enough space to save `k8s.zip`, run the following commands:

NOTE: In my container the IP address is 172.17.1.192. Now look at the routing table:

```
root@e77f6a1b3740:/# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.42.1     0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
```

So the IP Address of the docker host 172.17.42.1 is set as the default route and is accessible from your container.

```
$ cd /mnt/vda1
$ sudo wget 'http://host_ip:8000/k8s.zip' # (in my case host ip was 192.168.64.3 but you will have to connect to 192.168.64.1)
$ docker load < k8s.zip
$ sudo rm k8s.zip
$ docker images
```

* If meet the error: `starting cluster:  kubeadm init error`, run following command in `vm`:
```
$ sudo kubeadm reset
$ sudo /usr/bin/kubeadm init --config /var/lib/kubeadm.yaml --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests --ignore-preflight-errors=DirAvailable--data-minikube --ignore-preflight-errors=Port-10250 --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-etcd.yaml --ignore-preflight-errors=Swap --ignore-preflight-errors=CRI

```

<!-- ```
$ sudo /usr/bin/kubeadm alpha phase addon kube-dns
``` -->

* Check the log in `vm`
```
$ sudo journalctl -xeu kubelet
```

### 4. Check k8s node

```
$ kubectl version
```

will display log in termina:

```
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:55:54Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

and

```
$ kubectl get nodes
```

will display log in termina:

```
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    17m       v1.10.0
```

and
```
$ minikube dashboard
```

Have fun with k8s!
