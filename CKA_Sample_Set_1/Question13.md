# Question 13 | Multi Containers and Pod shared Volume

Use context: `kubectl config use-context k8s-c1-H`

Create a Pod named `multi-container-playground` in Namespace `default` with three containers, named `c1`, `c2`, and `c3`. There should be a volume attached to that Pod and mounted into every container, but the volume shouldn't be persisted or shared with other Pods.

Container `c1` should be of image `nginx:1.17.6-alpine` and have the name of the node where its Pod is running available as environment variable `MY_NODE_NAME`.

Container `c2` should be of image `busybox:1.31.1` and write the output of the `date` command every second in the shared volume into a file called `date.log`. You can use `while true; do date >> /your/vol/path/date.log; sleep 1; done` for this.

Container `c3` should be of image `busybox:1.31.1` and constantly send the content of file `date.log` from the shared volume to stdout. You can use `tail -f /your/vol/path/date.log` for this.

Check the logs of container `c3` to confirm correct setup.

<details>
<summary>Solution:</summary>

First, we create the Pod template:

```bash
kubectl run multi-container-playground --image=nginx:1.17.6-alpine --dry-run=client -o yaml > 13.yaml
```

Open the 13.yaml file and add the other containers and the commands they should execute:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container-playground
  name: multi-container-playground
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: c1                                                                      # change
    resources: {}
    env:                                                                          # add
    - name: MY_NODE_NAME                                                          # add
      valueFrom:                                                                  # add
        fieldRef:                                                                 # add
          fieldPath: spec.nodeName                                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c2                                                                      # add
    command: ["sh", "-c", "while true; do date >> /vol/date.log; sleep 1; done"]  # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c3                                                                      # add
    command: ["sh", "-c", "tail -f /vol/date.log"]                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                                                        # add
    - name: vol                                                                   # add
      emptyDir: {}                                                                # add
status: {}
```

Create the Pod using the updated configuration file:

```bash
kubectl apply -f 13.yaml
```

To confirm that the setup is correct, follow these steps:

Check the status of the Pod:

```bash
kubectl get pod multi-container-playground
```

Verify that container c1 has the node name as an environment variable:

```bash
kubectl exec multi-container-playground -c c1 -- env | grep MY
```

Check the logs of container c3 to see the output of date.log :

```bash
kubectl logs multi-container-playground -c c3
```

</details>
