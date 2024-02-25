# Question 13 Update Service, Pod

Perform the following steps:

1. Create a Pod named `check-ip` in Namespace `default` using the image `httpd:2.4.41-alpine`. Expose it on port 80 as a ClusterIP Service named `check-ip-service`. Remember/output the IP of that Service.

2. Change the Service CIDR to `11.96.0.0/12` for the cluster.

3. Create a second Service named `check-ip-service2` pointing to the same Pod to check if your settings took effect. Finally, check if the IP of the first Service has changed.

<details><summary>Solution:</summary>

Create the Pod and expose it as a Service:

```bash
k run check-ip --image=httpd:2.4.41-alpine
k expose pod check-ip --name check-ip-service --port 80
```

Get the IP of the Service:

```bash
k get svc check-ip-service -o=jsonpath='{.spec.clusterIP}'
```

Change the Service CIDR to 11.96.0.0/12 :

```bash
ssh cluster2-controlplane1
vim /etc/kubernetes/manifests/kube-apiserver.yaml
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Change the `--service-cluster-ip-range` flag to 11.96.0.0/12 in both files.

Wait for the kube-apiserver and kube-controller-manager to restart.

Create the second Service:

```bash
k expose pod check-ip --name check-ip-service2 --port 80
```

Check the IPs of both Services:

```bash
k get svc check-ip-service -o=jsonpath='{.spec.clusterIP}'
k get svc check-ip-service2 -o=jsonpath='{.spec.clusterIP}'
```

Confirm that the IP of the first Service has changed.

Remember to undo any changes made to the cluster configuration after verifying the results.

</details>
