# Question 3 | Update Kubernetes Version and join cluster

Your coworker said node `cluster3-node2` is running an older Kubernetes version and is not even part of the cluster. Update Kubernetes on that node to the exact version that's running on `cluster3-controlplane1`. Then add this node to the cluster. Use `kubeadm` for this.

<details><summary>Solution:</summary>

Upgrade Kubernetes to `cluster3-controlplane1` version
Search in the docs for `kubeadm upgrade`: [Kubeadm Upgrade](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade)

```shell
kubectl get node

NAME                     STATUS   ROLES           AGE   VERSION
cluster3-controlplane1   Ready    control-plane   22h   v1.28.2
cluster3-node1           Ready    <none>          22h   v1.28.2
```

Controlplane node seems to be running Kubernetes 1.28.2. Node cluster3-node2 might not yet be part of the cluster depending on previous tasks.

```shell
ssh cluster3-node2
root@cluster3-node2:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:19:40Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}

root@cluster3-node2:~# kubectl version --short
Client Version: v1.27.4
Kustomize Version: v5.0.1
The connection to the server localhost:8080 was refused - did you specify the right host or port?

root@cluster3-node2:~# kubelet --version
Kubernetes v1.27.4
```

Here kubeadm is already installed in the wanted version, so we don't need to install it. Hence we can run:

```shell
root@cluster3-node2:~# kubeadm upgrade node
couldn\'t create a Kubernetes client from file "/etc/kubernetes/kubelet.conf": failed to load admin kubeconfig: open /etc/kubernetes/kubelet.conf: no such file or directory
To see the stack trace of this error execute with --v=5 or higher
```

This is usually the proper command to upgrade a node. But this error means that this node was never even initialized, so nothing to update here. This will be done later using kubeadm join . For now, we can continue with kubelet and kubectl :

```shell
root@cluster3-node2:~# apt update
...
root@cluster3-node2:~# apt show kubectl -a | grep 1.28
...
Version: 1.28.2-00
Version: 1.28.1-00
Version: 1.28.0-00
root@cluster3-node2:~# apt install kubectl=1.28.2-00 kubelet=1.28.2-00
...
root@cluster3-node2:~# kubelet --version
Kubernetes v1.28.2
...
```

Now we're up to date with kubeadm , kubectl , and kubelet . Restart the kubelet :

```shell
root@cluster3-node2:~# service kubelet restart
...
root@cluster3-node2:~# service kubelet status
Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Fri 2023-09-22 14:37:37 UTC; 2s a>
       Docs: https://kubernetes.io/docs/home/
    Process: 34331 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBEL>
   Main PID: 34331 (code=exited, status=1/FAILURE)

Sep 22 14:37:37 cluster3-node2 systemd[1]: kubelet.service: Main process exited, code=exited, stat>
Sep 22 14:37:37 cluster3-node2 systemd[1]: kubelet.service: Failed with result \'exit-code\'.
```

These errors occur because we still need to run kubeadm join to join the node into the cluster. Let's do this in the next step.

Add cluster3-node2 to cluster
First we log into the controlplane1 and generate a new TLS bootstrap token, also printing out the join command:

```shell
ssh cluster3-controlplane1

root@cluster3-controlplane1:~# kubeadm token create --print-join-command
kubeadm join 192.168.100.31:6443 --token lyl4o0.vbkmv9rdph5qd660 --discovery-token-ca-cert-hash sha256:b0c94ccf935e27306ff24bce4b8f611c621509e80075105b3f25d296a94927ce 

root@cluster3-controlplane1:~# kubeadm token list
TOKEN                     TTL         EXPIRES                ...
lyl4o0.vbkmv9rdph5qd660   23h         2023-09-23T14:38:12Z   ...
n4dkqj.hu52l46jfo4he61e   <forever>   <never>                ...
s7cmex.ty1olulkuljju9am   18h         2023-09-23T09:34:20Z   ...
```

We see the expiration of 23h for our token, we could adjust this by passing the ttl argument.

Next we connect again to cluster3-node2 and simply execute the join command:

```shell
ssh cluster3-node2

root@cluster3-node2:~# kubeadm join 192.168.100.31:6443 --token lyl4o0.vbkmv9rdph5qd660 --discovery-token-ca-cert-hash sha256:b0c94ccf935e27306ff24bce4b8f611c621509e80075105b3f25d296a94927ce 

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
...
```

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```shell
root@cluster3-node2:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Fri 2023-09-22 14:39:57 UTC; 14s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 34695 (kubelet)
      Tasks: 12 (limit: 462)
     Memory: 55.4M
...
```

If you have troubles with kubeadm join you might need to run kubeadm reset.

check the node status:

```shell
k get node

NAME                     STATUS     ROLES           AGE    VERSION
cluster3-controlplane1   Ready      control-plane   102m   v1.28.2
cluster3-node1           Ready      <none>          97m    v1.28.2
cluster3-node2           Ready      <none>          108s   v1.28.2
```

We see cluster3-node2 is now available and up to date.

</details>
