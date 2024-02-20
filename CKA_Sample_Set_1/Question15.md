# Question 15 | Cluster Event Logging

Write a command into `/opt/course/15/cluster_events.sh` which shows the latest events in the whole cluster, ordered by time (`metadata.creationTimestamp`). Use `kubectl` for it.

Now delete the kube-proxy Pod running on node `cluster2-node1` and write the events this caused into `/opt/course/15/pod_kill.log`.

Finally, kill the `containerd` container of the kube-proxy Pod on node `cluster2-node1` and write the events into `/opt/course/15/container_kill.log`.

Do you notice differences in the events both actions caused?

<details>
<summary>Solution:</summary>

First, create the `/opt/course/15/cluster_events.sh` file and add the command to display the latest events in the cluster:

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp > /opt/course/15/cluster_events.sh
```

Next, delete the kube-proxy Pod running on node cluster2-node1 :

```bash
kubectl -n kube-system get pod -o wide | grep proxy # find pod running on cluster2-node1

kubectl -n kube-system delete pod kube-proxy-z64cg
```

Check the events to see the impact of deleting the Pod:
Write the events the killing caused into /opt/course/15/pod_kill.log:

```bash
sh /opt/course/15/cluster_events.sh > /opt/course/15/pod_kill.log
```

Finally we will try to provoke events by killing the container belonging to the container of the kube-proxy Pod:

Now, SSH into cluster2-node1 and kill the container belonging to the kube-proxy Pod:

```bash
ssh cluster2-node1

crictl ps | grep kube-proxy
crictl rm <container-id-1>
crictl ps | grep kube-proxy
```

We killed the main container , but also noticed that a new container was directly created. Thanks Kubernetes!

Check the events again to see the impact of killing the container:

```bash
sh /opt/course/15/cluster_events.sh > /opt/course/15/container_kill.log
```

By comparing the events in /opt/course/15/pod_kill.log and /opt/course/15/container_kill.log , you will notice differences in the events caused by deleting the entire Pod versus killing only the container. For example, when deleting the Pod, there may be events related to the DaemonSet recreating the Pod. On the other hand, when killing the container, only events related to recreating the container will be present, resulting in fewer events.

</details>
