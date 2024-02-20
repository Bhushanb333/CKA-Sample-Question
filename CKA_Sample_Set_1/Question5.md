# Question 5 | Kubectl sorting

There are various Pods in all namespaces. Write a command into `/opt/course/5/find_pods.sh` which lists all Pods sorted by their AGE (`metadata.creationTimestamp`).

Write a second command into `/opt/course/5/find_pods_uid.sh` which lists all Pods sorted by field `metadata.uid`. Use kubectl sorting for both commands.

<details>
<summary>Solution :</summary>

A good resource here (and for many other things) is the kubectl-cheat-sheet. You can find it quickly by searching for "cheat sheet" in the Kubernetes docs.

In /opt/course/5/find_pods.sh add

```shell
kubectl get pod -A --sort-by=.metadata.creationTimestamp
```

To execute:

```shell
sh /opt/course/5/find_pods.sh
/opt/course/5/find_pods_uid.sh

kubectl get pod -A --sort-by=.metadata.uid
```

To execute: sh /opt/course/5/find_pods_uid.sh
Make sure to adjust the file permissions to allow execution.

These commands will list all Pods sorted by their AGE or metadata UID respectively, using kubectl sorting.
</details>
