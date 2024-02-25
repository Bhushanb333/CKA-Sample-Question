# Question 4 | Create a Static Pod and Service

Create a Static Pod named `my-static-pod` in Namespace `default` on `cluster3-controlplane1`. It should be of image `nginx:1.16-alpine` and have resource requests for 10m CPU and 20Mi memory.

Then create a NodePort Service named `static-pod-service` which exposes that static Pod on port 80 and check if it has Endpoints and if it's reachable through the `cluster3-controlplane1` internal IP address. You can connect to the internal node IPs from your main terminal.

<details><summary>Solution:</summary>

```shell
ssh cluster3-controlplane1
cd /etc/kubernetes/manifests/

kubectl run my-static-pod \
    --image=nginx:1.16-alpine \
    -o yaml --dry-run=client > my-static-pod.yaml
```

Then edit the my-static-pod.yaml to add the requested resource requests:

```yaml
# /etc/kubernetes/manifests/my-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-static-pod
  name: my-static-pod
spec:
  containers:
  - image: nginx:1.16-alpine
    name: my-static-pod
    resources:
      requests:
        cpu: 10m
        memory: 20Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

And make sure it's running:

```shell
kubectl get pod -A | grep my-static
```

Now we expose that static Pod:

```shell
kubectl expose pod my-static-pod-cluster3-controlplane1 \
  --name static-pod-service \
  --type=NodePort \
  --port 80
```

This would generate a Service like:

```yaml
# kubectl expose pod my-static-pod-cluster3-controlplane1 --name static-pod-service --type=NodePort --port 80
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: my-static-pod
  name: static-pod-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: my-static-pod
  type: NodePort
status:
  loadBalancer: {}
```

Then run and test:

```shell
kubectl get svc,ep -l run=my-static-pod
```

</details>
