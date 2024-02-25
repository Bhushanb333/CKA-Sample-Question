# Question 7 | NetworkPolicy=

There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

To prevent this, create a NetworkPolicy called `np-backend` in Namespace `project-snake`. It should allow the `backend-*` Pods only to:

- Connect to `db1-*` Pods on port 1111
- Connect to `db2-*` Pods on port 2222

Use the `app` label of Pods in your policy.

After implementation, connections from `backend-*` Pods to `vault-*` Pods on port 3333 should, for example, no longer work.

<details><summary>Solution:</summary>

First, we look at the existing Pods and their labels:

```shell
k -n project-snake get pod
k -n project-snake get pod -L app
```

We test the current connection situation and see nothing is restricted:

```shell
k -n project-snake get pod -o wide
k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
```

Now we create the NP by copying and changing an example from the k8s docs:

```yaml
vim 24_np.yaml
# 24_np.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress                    # policy is only about Egress
  egress:
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db1
      ports:                        # second condition "port"
      - protocol: TCP
        port: 1111
    -                           # second rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db2
      ports:                        # second condition "port"
      - protocol: TCP
        port: 2222
```

The NP above has two rules with two conditions each. It can be read as:

Allow outgoing traffic if:

Destination pod has label app=db1 AND port is 1111
OR
Destination pod has label app=db2 AND port is 2222
Create the NetworkPolicy:

```shell
k -f 24_np.yaml create
```

And test again:

```shell
k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
```

Also, helpful to use kubectl describe on the NP to see how Kubernetes has interpreted the policy.

Great, looking more secure. Task done.

</details>
