# rbac-binding-over-wrong-resources
This example demonstrates a misapplication of RoleBindings. The user 
in this [post](https://stackoverflow.com/questions/59465936/rbac-role-to-manage-single-pod-with-dynamic-name/59471112#59471112)
wanted to perform RBAC at the `pod` level, but due to the ephemeral nature of pods they
should have used a RoleBinding at the deployment level.

## Setup
To set this up, please configure the `boutique-sre` user as per `../rbac-role-binding-misuse/README.md`/.

## Bug
To witness the bug, just run

```
kubectl apply -f bugYaml/
```
To see the problem, just run
```
kubectl auth can-i get pods --as boutique-sre
```
As you can see, they are unable to get the pods.

## Intended
The intention of this post was to be able to give the user permissions over pods. 
However, because pods are ephemeral and because `RoleBinding`s do not support
wildcards this is not possible. The accepted solution circumvents this problem
by advising the OP to give the User access to all `pods` in the relevant namespace.

To do this, you can apply the deployment-based `RoleBinding` by running

```
kubectl delete -f bugYaml/
kubectl apply -f goodYaml/
```

You can now see the proper behavior if you run

```
kubectl auth can-i get pods --as boutique-sre
```

Interestingly, there is no way to restrict the user to only be allowed to read pods within
one deployment. The best way to approximate this would be to put the Deployment in its own namespace,
but that raises new potential problems.

