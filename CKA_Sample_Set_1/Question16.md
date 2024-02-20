# Question 16 | Namespaces and Api Resources

Write the names of all namespaced Kubernetes resources (like Pod, Secret, ConfigMap...) into `/opt/course/16/resources.txt`.

Find the `project-*` Namespace with the highest number of Roles defined in it and write its name and the amount of Roles into `/opt/course/16/crowded-namespace.txt`.

<details>
<summary>Solution:</summary>

Namespace and Namespaced Resources

To get a list of all namespaced resources, you can use the following commands:

```bash
kubectl api-resources    # shows all resources

kubectl api-resources -h # help is always good

kubectl api-resources --namespaced -o name > /opt/course/16/resources.txt
```

This will result in the file /opt/course/16/resources.txt containing the names of all namespaced resources.

Namespace with the Most Roles
To find the project-* Namespace with the highest number of Roles defined in it, you can use the following commands:

```bash
kubectl -n project-c13 get role --no-headers | wc -l

kubectl -n project-c14 get role --no-headers | wc -l

kubectl -n project-hamster get role --no-headers | wc -l

kubectl -n project-snake get role --no-headers | wc -l

kubectl -n project-tiger get role --no-headers | wc -l
```

From the results, we can see that which namespace has the highest number of Roles with resources.

Finally, we can write the name and amount into the file /opt/course/16/crowded-namespace.txt :

```bash
# /opt/course/16/crowded-namespace.txt
project-c14 with 300 resources
```

</details>
