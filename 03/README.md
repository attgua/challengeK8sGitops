Create multi-container Pods, which containers share a directory.

- One pod should have both containers that run python. The First should install requirements, the second should run an application using those requirements. Describe your choices and describe why It Is an anti-pattern.




- One pod should have a container that prepares the file exposed on a Port by the other container, which runs nginx.

Container with nginx:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html


Container that prepares the file is a busybox:

  - image: busybox
    name: busy
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    args:
    - bin/sh
    - -c
    - echo hello world; ls; echo Hello from the busybox container > /pod-data/index.html ; sleep 3600


To test on the browser (with the accuracy that the service run the name of the pod, and the right port and with LoadBalancer as type)

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
        run: multicontainer-pod1
      sessionAffinity: None
      type: LoadBalancer
    status:
      loadBalancer: {}





