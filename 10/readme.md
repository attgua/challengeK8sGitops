## Configure network policies in two different namespaces so that the pod of namespaceA can communicate with pod of namespaceB but not viceversa.


With the networkpolicy A I have that no one can communicate with namespaceA (so either from namespaceB) because I blocked all the ingress in the namespace. I have also added a policy on namespaceB so it can't communicate outside (I have blocked all the egress)

```
* kubectl create ns namespace-a
* kubectl create ns namespace-b

* kubectl run nginx --image=nginx --port=80 --n namespace-a
* kubectl run nginx --image=nginx --port=80 --n namespace-b

* kubectl apply -f networkpolicy-a.yaml
* kubectl apply -f networkpolicy-a.yaml
```

Now to test enter in the nginx and try a curl to the other namespace.
For example:

```
* kubectl get pod -o wide

take note of the pod ip address

* kubectl exec nginx -n namespace-a -- bin/sh
- curl ip address_a

* kubectl exec nginx -n namespace-b -- bin/sh
- curl ip address_b
```

evaluate the differences.
