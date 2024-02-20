# Question 7 | Node and Pod Resource Usage

The metrics-server has been installed in the cluster. Your college would like to know the kubectl commands to:

- Show Nodes resource usage
- Show Pods and their containers resource usage

Please write the commands into `/opt/course/7/node.sh` and `/opt/course/7/pod.sh`.

<details>
<summary>Solution :</summary>

For Node resource usage, use the following command:

```shell
kubectl top node > /opt/course/7/node.sh
```

For Pods and their containers resource usage, use the following command:

```shell
kubectl top pod --containers=true > /opt/course/7/pod.sh
```

Make sure to adjust the file permissions to allow execution.
</details>
