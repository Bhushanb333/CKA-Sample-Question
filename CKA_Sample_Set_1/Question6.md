# Question 6 | Storage, PV, PVC, Pod volume

Create a new PersistentVolume named `safari-pv`. It should have a capacity of 2Gi, accessMode ReadWriteOnce, `hostPath` `/Volumes/Data`, and no `storageClassName` defined.

Next, create a new PersistentVolumeClaim in Namespace `project-tiger` named `safari-pvc`. It should request 2Gi storage, accessMode ReadWriteOnce, and should not define a `storageClassName`. The PVC should be bound to the PV correctly.

Finally, create a new Deployment `safari` in Namespace `project-tiger` which mounts that volume at `/tmp/safari-data`. The Pods of that Deployment should be of image `httpd:2.4.41-alpine`.

<details>
<summary>Solution :</summary>

Create the PersistentVolume (`safari-pv`):

```yaml
# 6_pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: safari-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/Volumes/Data"
```

Then create it:

```shell
k -f 6_pv.yaml create
```

Next, create the PersistentVolumeClaim (safari-pvc ):

# 6_pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
Then create it:

k -f 6_pvc.yaml create
Check that both the PV and PVC have the status Bound :

k -n project-tiger get pv,pvc
Finally, create the Deployment safari and mount the volume:

```yaml
# 6_dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: safari
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: safari
    spec:
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: safari-pvc
      containers:
        - image: httpd:2.4.41-alpine
          name: container
          volumeMounts:
            - name: data
              mountPath: /tmp/safari-data
```

Then create it:

```shell
k -f 6_dep.yaml create
```

Confirm that the volume is mounted correctly:

```shell
k -n project-tiger describe pod safari-<POD_ID> | grep -A2 Mounts
```

Make sure to replace <POD_ID> with the actual Pod ID.
</details>
