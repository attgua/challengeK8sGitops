## Create a deployment, a statefulset and a daemonset describing their differences


### *Deployment*

A deployment provides declarative approach to descibe pods and ReplicaSet.

It is a Kubernetes controller that matches the current state of the cluster to the desired state mentioned in the Deployment manifest and scale the pod trough the ReplicaSet.

### *Statefulset*

StatefulSet is the workload API object used to manage stateful applications.

Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

Every replica of a stateful set will have its own state, and each of the pods will be creating its own PVC(Persistent Volume Claim). A statefulset with 3 replicas will create 3 pods, each having its own Volume, so total 3 PVCs

### *Daemonset*

A DaemonSet is a controller that ensures that the pod runs on all the nodes of the cluster.
If a node is added/removed from a cluster, DaemonSet automatically adds/deletes the pod.


[Reference: https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4]
