# Question 3 | Scale down StatefulSet**

There are two Pods named `o3db-*` in Namespace `project-c13`. C13 management asked you to scale the Pods down to one replica to save resources.

<details>
<summary>Solution :</summary>

If we check the Pods we see two replicas:

```shell
k -n project-c13 get pod | grep o3db
o3db-0                                  1/1     Running   0          52s
o3db-1                                  1/1     Running   0          42s
```

From their name it looks like these are managed by a StatefulSet. But if we're not sure we could also check for the most common resources which manage Pods:

```shell
k -n project-c13 get deploy,ds,sts | grep o3db
statefulset.apps/o3db   2/2     2m56s
```

Confirmed, we have to work with a StatefulSet. To find this out we could also look at the Pod labels:

```shell
k -n project-c13 get pod --show-labels | grep o3db
o3db-0                                  1/1     Running   0          3m29s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-0
o3db-1                                  1/1     Running   0          3m19s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-1
```

To fulfill the task we simply run:

```shell
k -n project-c13 scale sts o3db --replicas 1
statefulset.apps/o3db scaled
```

```shell
k -n project-c13 get sts o3db
NAME   READY   AGE
o3db   1/1     4m39s
```

C13 Management is happy again.
</details>
