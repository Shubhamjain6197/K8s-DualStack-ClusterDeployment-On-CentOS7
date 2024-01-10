# K8s Dual Stack Cluster On - Oracle Linux 

This Document defines the process to setup Kubernetes Dual Stack on Oracle Linux v8

**Below steps needs to be executed on Master and Worker Nodes.**

```
$ swapoff -a

$ yum install docker

$ systemctl enable docker --now
$ systemctl start docker --now

$ vi /etc/docker/daemon.json
```
 
```
    {
    "exec-opts": ["native.cgroupdriver=systemd"]
    }
```

```
$ sudo tee /etc/yum.repos.d/kubernetes.repo <<EOF
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

```
$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
$ sudo systemctl enable kubelet && sudo systemctl start kubelet

$ yum install iproute-tc
$ yum install ipvsadm iproute-tc -y
 
$ sudo modprobe br_netfilter
$ sudo sysctl --system

$ sudo mkdir /etc/containerd
$ sudo containerd config default > /etc/containerd/config.toml

$ sudo systemctl restart containerd
$ sudo systemctl restart kubelet
```

## Only for Master-Node

```
$ kubeadm config images pull
 
$ kubeadm init --pod-network-cidr=192.168.0.0/16,2603:c021:4004:7208:e7fe:58b5:c0ea:93db/64 --service-cidr=10.96.0.0/16,2001:db8:42:1::/112
 
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

$ kubectl label nodes ip-172-31-3-181/hostnameof-workerpod kubernetes.io/role=worker-node

$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

$ sudo sysctl -w net.ipv6.conf.all.forwarding=1
```
**Install Calico networking and network policy for dual stack or IPv6 only** 

```
$ vi custom-resources.yaml # 
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
        cidr: 192.168.0.0/16   #change this ipv4 as per kubectl init cmd ips
        encapsulation: IPIP
        natOutgoing: Enabled
        nodeSelector: all()
      - blockSize: 122
        cidr: 2603:c021:4004:7208:e7fe:58b5:c0ea:93db/64  #change this ipv6 as per kubectl init cmd ips
        encapsulation: None
        natOutgoing: Enabled
        nodeSelector: all()
```
 
**This section configures the Calico API server. For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer**

```
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```
```
$ kubectl get pods -A 
```
