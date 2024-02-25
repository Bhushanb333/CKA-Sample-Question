# Question 1 | Fix Kubelet

There seems to be an issue with the kubelet not running on `cluster3-node1`. Fix it and confirm that the cluster has node `cluster3-node1` available in Ready state afterwards. You should be able to schedule a Pod on `cluster3-node1` afterwards.

Write the reason for the issue into `/opt/course/18/reason.txt`.

<details>
<summary>Solution:</summary>

The procedure for tasks like these should be to check if the kubelet is running. If not, start it, then check its logs and correct any errors if there are any.

It's always helpful to check if other clusters already have some of the components defined and running, so you can copy and use existing config files. Though in this case, it might not be necessary.

Check node status:

```shell
kubectl get node

NAME                     STATUS     ROLES           AGE   VERSION
cluster3-controlplane1   Ready      control-plane   14d   v1.28.2
cluster3-node1           NotReady   <none>          14d   v1.28.2
```

First, we check if the kubelet is running:

```shell
ssh cluster3-node1
root@cluster3-node1:~# ps aux | grep kubelet
root     29294  0.0  0.2  14856  1016 pts/0    S+   11:30   0:00 grep --color=auto kubelet
```

Nope, so we check if it's configured using systemd as a service:

```shell
root@cluster3-node1:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: inactive (dead) since Sun 2019-12-08 11:30:06 UTC; 50min 52s ago
...
```

Yes, it's configured as a service with the config at /etc/systemd/system/kubelet.service.d/10-kubeadm.conf , but we see it's inactive. Let's try to start it:

```shell
root@cluster3-node1:~# service kubelet start
root@cluster3-node1:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since Thu 2020-04-30 22:03:10 UTC; 3s ago
     Docs: https://kubernetes.io/docs/home/
  Process: 5989 ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=203/EXEC)
 Main PID: 5989 (code=exited, status=203/EXEC)
 Apr 30 22:03:10 cluster3-node1 systemd[5989]: kubelet.service: Failed at step EXEC spawning /usr/local/bin/kubelet: No such file or directory
Apr 30 22:03:10 cluster3-node1 systemd[1]: kubelet.service: Main process exited, code=exited, status=203/EXEC
Apr 30 22:03:10 cluster3-node1 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

We see it's trying to execute /usr/local/bin/kubelet with some parameters defined in its service config file. A good way to find errors and get more logs is to run the command manually (usually also with its parameters).
 ...

```shell
root@cluster3-node1:~# /usr/local/bin/kubelet
bash: /usr/local/bin/kubelet: No such file or directory

root@cluster3-node1:~# whereis kubelet
kubelet: /usr/bin/kubelet
```

Another way would be to see the extended logging of a service like using 

```shell
journalctl -u kubelet.
```

Well, there we have it, wrong path specified. Correct the path in file /etc/systemd/system/kubelet.service.d/10-kubeadm.conf and run:

```shell
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf # fix

systemctl daemon-reload && systemctl restart kubelet

systemctl status kubelet  # should now show running
```

Also the node should be available for the api server, give it a bit of time though:

```shell
k get node

NAME                     STATUS   ROLES           AGE   VERSION
cluster3-controlplane1   Ready    control-plane   14d   v1.28.2
cluster3-node1           Ready    <none>          14d   v1.28.2
```

Finally we write the reason into the file:

```shell
echo "wrong path to kubelet binary specified in service config" >> /opt/course/18/reason.txt
```

</details>
