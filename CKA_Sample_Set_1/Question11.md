# Question 11 | DaemonSet on all Nodes

Use context: `kubectl config use-context k8s-c1-H`

Use Namespace `project-tiger` for the following. Create a DaemonSet named `ds-important` with image `httpd:2.4-alpine` and labels `id=ds-important` and `uuid=18426a0b-5f59-4e10-923f-c0e078e82462`. The Pods it creates should request 10 millicore CPU and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, including control planes.

<details>
<summary>Solution:</summary>

As of now, we aren't able to create a DaemonSet directly using `kubectl`, so we create a Deployment and make the necessary changes.

```shell
k -n project-tiger create deployment --image=httpd:2.4-alpine ds-important $do > 11.yaml
```

Then we adjust the YAML to:

```yaml
# 11.yaml
apiVersion: apps/v1
kind: DaemonSet                                     # change from Deployment to DaemonSet
metadata:
  creationTimestamp: null
  labels:                                           # add
    id: ds-important                                # add
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462      # add
  name: ds-important
  namespace: project-tiger                          # important
spec:
  #replicas: 1                                      # remove
  selector:
    matchLabels:
      id: ds-important                              # add
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462    # add
  #strategy: {}                                     # remove
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: ds-important                            # add
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462  # add
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: ds-important
        resources:
          requests:                                 # add
            cpu: 10m                                # add
            memory: 10Mi                            # add
      tolerations:                                  # add
      - effect: NoSchedule                          # add
        key: node-role.kubernetes.io/control-plane  # add
#status: {}                                         # remove
```

It was requested that the DaemonSet runs on all nodes, so we need to specify the toleration for this.

Let's confirm:

```shell
k -f 11.yaml create
k -n project-tiger get ds

NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-important   3         3         3       3            3           <none>          8s

k -n project-tiger get pod -l id=ds-important -o wide

NAME                      READY   STATUS          NODE
ds-important-6pvgm        1/1     Running   ...   cluster1-node1
ds-important-lh5ts        1/1     Running   ...   cluster1-controlplane1
ds-important-qhjcq        1/1     Running   ...   cluster1-node2
```

</details>
