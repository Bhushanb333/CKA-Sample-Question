# Question 8 | Get Controlplane Information

Ssh into the controlplane node with `ssh cluster1-controlplane1`. Check how the controlplane components `kubelet`, `kube-apiserver`, `kube-scheduler`, `kube-controller-manager` and `etcd` are started/installed on the controlplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node.

Write your findings into file `/opt/course/8/controlplane-components.txt`. The file should be structured like:
/opt/course/8/controlplane-components.txt

kubelet: [TYPE]
kube-apiserver: [TYPE]
kube-scheduler: [TYPE]
kube-controller-manager: [TYPE]
etcd: [TYPE]
dns: [TYPE] [NAME]

Choices of `[TYPE]` are: `not-installed`, `process`, `static-pod`, `pod`

<details>
<summary>Solution :</summary>

We could start by finding processes of the requested components, especially the `kubelet` at first:

```shell
ssh cluster1-controlplane1
root@cluster1-controlplane1:~# ps aux | grep kubelet # shows kubelet process
```

We can see which components are controlled via systemd looking at `/etc/systemd/system` directory:

```shell
root@cluster1-controlplane1:~# find /etc/systemd/system/ | grep kube

/etc/systemd/system/kubelet.service.d
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
/etc/systemd/system/multi-user.target.wants/kubelet.service

root@cluster1-controlplane1:~# find /etc/systemd/system/ | grep etcd
```

This shows `kubelet` is controlled via systemd, but no other service named `kube` nor `etcd`. It seems that this cluster has been setup using `kubeadm`, so we check in the default manifests directory:

```shell
root@cluster1-controlplane1:~# find /etc/kubernetes/manifests/

/etc/kubernetes/manifests/
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/etcd.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/manifests/kube-scheduler.yaml
```

The `kubelet` could also have a different manifests directory specified via parameter `--pod-manifest-path` in its systemd startup config

This means the main 4 controlplane services are setup as static Pods. Actually, let's check all Pods running on in the `kube-system` Namespace on the controlplane node:

```shell
root@cluster1-controlplane1:~# kubectl -n kube-system get pod -o wide | grep controlplane1

coredns-5644d7b6d9-c4f68 1/1 Running ... cluster1-controlplane1 coredns-5644d7b6d9-t84sc 1/1 Running ... cluster1-controlplane1 etcd-cluster1-controlplane1 1/1 Running ... cluster1-controlplane1 kube-apiserver-cluster1-controlplane1 1/1 Running ... cluster1-controlplane1 kube-controller-manager-cluster1-controlplane1 1/1 Running ... cluster1-controlplane1 kube-proxy-q955p 1/1 Running ... cluster1-controlplane1 kube-scheduler-cluster1-controlplane1 1/1 Running ... cluster1-controlplane1 weave-net-mwj47 2/2 Running ... cluster1-controlplane1
```

There we see the 5 static pods, with -cluster1-controlplane1 as suffix.

We also see that the dns application seems to be coredns, but how is it controlled?

```shell
root@cluster1-controlplane1$ kubectl -n kube-system get ds

NAME         DESIRED   CURRENT   ...   NODE SELECTOR            AGE
kube-proxy   3         3         ...   kubernetes.io/os=linux   155m
weave-net    3         3         ...   <none>                   155m
```

```shell
root@cluster1-controlplane1$ kubectl -n kube-system get deploy

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           155m
```

Seems like coredns is controlled via a Deployment. We combine our findings in the requested file:

```txt
vi /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
```
</details>
