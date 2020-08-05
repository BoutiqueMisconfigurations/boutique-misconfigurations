# rbac
This example requires more setup than some of the others. It showcases how using the wrong type of 
`RoleBinding` can create namespace-dependent access control accidentally, a la [this](https://stackoverflow.com/questions/60877743/kubernetes-rbac-allowed-by-rolebinding-but-cannot-list-resource) StackOverflow post.

## Setup
Please perform the following step to setup the RBAC system correctly:
1) Add a user named `admin-sre` following the steps detailed [here](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user). There are a few differences, however, so read this before you begin:
   - Do not perform the steps in `Create Role and Role Binding`, we will do this later.
   - Do not run the last command `kubectl config use-context`.
   - CSR YAML files are provided with placeholders for the key you generated. You therefore do not need to run `cat <<EOF...` in the  section labelled *Create Certificate Request Kubernetes Object*. Once filled in, you can simply call `kubectl apply -f admin-sre-csr.yaml`.
## Bug
The objective in this example is to give the `admin-sre` user the ability to read all pods in the entire cluster.
Due to the namespacing of `RoleBinding` objects, however, the StackOverflow OP from above mistakenly only gave read
permission to one namespace within the cluster, *despite using a ClusterRole object*. To witness this bug, simply
run
```
kubectl apply -f bugYaml/
```

Now, we can see the improper behavior by running

```
kubectl auth can-i get pods --as admin-sre -n=kube-system
```

```
kubectl auth can-i get pods --as admin-sre -n=default
```

As you can see, we are able to execute the first command, but we are unable to
run the second, because the second is operating in the `default` namespace,
overwhich `admin-sre` does not yet have authority to read.

## Proper Behaviour
To see the correct behavior first run

```
kubectl delete -f bugYaml/
```

then

```
kubectl apply -f goodYaml/
```

Now, when you run

```
kubectl auth can-i get pods --as admin-sre -n=kube-system
```

```
kubectl auth can-i get pods --as admin-sre -n=default
```

both should work.
