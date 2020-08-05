# improperly-namespaced-networkpolicy

Based on the bug described in https://discuss.istio.io/t/networkpolicy-not-taking-effect/3341, this example 
shows how the improper namespacing of a Kubenetes NetworkPolicy can break the data-plane of an otherwise
healthy service mesh. 

## Files
- `bugYaml/np.yaml`: contains the improperly namespaced YAML. It was copied from the post linked above,
with slight modifications to include the abstraction choices described in `../../README.md`.
- `goodYaml/np.yaml`: contains the same YAML as `npWrongNamespace` except it is correctly configured. 

## Behavior
### Bug
To see the bug this example exposes, first run

```kubectl apply -f goodYaml/```

Now, you should see the applied NetworkPolicy by running:

```kubectl get NetworkPolicy```

Now, try to access the website at the IP address you located in the initial setup. **The site should still be visible, as the NetworkPolicy is not applied to the correct namespace.**

### Intended
To see the intended behavior, remove the previous NetworkPolicy by running:

```kubectl delete -f bugYaml/```

Now, apply the correct policy with:

```kubectl apply -f goodYaml/```

To see the policy now, you must run the previous command with the `namespace` flag:

```kubectl get NetworkPolicy --namespace=istio-system```

Now, if you try to go to the web page it will be unreachable, because the NetworkPolicy is operating over the correct domain.
