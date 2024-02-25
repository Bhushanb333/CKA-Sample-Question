# Question 2 | Create Secret and mount into Pod

Do the following in a new Namespace secret. Create a Pod named secret-pod of image busybox:1.31.1 which should keep running for some time.

There is an existing Secret located at `/opt/course/19/secret1.yaml`, create it in the Namespace secret and mount it readonly into the Pod at `/tmp/secret1`.

Create a new Secret in Namespace secret called secret2 which should contain `user=user1` and `pass=1234`. These entries should be available inside the Pod's container as environment variables `APP_USER` and `APP_PASS`.

Confirm everything is working.

<details><summary>Solution:</summary>

First, we create the Namespace and the requested Secrets in it:

```shell
kubectl create ns secret

cp /opt/course/19/secret1.yaml 19_secret1.yaml

vim 19_secret1.yaml
```

We need to adjust the Namespace for that Secret:

```yaml
# 19_secret1.yaml
apiVersion: v1
data:
  halt: IyEgL2Jpbi9zaAo...
kind: Secret
metadata:
  creationTimestamp: null
  name: secret1
  namespace: secret           # change
```

```shell
kubectl -f 19_secret1.yaml create
```

Next, we create the second Secret:

```shell
kubectl -n secret create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234
```

Now, we create the Pod template:

```shell
kubectl -n secret run secret-pod --image=busybox:1.31.1 -- sh -c "sleep 5d" > 19.yaml

vim 19.yaml
```

Then, make the necessary changes:

```yaml
# 19.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret                       # add
spec:
  containers:
  - args:
    - sh
    - -c
    - sleep 1d
    image: busybox:1.31.1
    name: secret-pod
    resources: {}
    env:                                  # add
    - name: APP_USER                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: user                       # add
    - name: APP_PASS                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: pass                       # add
    volumeMounts:                         # add
    - name: secret1                       # add
      mountPath: /tmp/secret1             # add
      readOnly: true                      # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                # add
  - name: secret1                         # add
    secret:                               # add
      secretName: secret1                 # add
status: {}
```

It might not be necessary in current K8s versions to specify the readOnly: true because it's the default setting anyways.

And execute:

```shell
kubectl -f 19.yaml create
```

Finally, we check if all is correct:

```shell
kubectl -n secret exec secret-pod -- env | grep APP
kubectl -n secret exec secret-pod -- find /tmp/secret1
kubectl -n secret exec secret-pod -- cat /tmp/secret1/halt
```

</details>
