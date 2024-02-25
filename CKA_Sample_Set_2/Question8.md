# Question 8 | Etcd Snapshot Save and Restore

Make a backup of etcd running on `cluster3-controlplane1` and save it on the controlplane node at `/tmp/etcd-backup.db`.

Then create any kind of Pod in the cluster.

Finally, restore the backup, confirm the cluster is still working, and that the created Pod is no longer with us.

<details><summary>Solution:</summary>

### Etcd Backup

First, we log into the controlplane and try to create a snapshot of etcd:

```shell
ssh cluster3-controlplane1
ETCDCTL_API=3 etcdctl -h #get man page for syntax
```

If ca.crt, key and crt file path not give use following

```shell
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
```

We use the authentication information and pass it to etcdctl:

```shell
root@cluster3-controlplane1:~# ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

Snapshot saved at /tmp/etcd-backup.db
```

NOTE: Dont use snapshot status because it can alter the snapshot file and render it invalid

Etcd restore
Now create a Pod in the cluster and wait for it to be running:

```shell
root@cluster3-controlplane1:~# kubectl run test --image=nginx
pod/test created

root@cluster3-controlplane1:~# kubectl get pod -l run=test -w

NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          60s
```

Next we stop all controlplane components:

```shell
root@cluster3-controlplane1:~# cd /etc/kubernetes/manifests/
root@cluster3-controlplane1:/etc/kubernetes/manifests# mv * ..
root@cluster3-controlplane1:/etc/kubernetes/manifests# watch crictl ps
```

Now we restore the snapshot into a specific directory:

```shell
root@cluster3-controlplane1:~# ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
--data-dir /var/lib/etcd-backup \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

root@cluster3-controlplane1:~# vim /etc/kubernetes/etcd.yaml
```

```yaml
# /etc/kubernetes/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
...
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup                # change
      type: DirectoryOrCreate
    name: etcd-data
status: {}
```

Now we move all controlplane yaml again into the manifest directory. Give it some time (up to several minutes) for etcd to restart and for the api-server to be reachable again:

```shell
root@cluster3-controlplane1:/etc/kubernetes/manifests# mv ../*.yaml .

root@cluster3-controlplane1:/etc/kubernetes/manifests# watch crictl ps
```

Then we check again for the Pod:

```shell
root@cluster3-controlplane1:~# kubectl get pod -l run=test
```

No resources found in default namespace.
Awesome, backup and restore worked as our pod is gone.

</details>
