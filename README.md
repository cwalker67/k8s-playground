# Kubernetes Playground
This project contains a `Vagrantfile` and associated `Ansible` playbook scripts
to provisioning a 3 nodes Kubernetes cluster using `VirtualBox` and `Ubuntu
16.04`.

### Prerequisites
You need the following installed to use this playground.
- `Vagrant`, version 1.9.3 or better. Earlier versions of vagrant do not work
with the Vagrant Ubuntu 16.04 box and network configuration.
- `VirtualBox`, tested with Version 5.1.18 r114002
- Internet access, this playground pulls Vagrant boxes from the Internet as well
as installs Ubuntu application packages from the Internet.

### Bringing Up The cluster
To bring up the cluster, clone this repository to a working directory.

```
git clone http://github.com/davidkbainbridge/k8s-playground
```

Change into the working directory and `vagrant up`

```
cd k8s-playground
vagrant up
```

Vagrant will start four machines. Each machine will have a NAT-ed network
interface, through which it can access the Internet, and a `private-network`
interface in the subnet 172.42.42.0/24. The private network is used for
intra-cluster communication.

The machines created are:

| NAME | IP ADDRESS | ROLE |
| --- | --- | --- |
| k8s-master | 172.42.42.100 | Cluster Master |
| k8s-node-1 | 172.42.42.101 | Cluster Worker |
| k8s-node-2 | 172.42.42.102 | Cluster Worker |
| k8s-node-3 | 172.42.42.103 | Cluster Worker |

As the cluster brought up the cluster master (**k8s-master**) will perform a `kubeadm
init` and the cluster workers will perform a `kubeadmin join`. This cluster is
using a static Kubernetes cluster token.

After the `vagrant up` is complete, the following command and output should be
visible on the cluster master (**k8s-master**).

```
vagrant ssh k8s-master
kubectl -n kube-system get po -o wide

NAME                             READY     STATUS              RESTARTS   AGE       IP            NODE
etcd-k8s1                        1/1       Running             0          10m       172.42.42.1   k8s1
kube-apiserver-k8s1              1/1       Running             1          10m       172.42.42.1   k8s1
kube-controller-manager-k8s1     1/1       Running             0          11m       172.42.42.1   k8s1
kube-discovery-982812725-pv5ib   1/1       Running             0          11m       172.42.42.1   k8s1
kube-dns-2247936740-cucu9        0/3       ContainerCreating   0          10m       <none>        k8s1
kube-proxy-amd64-kt8d6           1/1       Running             0          10m       172.42.42.1   k8s1
kube-proxy-amd64-o73p7           1/1       Running             0          5m        172.42.42.3   k8s3
kube-proxy-amd64-piie9           1/1       Running             0          8m        172.42.42.2   k8s2
kube-scheduler-k8s1              1/1       Running             0          11m       172.42.42.1   k8s1
```

### Starting Networking
Stating the clustering networking is **NOT** automated and must be completed
after the `vagrant up` is complete. A script to start the networking is
installed on the cluster master (**k8s-master**) as `/usr/local/bin/start-weave`.

```
vagrant ssh k8s-master
ubuntu@k8s1:~$ start-weave
clusterrole "weave-net" created
serviceaccount "weave-net" created
clusterrolebinding "weave-net" created
daemonset "weave-net" created
```

After the network is started, assuming `weave-net` is used, the following
command and output should be visible on the master node (**k8s-master**):

```
vagrant ssh k8s-master
$ kubectl -n kube-system get po -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP            NODE
etcd-k8s1                        1/1       Running   0          14m       172.42.42.1   k8s1
kube-apiserver-k8s1              1/1       Running   1          13m       172.42.42.1   k8s1
kube-controller-manager-k8s1     1/1       Running   0          14m       172.42.42.1   k8s1
kube-discovery-982812725-pv5ib   1/1       Running   0          14m       172.42.42.1   k8s1
kube-dns-2247936740-cucu9        3/3       Running   0          14m       10.40.0.1     k8s1
kube-proxy-amd64-kt8d6           1/1       Running   0          13m       172.42.42.1   k8s1
kube-proxy-amd64-o73p7           1/1       Running   0          8m        172.42.42.3   k8s3
kube-proxy-amd64-piie9           1/1       Running   0          11m       172.42.42.2   k8s2
kube-scheduler-k8s1              1/1       Running   0          14m       172.42.42.1   k8s1
weave-net-33rjx                  2/2       Running   0          3m        172.42.42.2   k8s2
weave-net-3z7jj                  2/2       Running   0          3m        172.42.42.1   k8s1
weave-net-uvv48                  2/2       Running   0          3m        172.42.42.3   k8s3
```


### Clean Up
On each vagrant machine is installed a utility as `/usr/local/bin/clean-k8s`.
executing this script as `sudo` will reset the servers back to a point where
you can execute vagrant provisioning.
