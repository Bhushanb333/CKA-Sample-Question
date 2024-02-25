# Question 2 Kube-proxy

You're asked to confirm that kube-proxy is running correctly on all nodes in Namespace `project-hamster`. Perform the following steps:

1. Create a new Pod named `p2-pod` with two containers: `nginx:1.21.3-alpine` and `busybox:1.31`. Ensure that the `busybox` container keeps running for some time.

2. Create a new Service named `p2-service` that exposes the `p2-pod` internally in the cluster on port 3000 -> 80.

3. Find the `kube-proxy` container on all nodes (`cluster1-controlplane1`, `cluster1-node1`, and `cluster1-node2`) and confirm that it's using iptables. Use the `crictl` command for this.

4. Write the iptables rules of all nodes belonging to the created Service `p2-service` into the file `/opt/course/p2/iptables.txt`.

5. Finally, delete the Service and confirm that the iptables rules are removed from all nodes.

<details><summary>Solution:</summary>

Create the Pod:

```bash
k run p2-pod --image=nginx:1.21.3-alpine --dry-run=client -o yaml > p2-pod.yaml
```

Edit the p2-pod.yaml file to add the second container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: p2-pod
  name: p2-pod
  namespace: project-hamster             # add
spec:
  containers:
  - image: nginx:1.21.3-alpine
    name: p2-pod
  - image: busybox:1.31                  # add
    name: c2                             # add
    command: ["sh", "-c", "sleep 1d"]    # add
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Create the Pod:

```bash
k apply -f p2-pod.yaml
```

Create the Service:

```bash
k -n project-hamster expose pod p2-pod --name p2-service --port 3000 --target-port 80

k -n project-hamster get pod,svc,ep
```

Confirm kube-proxy is running and is using iptables
First we get nodes in the cluster:

```bash
k get node
NAME                     STATUS   ROLES           AGE   VERSION
cluster1-controlplane1   Ready    control-plane   98m   v1.28.2
cluster1-node1           Ready    <none>          96m   v1.28.2
cluster1-node2           Ready    <none>          95m   v1.28.2
```

The idea here is to log into every node, find the kube-proxy container and check its logs:

Find the kube-proxy container and check its logs on each node:

```bash
ssh cluster1-controlplane1 crictl ps | grep kube-proxy
ssh cluster1-controlplane1 crictl logs <kube-proxy-container-id>

ssh cluster1-node1 crictl ps | grep kube-proxy
ssh cluster1-node1 crictl logs <kube-proxy-container-id>

ssh cluster1-node2 crictl ps | grep kube-proxy
ssh cluster1-node2 crictl logs <kube-proxy-container-id>
```

Write the iptables rules of all nodes belonging to the p2-service into the file /opt/course/p2/iptables.txt :

```bash
ssh cluster1-controlplane1 iptables-save | grep p2-service >> /opt/course/p2/iptables.txt
ssh cluster1-node1 iptables-save | grep p2-service >> /opt/course/p2/iptables.txt
ssh cluster1-node2 iptables-save | grep p2-service >> /opt/course/p2/iptables.txt
```

Delete the Service:

```bash
k -n project-hamster delete svc p2-service
```

Confirm that the iptables rules are removed from all nodes:

```bash
ssh cluster1-controlplane1 iptables-save | grep p2-service
ssh cluster1-node1 iptables-save | grep p2-service
ssh cluster1-node2 iptables-save | grep p2-service
```

Kubernetes Services are implemented using iptables rules. Whenever a Service is altered, created, deleted, or when the Endpoints of a Service change, the kube-apiserver updates the iptables rules on every node through kube-proxy to reflect the current state.

</details>
