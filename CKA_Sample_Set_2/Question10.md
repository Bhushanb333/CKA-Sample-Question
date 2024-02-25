# Question 10 | Curl Manually Contact API**

There is an existing ServiceAccount `secret-reader` in Namespace `project-hamster`. Create a Pod of image `curlimages/curl:7.65.3` named `tmp-api-contact` which uses this ServiceAccount. Make sure the container keeps running.

Exec into the Pod and use curl to access the Kubernetes Api of that cluster manually, listing all available secrets. You can ignore insecure https connection. Write the command(s) for this into file `/opt/course/e4/list-secrets.sh`.

<details><summary>Solution:</summary>

Refer to the [Kubernetes documentation](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod) for understanding how the Kubernetes API works and how to manually connect to it using `curl`.

First, create the Pod:

```bash
k run tmp-api-contact \
  --image=curlimages/curl:7.65.3 $do \
  --command > e2.yaml -- sh -c 'sleep 1d'
```

Edit the e2.yaml file to add the service account name and namespace:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: tmp-api-contact
  name: tmp-api-contact
  namespace: project-hamster          # add
spec:
  serviceAccountName: secret-reader   # add
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: curlimages/curl:7.65.3
    name: tmp-api-contact
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Apply the Pod configuration:

```bash
k -f e2.yaml create
```

Exec into the Pod:

```bash
k -n project-hamster exec tmp-api-contact -it -- sh
```

Within the container, use curl to access the Kubernetes API:

```bash
curl https://kubernetes.default
curl -k https://kubernetes.default # ignore insecure as allowed in ticket description
curl -k https://kubernetes.default/api/v1/secrets # should show Forbidden 403
```

To authenticate with the ServiceAccount, retrieve the token from the mounted folder:

```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

Now, make the curl request with the token:

```bash
curl -k https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
```

To use an encrypted HTTPS connection, use the following command:

```bash
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
curl --cacert ${CACERT} https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
```

To verify if the ServiceAccount has the necessary permissions, you can run:

```bash
k auth can-i get secret --as system:serviceaccount:project-hamster:secret-reader
```

Finally, write the commands into the requested location:

```txt
# /opt/course/e4/list-secrets.sh
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
```

</details>
