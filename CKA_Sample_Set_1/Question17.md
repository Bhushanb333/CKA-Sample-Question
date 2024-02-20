# Question 17 | Find Container of Pod and check info

In Namespace `project-tiger`, create a Pod named `tigers-reunite` of image `httpd:2.4.41-alpine` with labels `pod=container` and `container=pod`. Find out on which node the Pod is scheduled. SSH into that node and find the containerd container belonging to that Pod.

Using the `crictl` command:

Write the ID of the container and the `info.runtimeType` into `/opt/course/17/pod-container.txt`.

Write the logs of the container into `/opt/course/17/pod-container.log`.

<details>
<summary>Solution:</summary>

First, create the Pod:

```bash
kubectl -n project-tiger run tigers-reunite \
  --image=httpd:2.4.41-alpine \
  --labels "pod=container,container=pod"
```

Next, find out the node it's scheduled on:

```bash
kubectl -n project-tiger get pod -o wide

# or fancy:
kubectl -n project-tiger get pod tigers-reunite -o jsonpath="{.spec.nodeName}"
```

Then, SSH into that node and check the container info:

```bash
ssh cluster1-node2

crictl ps | grep tigers-reunite

crictl inspect <container-id> | grep runtimeType
```

Fill the requested file on the main terminal:

```bash
echo "<container-id> <runtime-type>" > /opt/course/17/pod-container.txt
```

Finally, write the container logs in the second file:

```bash
ssh cluster1-node2 'crictl logs <container-id>' &> /opt/course/17/pod-container.log
```

The &> in the above command redirects both the standard output and standard error.

You could also simply run crictl logs on the node and copy the content manually if it's not a lot.

</details>
