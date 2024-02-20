# Question 4 | Pod Ready if Service is reachable

Do the following in Namespace default. Create a single Pod named `ready-if-service-ready` of image `nginx:1.16.1-alpine`. Configure a LivenessProbe which simply executes command `true`. Also configure a ReadinessProbe which checks if the URL `http://service-am-i-ready:80` is reachable using the command `wget -T2 -O- http://service-am-i-ready:80`. Start the Pod and confirm it isn't ready because of the ReadinessProbe.

Create a second Pod named `am-i-ready` of image `nginx:1.16.1-alpine` with label `id: cross-server-ready`. The already existing Service `service-am-i-ready` should now have that second Pod as an endpoint.

Now the first Pod should be in a ready state, confirm that.

<details>
<summary>Solution :</summary>

It's a bit of an anti-pattern for one Pod to check another Pod for being ready using probes, hence the normally available `readinessProbe.httpGet` doesn't work for absolute remote URLs. Still, the workaround requested in this task should show how probes and Pod<->Service communication work.

First, we create the first Pod:

```shell
k run ready-if-service-ready --image=nginx:1.16.1-alpine --dry-run=client -o yaml > 4_pod1.yaml
```

open file `vim 4_pod1.yaml`
Next, perform the necessary additions manually:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ready-if-service-ready
  name: ready-if-service-ready
spec:
  containers:
  - image: nginx:1.16.1-alpine
    name: ready-if-service-ready
    resources: {}
    livenessProbe:                                      # add from here
      exec:
        command:
        - 'true'
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - 'wget -T2 -O- http://service-am-i-ready:80'   # to here
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then, create the Pod:

```shell
k -f 4_pod1.yaml create
```

And confirm it's in a non-ready state:

```shell
k get pod ready-if-service-ready
NAME                     READY   STATUS    RESTARTS   AGE
ready-if-service-ready   0/1     Running   0          7s
```

We can also check the reason for this using describe:

```shell
k describe pod ready-if-service-ready
 ...
  Warning  Unhealthy  18s   kubelet, cluster1-node1  Readiness probe failed: Connecting to service-am-i-ready:80 (10.109.194.234:80)
wget: download timed out
```

Now we create the second Pod:

```shell
k run am-i-ready --image=nginx:1.16.1-alpine --labels="id=cross-server-ready"
The already existing Service service-am-i-ready should now have an Endpoint:

k describe svc service-am-i-ready
k get ep # also possible
```

This will result in our first Pod being ready, just give it a minute for the Readiness probe to check again:

```shell
k get pod ready-if-service-ready
NAME                     READY   STATUS    RESTARTS   AGE
ready-if-service-ready   1/1     Running   0          53s
```
</details>
