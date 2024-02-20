# Question 12 | Deployment on all Nodes

Use Namespace `project-tiger` for the following. Create a Deployment named `deploy-important` with label `id=very-important` (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named `container1` with image `nginx:1.17.6-alpine` and the second one named `container2` with image `google/pause`.

There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: `cluster1-node1` and `cluster1-node2`. Because the Deployment has three replicas, the result should be that on both nodes one Pod is running. The third Pod won't be scheduled unless a new worker node will be added.

In a way, we simulate the behavior of a DaemonSet here, but using a Deployment and a fixed number of replicas.

<details>
<summary>Solution:</summary>

There are two possible ways to achieve this: using `podAntiAffinity` and using `topologySpreadConstraints`.

PodAntiAffinity :-
The idea here is that we create an "Inter-pod anti-affinity" which allows us to specify that a Pod should only be scheduled on a node where another Pod of a specific label (in this case, the same label) is not already running.

Let's begin by creating the Deployment template:

```shell
kubectl -n project-tiger create deployment --image=nginx:1.17.6-alpine deploy-important --dry-run=client -o yaml > 12.yaml
```

Then, modify the YAML file as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: google/pause             # add
        name: container2                # add
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
status: {}
```

This YAML file sets up podAntiAffinity to ensure that Pods with the same label are not scheduled on the same node.

Apply and Let's run it:

```shell
k -f 12.yaml create
```

Then we check the Deployment status where it shows 2/3 ready count:

```shell
k -n project-tiger get deploy -l id=very-important

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deploy-important   2/3     3            2           2m35s
```

And running the following we see one Pod on each worker node and one not scheduled.

```shell
k -n project-tiger get pod -o wide -l id=very-important

NAME                                READY   STATUS    ...   NODE             
deploy-important-58db9db6fc-9ljpw   2/2     Running   ...   cluster1-node1
deploy-important-58db9db6fc-lnxdb   0/2     Pending   ...   <none>          
deploy-important-58db9db6fc-p2rz8   2/2     Running   ...   cluster1-node2
```

If we kubectl describe the Pod deploy-important-58db9db6fc-lnxdb it will show us the reason for not scheduling is our implemented podAntiAffinity ruling:

</details>
