# Question 9 | Find Pods first to be terminated

Check all available Pods in the Namespace `project-c13` and find the names of those that would probably be terminated first if the nodes run out of resources (cpu or memory) to schedule all Pods. Write the Pod names into `/opt/course/e1/pods-not-stable.txt`.

<details><summary>Solution:</summary>

When available cpu or memory resources on the nodes reach their limit, Kubernetes will look for Pods that are using more resources than they requested. These will be the first candidates for termination. If some Pods containers have no resource requests/limits set, then by default those are considered to use more than requested.

Kubernetes assigns Quality of Service classes to Pods based on the defined resources and limits, read more here: [Kubernetes Quality of Service Pod](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod)

Hence we should look for Pods without resource requests defined, we can do this with a manual approach:

```bash
k -n project-c13 describe pod | less -p Requests # describe all pods and highlight Requests
```

Or we do:

```bash
k -n project-c13 describe pod | egrep "^(Name:|    Requests:)" -A1
```

We see that the Pods of Deployment `c13-3cc-runner-heavy` don't have any resources requests specified. Hence our answer would be:

```txt
# /opt/course/e1/pods-not-stable.txt
c13-3cc-runner-heavy-65588d7d6-djtv9map
c13-3cc-runner-heavy-65588d7d6-v8kf5map
c13-3cc-runner-heavy-65588d7d6-wwpb4map
o3db-0
o3db-1 # maybe not existing if already removed via previous scenario 
```

To automate this process you could use jsonpath like this:

```bash
k -n project-c13 get pod \
  -o jsonpath="{range .items[*]} {.metadata.name}{.spec.containers[*].resources}{'\n'}"
```

This lists all Pod names and their requests/limits, hence we see the three Pods without those defined.

Or we look for the Quality of Service classes:

```bash
k get pods -n project-c13 \
  -o jsonpath="{range .items[*]}{.metadata.name} {.status.qosClass}{'\n'}"
```

Here we see three with `BestEffort`, which Pods get that don't have any memory or cpu limits or requests defined.

A good practice is to always set resource requests and limits. If you don't know the values your containers should have you can find this out using metric tools like Prometheus. You can also use `kubectl top pod` or even `kubectl exec` into the container and use `top` and similar tools.

</details>
