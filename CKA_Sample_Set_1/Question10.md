# Question 10 | RBAC ServiceAccount Role RoleBinding

Create a new ServiceAccount `processor` in Namespace `project-hamster`. Create a Role and RoleBinding, both named `processor` as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

<details>
<summary>Solution:</summary>

Let's talk a little about RBAC resources.
A ClusterRole/Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.
A ClusterRoleBinding/RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.
Because of this, there are 4 different RBAC combinations and 3 valid ones:

- Role + RoleBinding (available in single Namespace, applied in single Namespace)
- ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
- ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
- Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)

To the solution, first we create the ServiceAccount:

```shell
kubectl -n project-hamster create sa processor
```

Then, we create the Role:

```shell
kubectl -n project-hamster create role processor \
  --verb=create \
  --resource=secret \
  --resource=configmap
```

This creates a Role named processor with the necessary permissions.

Next, we create the RoleBinding:

```shell
kubectl -n project-hamster create rolebinding processor \
  --role=processor \
  --serviceaccount=project-hamster:processor
```

This binds the Role processor to the ServiceAccount processor .

To test the RBAC setup, we can use kubectl auth can-i command:

```shell
kubectl -n project-hamster auth can-i create secret \
  --as=system:serviceaccount:project-hamster:processor

kubectl -n project-hamster auth can-i create configmap \
  --as=system:serviceaccount:project-hamster:processor
```

</details>
