# improperly-specified-authpolicy-yaml
Based on the problem described in https://discuss.istio.io/t/how-make-authorizationpolicy-work/7193, this example 
shows one of the two bugs. The first bug is a misunderstanding of the order of operations in Istio Authorization, caused by
assuming that an explicit `ALLOW` policy would supercede a `DENY` policy. The second bug is a mistake in the YAML
formatting, where even something as minor as an improper placement of a `-` in an Istio AuthorizationPolicy 
can break reachability. This example will only showcse the second bug (see `../auth-concept-misunderstanding` for the first bug).

This bug is mentioned both in the first linked post as well as [another](https://stackoverflow.com/questions/62367180/istio-authorizationpolicy-rules-questions/62427420#62427420) post from a different user (we suspect).
## Bug Explanation
The bug described by this user is due to the mistaken additional `-` before the `to` field.
This causes the AuthPolicy to have two separate rules instead of as one rule that has both `to` and `from`
fields, as intended. The resulting AuthorizationPolicy means that the `frontend-serviceaccount` can communicate
with the `cartservice` on any port, and ANY service can communicate with `cartservice` on port 7070. This
is a problem, because the intent was to allow only the `frontend-serviceaccount` to communicate on port 7070,
with all other traffic blocked.

The result is an unpleasant bug where it appears that the authpolicy worked, while you actually leave
 the `cartservice` able to receive arbitrary packets from any service in the mesh.


## Files
- `bugYaml/ap.yaml`: contains the improper AuthorizationPolicy. It was copied from the post linked above,
with slight modifications to include the abstraction choices described in `../../README.md`.
- `goodYaml/ap.yaml`: contains the same YAML as `bugYaml/ap.yaml` except it is correctly configured. 

## Behavior
### Bug 
To see the bug this example exposes, first run:

```kubectl apply -f bugYamls/```

Now, you should see the single applied AuthorizationPolicy by running:

```
kubectl get authorizationpolicy
```

Now, to see the bug we must SSH into a service that should not be able to connect to the `cartservice`. (That is,
the intention of the AuthPolicy is that the service can't reach `cartservice`, but in reality it will not restrict the traffic)

To witness the bad behavior, run:
```
$ kubectl get pods
$ kubectl exec --stdin --tty <fully qualified adservice pod name> -- /bin/bash
root@adservice-<ID>:/app# apt-get update
root@adservice-<ID>:/app# apt-get install curl
root@adservice-<ID>:/app# curl cartservice:7070
```
The result of `curl` is not an RBAC error, as we would like, it is actually a `cartservice`-side connection reset. The fact
that it is not an RBAC error indicates a vulnerability in `cartservice`, since it is reachable by any service at port 7070.
### Intended
To see the intended behavior, update the buggy `AuthorizationPolicy` by running (on your device, not in `adservice`):

```kubectl apply -f goodYaml/```

Now, go back to your SSH session within `adservice`. The same `curl cartservice:7070` command now properly returns an RBAC error, indicating that the traffic is not reaching the
`cartservice` due to the `AuthorizationPolicy`. 
