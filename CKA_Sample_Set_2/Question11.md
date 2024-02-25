# Question 11  Get ETCD Details

The cluster admin asked you to find out the following information about etcd running on `cluster2-controlplane1`:

- Server private key location
- Server certificate expiration date
- Is client certificate authentication enabled

Write this information into `/opt/course/p1/etcd-info.txt`.

Finally, you're asked to save an etcd snapshot at `/etc/etcd-snapshot.db` on `cluster2-controlplane1` and display its status.

<details><summary>Solution:</summary>

Find out etcd information:

Let's check the nodes:

```bash
k get node
```

SSH into cluster2-controlplane1 :

```bash
ssh cluster2-controlplane1
```

Check how etcd is set up in this cluster:

```bash
kubectl -n kube-system get pod
```

We see that etcd is running as a Pod. Let's find the location of the etcd manifest file:

`find /etc/kubernetes/manifests/`
Open the etcd manifest file to see its configuration:

`vim /etc/kubernetes/manifests/etcd.yaml`

In the etcd.yaml file, we can find the server private key location and other information:

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.102.11:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt              # server certificate
    - --client-cert-auth=true                                      # enabled
    - --data-dir=/var/lib/etcd
    - ...
    - --key-file=/etc/kubernetes/pki/etcd/server.key               # server private key
    - ...
```

To find the server certificate expiration date, run the following command:

```bash
openssl x509 -noout -text -in /etc/kubernetes/pki/etcd/server.crt | grep Validity -A2
```

Write the information into the requested file:

```txt
# /opt/course/p1/etcd-info.txt
Server private key location: /etc/kubernetes/pki/etcd/server.key
Server certificate expiration date: [expiration date]
Is client certificate authentication enabled: [yes/no]
```

Create etcd snapshot:

Save the etcd snapshot file:

```bash
ETCDCTL_API=3 etcdctl snapshot save /etc/etcd-snapshot.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key
```

Display the status of the etcd snapshot file:

```bash
ETCDCTL_API=3 etcdctl snapshot status /etc/etcd-snapshot.db
The status shows:

Hash: [hash]
Revision: [revision]
Total Keys: [total keys]
Total Size: [total size]
```

</details>
