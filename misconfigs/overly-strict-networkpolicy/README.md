# overly-strict-networkpolicy
This bug is caused by an over-constraint within a `NetworkPolicy`. While
working to secure their cluster with service-to-service traffic restrictions,
the OP of [this post](https://stackoverflow.com/questions/47327554/kubernetes-networkpolicy-allow-loadbalancer)
 actually banned all traffic from the frontend `LoadBalancer`. This led to a disconnection of their service.

## Bug
To see this bug, we first need to label the frontend-external loadbalancer. To do so,  run:
```
kubectl get pods
kubectl label <frontend-pod> lbl=frontend-loadbalancer
```


```
kubectl apply -f bugYaml/
```

## Intended Behavior
The intended behavior of the admin in this case was misguided, as you shouldn't restrict
traffic to the external-facing load balancer. Thus, the proper behavior can be seen by just
running:
```
kubectl delete -f bugYaml/
```
