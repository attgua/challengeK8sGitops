## Install a Kubernetes cluster and create a backup of its etcd.



To create a K8s cluster I chose to use as node 3 machines on Virtualbox with installed Ubuntu server 20.04.

The following are the list of commands which I used and the procedure which I followed.

### *Step 1: Install Kubernetes Servers* 

```
sudo apt update
sudo apt -y upgrade && sudo systemctl reboot
```

### *Step 2: Install kubelet, kubeadm and kubectl*

```
sudo apt update
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### *Step 3: Disable Swap*

```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a


# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```

### *Step 4: Install Container runtime [I have used Docker, but others where also possible (CRI-O, Containerd...)]*

```
# Add repo and Install packages
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io docker-ce docker-ce-cli

# Create required directories
sudo mkdir -p /etc/systemd/system/docker.service.d

# Create daemon json config file
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Start and enable Services
sudo systemctl daemon-reload 
sudo systemctl restart docker
sudo systemctl enable docker
```

### *Step 5: Initialize master node*

```

# Make sure that the br_netfilter module is loaded 
lsmod | grep br_netfilter

sudo systemctl enable kubelet

sudo kubeadm config images pull --cri-socket /var/run/docker.sock


# Check host name on master
sudo vim /etc/hosts

# -> 192.168.0.3 atti-master

# Create cluster

sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --upload-certs \
  ----control-plane-endpoint=atti-master


# Configure kubectl using commands in the output:

mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

### *Step 6: Install network plugin on Master (Calico)*

```
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml 
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml

# Eventually untaint master node

```


### *Step 7: Add worker nodes*

```

# Check host name on worker machine

sudo vim /etc/hosts

# -> 192.168.0.3 atti-master


# The join command that was given is used to add a worker node to the cluster.

kubeadm join atti-master \
  --token ... \
  --discovery-token-ca-cert-hash ...

```

#### The first 4 point where executed on all the nodes. Point 5 and 6 only on the Master while point 7 on the workers.


### *Etcd Backup*


```

# Backup certificates
sudo cp -r /etc/kubernetes/pki backup/
# Make etcd snapshot
sudo docker run --rm -v $(pwd)/backup:/backup \
    --network host \
    -v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd:3.4.3-0 \
    etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
    --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
    snapshot save /backup/etcd-snapshot-latest.db

# Backup kubeadm-config
sudo cp /etc/kubeadm/kubeadm-config.yaml backup/


```

- The first commnad copies the folder containing all the certificates that kubeadm creates.

- In the second command the idea is to create a snapshot of the etcd database. 
This is done by communicating with the running etcd instance in Kubernetes and asking it to create a snapshot. 
The reason for this long command is to avoid messing with etcd running in Kubernetes as much as possible. It launch a separate container using the same docker image that kubeadm used for setting up the cluster (k8s.gcr.io/etcd:3.4.3-0). But in order to communicate with the etcd pod in Kubernetes, it is needed to: 
  - Use the host network in order to access 127.0.0.1:2379, where etcd is exposed (--network host)

  - Mount the backup folder where it is desidered to save the snapshot (-v $(pwd)/backup:/backup)

 - Mount the folder containing the certificates needed to access etcd (-v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd)

 - Specify the correct etcd API version as environment variable (--env ETCDCTL_API=3)

 - The actual command for creating a snapshot (etcdctl snapshot save /backup/etcd-snapshot-latest.db)

 - Some flags for the etcdctl command:
      - Specify where to connect to (--endpoints=https://127.0.0.1:2379)
      
      - Specify certificates to use (--cacert=..., --cert=..., --key=...)

   *So it is started a docker container with the etcdctl tool installed. It is ordered to create a snapshot of the etcd instance running in the Kubernetes cluster and store it in a backup folder that is mounted from the host.*

- The final command is optional and only relevant if is used as configuration file for kubeadm.


When the time has come to restore the control plane, it is necessary to copy back everything from the backup and initiate the control plane again.


Restoration:

```

# Restore certificates
sudo cp -r backup/pki /etc/kubernetes/

# Restore etcd backup
sudo mkdir -p /var/lib/etcd
sudo docker run --rm \
    -v $(pwd)/backup:/backup \
    -v /var/lib/etcd:/var/lib/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd:3.4.3-0 \
    /bin/sh -c "etcdctl snapshot restore '/backup/etcd-snapshot-latest.db' ; mv /default.etcd/member/ /var/lib/etcd/"

# Restore kubeadm-config
sudo mkdir /etc/kubeadm
sudo cp backup/kubeadm-config.yaml /etc/kubeadm/

# Initialize the master with backup
sudo kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd \
    --config /etc/kubeadm/kubeadm-config.yaml


```

This is a reversal of the previous steps.

[Resource: https://elastisys.com/backup-kubernetes-how-and-why/] 


