# challengeK8sGitops

1. Deploy a container from scratch.
2. Check the Image version without the describe command.
3. Create multi-container Pods, which containers share a directory.
  A. One pod should have both containers that run python. The First should install requirements, the second should run an application using those requirements. Describe your choices and describe why It Is an anti-pattern.
  B. One pod should have a container that prepares the file exposed on a Port by the other container, which runs nginx.
4. Expose the previous nginx container using a Kubernetes ingress and makes It reachable from your environment on a FQDN.
5. List all the pods showing name and namespace with a json path expression.
6. Prepare an Helm Chart deploying 4.
7. Prepare and package an Helm Chart that configures RBAC for a list of namespaces. Describe your choices in the Chart README.
8. Install ArgoCD and create a project that installs 7.
9. Install a Kubernetes cluster and create a backup of its etcd.
10. Configure network policies in two different namespaces so that the pod of namespaceA can communicate with pod of namespaceB but not viceversa.
11. Create a deployment, a statefulset and a daemonset describing their differences
12. Prepare an Helm Chart managing two different environments.
13. Create a pod that mounts a secret and a configMap both as environmental variables and as files.
14. Deploy a pod that runs with a security context and verify its user. 