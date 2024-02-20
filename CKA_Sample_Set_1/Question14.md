# Question 14 | Find out Cluster Information

You're asked to find out the following information about the cluster `k8s-c1-H`:

How many controlplane nodes are available?
How many worker nodes are available?
What is the Service CIDR?
Which Networking (or CNI Plugin) is configured and where is its config file?
Which suffix will static pods have that run on cluster1-node1?

Write your answers into file /opt/course/14/cluster-info, structured like this:
/opt/course/14/cluster-info
1: [ANSWER]
2: [ANSWER]
3: [ANSWER]
4: [ANSWER]
5: [ANSWER]

<details>
<summary>Solution:</summary>

How many controlplane and worker nodes are available?

```bash
kubectl get node
```

We see one controlplane and two workers.

What is the Service CIDR?

```bash
ssh cluster1-controlplane1
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep range
```

Which Networking (or CNI Plugin) is configured and where is its config file?

```bash
find /etc/cni/net.d/
cat /etc/cni/net.d/10-weave.conflist
```

By default, the kubelet looks into /etc/cni/net.d to discover the CNI plugins. This will be the same on every controlplane and worker nodes.

Which suffix will static pods have that run on cluster1-node1? The suffix is the node hostname with a leading hyphen. It used to be -static in earlier Kubernetes versions.

Result: The resulting /opt/course/14/cluster-info could look like:

```bash
cat /opt/course/14/cluster-info
1: 1
2: 2
3: 10.96.0.0/12
4: Weave, /etc/cni/net.d/10-weave.conflist
5: -cluster1-node1
```

</details>
