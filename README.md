### On the Master(Control Plane) Nodes:

1. Install Docker
2. Start and enable Docker
3. Install kubeadm, kubelet, and kubectl
4. Disable swap
5. Initialize the master node using kubeadm
6. Set up the kubectl configuration
7. Install a pod network, such as Calico,Flannel
8. Assign role to worker nodes.

### On the Worker Nodes:

1. Install Docker
2. Start and enable Docker
3. Set up the kubectl configuration
4. Join the worker node to the cluster using the token generated during master node initialization
5. Repeat steps for the second worker node
6. After completing these steps, you should have a Kubernetes cluster with one master node and two worker nodes. You can verify the cluster status using the kubectl command.

##

### Here are the commands to set up a Kubernetes cluster with one master node and multiple worker nodes:

## On the Master(Control Plane) Node & Worker Nodes:

**Disable Swap and Firewalld**
```
swapoff -a
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
systemctl disable firewalld
systemctl stop firewalld

```
**Install Docker:**
```
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf list docker-ce
dnf install docker-ce --nobest -y
systemctl start docker
systemctl enable docker
```
```
yum install docker
systemctl enable docker --now
systemctl start docker --now
docker info | grep -i cgroup
```
```
vi /etc/docker/daemon.json

{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```
```
systemctl restart docker
```

**Setup Kubernets repos:**

```
sudo tee /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```
**Install kubeadm, kubelet, and kubectl:**
```
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet && sudo systemctl start kubelet

```

**Setup Routes**

```
yum install iproute-tc

```
**Sysctl params required by setup, params persist across reboot** 

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

**Apply sysctl params without reboot**
```
sudo sysctl --system

sudo mkdir /etc/containerd
sudo containerd config default > /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

## On Master(Control Plane) only

**Pull Kubeadm images and enable IPV4/6**

```
kubeadm config images pull
sudo sysctl -w net.ipv4.conf.all.forwarding=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
```

**Intialize Kubeadm for Master(Control Plane) Node**

```
kubeadm init --pod-network-cidr=192.168.0.0/16,2001:db8:42:0::/56 --service-cidr=10.96.0.0/16,2001:db8:42:1::/112
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=/etc/kubernetes/kubelet.conf
```
**Install CNI Tigera**
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
**Create Custom Resource file to setup Calico Network**
```
vi custom-resources.yaml
```
```
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
   # Configures Calico networking.
  calicoNetwork:
     # Note: The ipPools section cannot be modified post-install.
    ipPools:
      - blockSize: 26
        cidr: 192.168.0.0/16
        encapsulation: IPIP
        natOutgoing: Enabled
        nodeSelector: all()
      - blockSize: 122
        cidr: 2001:db8:42:0::/56
        encapsulation: None
        natOutgoing: Enabled
        nodeSelector: all()
---
# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```
```
kubectl apply -f custom-resources.yaml
kubectl get pods -A 
```
## Commands to be executed on Worker Nodes only

**Kubeadm inti commands will generate below command**
```
kubeadm join 10.0.0.74:6443 --token 455py3.xxxxxxxx \
        --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxx
```
*If you loose join commands you can regenerate it by running below command on master node as join command us valid only for 24 hours.*

```
kubeadm token create --print-join-command
```
## Commands to be executed on Master(Control Plane) Nodes only

**Now we need to assign role to newly setup worker node**

```
kubectl label nodes worker-node-name kubernetes.io/role=worker-node
```

*Repeat steps for the second worker node. After completing these steps, you should have a Kubernetes cluster with one master node and two worker nodes. You can verify the cluster status using the kubectl command.*

**Sample application to test the setup**

```
https://github.com/dockersamples/example-voting-app
```

## Some usefull commands

**Reset Kubeadm**
```
kubeadm reset
```
**Kubernetes Commands**
```
kubectl get nodes
kubectl get pods -A -o wide #this will give all the pod details in all namespaces
kubectl get pods -n kube-system
kubectl describe pod pod-name
kubectl logs -f pod-name
```
