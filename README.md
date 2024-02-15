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

swapoff -a  #make sure this cmd is executed
yum install docker
systemctl enable docker --now
systemctl start docker --now
docker info | grep -i cgroup

vi /etc/docker/daemon.json
 
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

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
** Setup Routes**
```
yum install iproute-tc
```

** sysctl params required by setup, params persist across reboot ** 
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
** Apply sysctl params without reboot **
```
sudo sysctl --system

sudo mkdir /etc/containerd
sudo containerd config default > /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl restart kubelet
```



*Repeat steps for the second worker node. After completing these steps, you should have a Kubernetes cluster with one master node and two worker nodes. You can verify the cluster status using the kubectl command.*
