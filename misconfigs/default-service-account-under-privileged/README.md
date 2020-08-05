# default-service-account-under-privileged
The `default ServiceAccount` does not begin with the privileges to read
pods or nodes. This can result in RBAC bugs for production systems
where build tools (e.g., gitlab) can fail. Consider [this](https://stackoverflow.com/questions/53908848/kubernetes-pods-nodes-is-forbidden/53909115#53909115) forum post.
This user  expects to be able to build but it is failing due to the `defualt ServiceAccount`'s lack of privileges.
## Bug
To witness the bug, you can simply run

```
$ kubectl auth can-i read pods --as system:serviceaccount:default:default
```
This will output `no` due to the bug described above.

## Fix
To fix this bug you can simply run

```
kubectl apply -f goodYaml/
```

This applies a very slightly modified version of the answer to the StackOverflow post linked above,
as it properly addresses the bug.

