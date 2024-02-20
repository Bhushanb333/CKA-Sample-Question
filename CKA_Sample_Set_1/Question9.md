# Question 9 | Kill Scheduler, Manual Scheduling

Ssh into the controlplane node with `ssh cluster2-controlplane1`. Temporarily stop the kube-scheduler, this means in a way that you can start it again afterwards.

Create a single Pod named `manual-schedule` of image `httpd:2.4-alpine`, confirm it's created but not scheduled on any node.

Now you're the scheduler and have all its power, manually schedule that Pod on node `cluster2-controlplane1`. Make sure it's running.

Start the kube-scheduler again and confirm it's running correctly by creating a second Pod named `manual-schedule2` of image `httpd:2.4-alpine` and check if it's running on `cluster2-node1`.

<details>
<summary>Solution:</summary>

Stop the Scheduler
First we find the controlplane node:

```shell
k get node

NAME                     STATUS   ROLES           AGE   VERSION
cluster2-controlplane1   Ready    control-plane   26h   v1.28.2
cluster2-node1           Ready    <none>          26h   v1.28.2
```

Then we connect and check if the scheduler is running:

```shell
ssh cluster2-controlplane1

root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule

kube-scheduler-cluster2-controlplane1            1/1     Running   0          6s
Kill the Scheduler (temporarily):

root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/

root@cluster2-controlplane1:~# mv kube-scheduler.yaml ..
```

And it should be stopped:

```shell
root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule

root@cluster2-controlplane1:~# 
```

Create a Pod Now we create the Pod:

```shell
k run manual-schedule --image=httpd:2.4-alpine
```

And confirm it has no node assigned:

```shell
k get pod manual-schedule -o wide

NAME              READY   STATUS    ...   NODE     NOMINATED NODE
manual-schedule   0/1     Pending   ...   <none>   <none>        
```

Manually schedule the Pod Let's play the scheduler now:

```shell
k get pod manual-schedule -o yaml > 9.yaml
```

```yaml
# 9.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-04T15:51:02Z"
  labels:
    run: manual-schedule
  managedFields:
...
    manager: kubectl-run
    operation: Update
    time: "2020-09-04T15:51:02Z"
  name: manual-schedule
  namespace: default
  resourceVersion: "3515"
  selfLink: /api/v1/namespaces/default/pods/manual-schedule
  uid: 8e9d2532-4779-4e63-b5af-feb82c74a935
spec:
  nodeName: cluster2-controlplane1        # add the controlplane node name
  containers:
  - image: httpd:2.4-alpine
    imagePullPolicy: IfNotPresent
    name: manual-schedule
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-nxnc7
      readOnly: true
  dnsPolicy: ClusterFirst
```

The only thing a scheduler does, is that it sets the nodeName for a Pod declaration. How it finds the correct node to schedule on, that's a very much complicated matter and takes many variables into account.

As we cannot kubectl apply or kubectl edit , in this case we need to delete and create or replace:

```shell
k -f 9.yaml replace --force
```

How does it look?

```shell
k get pod manual-schedule -o wide

NAME              READY   STATUS    ...   NODE            
manual-schedule   1/1     Running   ...   cluster2-controlplane1
```

It looks like our Pod is running on the controlplane now as requested, although no tolerations were specified. Only the scheduler takes tains/tolerations/affinity into account when finding the correct node name. That's why it's still possible to assign Pods manually directly to a controlplane node and skip the scheduler.

Start the scheduler again

```shell
ssh cluster2-controlplane1

root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/

root@cluster2-controlplane1:~# mv ../kube-scheduler.yaml .
```

Checks it's running:

```shell
root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule
kube-scheduler-cluster2-controlplane1            1/1     Running   0          16s
```

Schedule a second test Pod:

```shell
k run manual-schedule2 --image=httpd:2.4-alpine
k get pod -o wide | grep schedule

manual-schedule    1/1     Running   ...   cluster2-controlplane1
manual-schedule2   1/1     Running   ...   cluster2-node1
```

</details>
