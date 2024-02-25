# Question 6 | Kubelet client/server cert info

Node `cluster2-node1` has been added to the cluster using `kubeadm` and TLS bootstrapping.

Find the "Issuer" and "Extended Key Usage" values of the `cluster2-node1`:

- Kubelet client certificate, the one used for outgoing connections to the kube-apiserver.
- Kubelet server certificate, the one used for incoming connections from the kube-apiserver.

Write the information into file `/opt/course/23/certificate-info.txt`.

Compare the "Issuer" and "Extended Key Usage" fields of both certificates and make sense of these.

<details><summary>Solution:</summary>

To find the correct kubelet certificate directory, we can look for the default value of the `--cert-dir` parameter for the kubelet. For this, search for "kubelet" in the Kubernetes docs which will lead to: [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet). We can check if another certificate directory has been configured using `ps aux` or in `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`.

First, we check the kubelet client certificate:

```shell
ssh cluster2-node1
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep Issuer
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep "Extended Key Usage" -A1
```

Next, we check the kubelet server certificate:

```shell
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep Issuer
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep "Extended Key Usage" -A1
```

We see that the server certificate was generated on the worker node itself and the client certificate was issued by the Kubernetes API. The "Extended Key Usage" also shows if it's for client or server authentication.

More about this: [Kubelet TLS Bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping)

</details>
