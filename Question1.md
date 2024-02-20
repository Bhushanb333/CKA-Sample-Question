# Question 1 | Contexts

1.} You have access to multiple clusters from your main terminal through `kubectl` contexts. Write all those context names into `/opt/course/1/contexts`.

2.} Next write a command to display the current context into `/opt/course/1/context_default_kubectl.sh`, the command should use `kubectl`.

3.} Finally write a second command doing the same thing into `/opt/course/1/context_default_no_kubectl.sh`, but without the use of `kubectl`.

Answer:

**1.} Solution**
Maybe the fastest way is just to run:

```shell
k config get-contexts # copy manually
```
Or

```shell
k config get-contexts -o name > /opt/course/1/contexts
```

Or using jsonpath:

```shell
k config view -o yaml # overview
k config view -o jsonpath="{.contexts[*].name}"
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" # new lines
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" > /opt/course/1/contexts
```

The content should then look like:

```shell
cat /opt/course/1/contexts
k8s-c1-H
k8s-c2-AC
k8s-c3-CCC
```

**2.} Solution**
Next create the first command:
```shell
vi /opt/course/1/context_default_kubectl.sh
kubectl config current-context
```

Verify execution
```shell
sh /opt/course/1/context_default_kubectl.sh
k8s-c1-H
```
**3.} Solution**
And the second one:

```shell
vi /opt/course/1/context_default_no_kubectl.sh
```
and add line

```
cat ~/.kube/config | grep current
```
Verify execution
```shell
sh /opt/course/1/context_default_no_kubectl.sh
current-context: k8s-c1-H
```

In the real exam you might need to filter and find information from bigger lists of resources, hence knowing a little jsonpath, vi editor and simple bash filtering will be helpful.

The second command could also be improved to:

```shell
vi /opt/course/1/context_default_no_kubectl.sh
cat ~/.kube/config | grep current | sed -e "s/current-context: //"
```
