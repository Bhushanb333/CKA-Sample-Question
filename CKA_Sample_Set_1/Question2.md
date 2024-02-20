# Question 2 | Schedule Pod on Controlplane Nodes**

Create a single Pod of image `httpd:2.4.41-alpine` in Namespace `default`. The Pod should be named `pod1` and the container should be named `pod1-container`. This Pod should only be scheduled on controlplane nodes. Do not add new labels to any nodes.

<details>
<summary>Solution :</summary>

First we find the controlplane node(s) and their taints:

```shell
k get node # find controlplane node

k describe node cluster1-controlplane1 | grep Taint -A1 # get controlplane node taints

k get node cluster1-controlplane1 --show-labels # get controlplane node labels
```

Next we create the Pod template:

```shell
k run pod1 --image=httpd:2.4.41-alpine --dry-run=client -o yaml > 2.yaml
```

Perform the necessary changes manually. Use the Kubernetes docs and search for example for tolerations and nodeSelector to find examples:

```shell
vim 2.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: httpd:2.4.41-alpine
    name: pod1-container                       # change
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:                                 # add
  - effect: NoSchedule                         # add
    key: node-role.kubernetes.io/control-plane # add
  nodeSelector:                                # add
    node-role.kubernetes.io/control-plane: ""  # add
status: {}
```

Important here to add the toleration for running on controlplane nodes, but also the nodeSelector to make sure it only runs on controlplane nodes. If we only specify a toleration the Pod can be scheduled on controlplane or worker nodes.

Now we create it:

```shell
k -f 2.yaml create
```

Let's check if the pod is scheduled:

```shell
k get pod pod1 -o wide
```

Output should be like

```text
NAME   READY   STATUS    RESTARTS   ...    NODE                     NOMINATED NODE
pod1   1/1     Running   0          ...    cluster1-controlplane1   <none>
```
