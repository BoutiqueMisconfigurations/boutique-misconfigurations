#improperly-specified-authpolicy-yaml
Based on the problem described in https://discuss.istio.io/t/how-make-authorizationpolicy-work/7193, this example 
shows one of the two bugs. The first bug is a misunderstanding of the order of operations in Istio Authorization, caused by
assuming that an explicit `ALLOW` policy would supercede a `DENY` policy. The second bug is a mistake in the YAML
formatting, where even something as minor as an improper placement of a `-` in an Istio AuthorizationPolicy 
can break reachability. This example will only showcse the first bug (see `../auth-stray-dash` for the second bug)

The bug described by this user is due to the mistaken additional `-` before the `to` field, which
acts as two separate rules instead of as one rule that has both `to` and `from` fields, as intended.

## Files
- `bugYaml/ap.yaml`: contains the improper AuthorizationPolicy. It was copied from the post linked above,
with slight modifications to include the abstraction choices described in `../../README.md`.
- `goodYaml/ap.yaml`: contains the same YAML as `bugYaml/ap.yaml` except it is correctly configured. 

## Behavior
### Bug 
To see the bug this example exposes, first run:

```kubectl apply -f bugYamls/```

Now, you should see two applied AuthorizationPolicies by running:

```
kubectl get authorizationpolicy
```

Now, try to access the website at the IP address you located in the initial setup (hard refresh or incognito to ensure 
a full reload). You will see something similar to:
```
HTTP Status: 500 Internal Server Error

rpc error: code = PermissionDenied desc = RBAC: access denied
could not retrieve cart
```

### Intended
To see the intended behavior, delete the buggy `AuthorizationPolicy` running:

```kubectl delete -f bugYaml/```

Now, apply the correct policy with:

```kubectl apply -f goodYaml/```

Now, if you navigate to the web page the cart is reachable, and the AuthorizationPolicy is now correctly applied. To
somewhat verify this, you can try `ssh`ing into another node and sending traffic to the `cartservice`.

I did this by running the following commands:

```
$ kubectl get pods
$ kubectl exec --stdin --tty <fully qualified adservice pod name> -- /bin/bash
root@adservice-<ID>:/app# apt-get update
root@adservice-<ID>:/app# apt-get install curl
root@adservice-<ID>:/app# curl cartservice:7070
```
The output of this `curl` call is an RBAC error, indicating that the traffic is not reaching the
`cartservice` due to the `AuthorizationPolicy`. If you are still not convinced it is working, you can `kubectl delete -f goodYaml/`, and then run `curl` from the `adservice` again. This time the error is a remote reset, indicating that the traffic reached the `cartservice`.
