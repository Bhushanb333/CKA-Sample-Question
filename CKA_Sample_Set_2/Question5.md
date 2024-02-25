# Question 5 | Check how long certificates are valid

Check how long the `kube-apiserver` server certificate is valid on `cluster2-controlplane1`. Do this with `openssl` or `cfssl`. Write the expiration date into `/opt/course/22/expiration`.

Also, run the correct `kubeadm` command to list the expiration dates and confirm both methods show the same date.

Write the correct `kubeadm` command that would renew the `apiserver` server certificate into `/opt/course/22/kubeadm-renew-certs.sh`.

<details><summary>Solution:</summary>

First, let's find that certificate:

```shell
ssh cluster2-controlplane1
find /etc/kubernetes/pki | grep apiserver
```

Next, we use openssl to find out the expiration date:

```shell
openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
```

There we have it, so we write it in the required location on our main terminal:

```txt
# /opt/course/22/expiration
Dec 20 18:05:20 2023 GMT
```

And we use the feature from kubeadm to get the expiration too:

```shell
kubeadm certs check-expiration | grep apiserver
```

Looking good. And finally, we write the command that would renew all certificates into the requested location:

```txt
# /opt/course/22/kubeadm-renew-certs.sh
kubeadm certs renew apiserver
```

</details>
