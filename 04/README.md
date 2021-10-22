# Expose nginx container

Already implemented in point 3B for test operations.
The service (part of the multicontainer-pod2 file) exposes nginx on port 30991.
The ip address instead depends on which node of the cluster will take charge of the pod.


```
apiVersion: v1
kind: Service
metadata:
  labels:
    run: exposition-service
  name: exposition-service
  namespace: default
spec:
  ports:
  - nodePort: 30991
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: multicontainer-pod2
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer: {}

```