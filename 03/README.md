## Create multi-container Pods, which containers share a directory.

- One pod should have both containers that run python. The First should install requirements, the second should run an application using those requirements. Describe your choices and describe why It Is an anti-pattern.

  * In A/app is present the Python application that could be tested alone with:
    ```
    
    pip install -r requirements.txt
    python main.py

    ```
  * In A/docker are presents the three Docker images used:
    - the first one is used for the test application.
    - the second one is used for install the requirements for the task application.
    - the third one is used for execute the flask server for the task application.

  * in A/Kubernetes are present two .yaml files:
    - the deployment.yaml is used for the test application.
    - the multicontainer-pod1.yaml is used for the task application.
      Inside the multicontainer-pod1.yaml:
        - tha first (init)Container named `python-write` has the image `attgua/python-write:0.15` builded with the second Dockefile and starting first install the requirements on a shared folder.
        - the second container named `python-execute`, that has the image `attgua/python-execute:0.15`, builded with the third dockerfile, moves the requirements installed in the the right python3 folder and starts the main.py application.
        - the service `hello-python-service` is used to expose the application on the port `30992` so its visible outside the cluster.

  * It is an anti-pattern because:
    - The minimum unit controlled by k8s is the pod so having multiple images in the same pod bring to different problems like:
      - if only one image is not working correctly we have to kill them all by restarting the pod even if they work correctly.
      - Replication of equal units scales worse
      - Having images that do different things within the same block is inconvenient during the checking or troubleshooting phase.
      - However it works but it is indeed an anti-pattern. What is typically worthwhile is to keep a 1-1 image pod report (Excluding those automatically generated such as that of the ip address etc).

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





