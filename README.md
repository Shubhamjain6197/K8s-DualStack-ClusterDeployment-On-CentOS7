### On the Master Node:

1. Install Docker
2. Start and enable Docker
3. Install kubeadm, kubelet, and kubectl
4. Disable swap
5. Initialize the master node using kubeadm
6. Set up the kubectl configuration
7. Install a pod network, such as Flannel

### On the Worker Nodes:

1. Install Docker
2. Start and enable Docker
3. Join the worker node to the cluster using the token generated during master node initialization
4. Set up the kubectl configuration
5. Repeat steps for the second worker node
6. After completing these steps, you should have a Kubernetes cluster with one master node and two worker nodes. You can verify the cluster status using the kubectl command.

##

### Here are the commands to set up a Kubernetes cluster with one master node and two worker nodes:

## On the Master Node:

**Install Docker:**
```
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf list docker-ce
dnf install docker-ce --nobest -y
systemctl start docker
systemctl enable docker
```
**Start and enable Docker:**
```
sudo systemctl start docker
sudo systemctl enable docker
```
**Install kubeadm, kubelet, and kubectl:**
```
sudo yum install -y kubeadm kubelet kubectl
```
**Disable swap:**
```
sudo swapoff -a
```
**Configure kubeadm to initialize the master node:**
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
**Set up the kubectl configuration:**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
**Install a pod network, such as Flannel or Tigara:**
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
## On the each Worker Nodes:

**Install Docker:**
```
sudo yum install -y docker
```
**Start and enable Docker:**
```
sudo systemctl start docker
sudo systemctl enable docker
```
**Join the worker node to the cluster using the token generated during master node initialization:**
```
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
*Replace <token> with the token generated during master node initialization, <master-ip> with the IP address of the master node, <master-port> with the port used by the Kubernetes API server (usually 6443), and <hash> with the SHA-256 hash of the discovery token certificate.*

**Set up the kubectl configuration:**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
*Repeat steps for the second worker node. After completing these steps, you should have a Kubernetes cluster with one master node and two worker nodes. You can verify the cluster status using the kubectl command.*
