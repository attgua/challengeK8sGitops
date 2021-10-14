Deploy a container from scratch.


create Dockerfile

build and push Dockerfile on DockerHub

-> docker build -t attgua/challenge  . 
-> docker tag attgua/challenge attgua/challenge:0.1

-> docker push attgua/challenge:0.1

use the image in a pod like:

apiVersion: v1
kind: Pod
metadata:
   name: challenge-1
spec:
   containers:
   - name: challenge-1
     image: attgua/challenge:0.0.1
     ports:
     - containerPort: 6379


