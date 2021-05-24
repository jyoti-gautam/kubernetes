# kubernetes
find here about kubernetes


Set up load balancer node
Install Haproxy
yum update && yum install -y haproxy	

Configure haproxy
Append the below lines to /etc/haproxy/haproxy.cfg

frontend kubernetes-frontend
    bind 172.16.16.100:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server kmaster1 172.16.16.101:6443 check fall 3 rise 2
    server kmaster2 172.16.16.102:6443 check fall 3 rise 2

Restart haproxy service
systemctl restart haproxy
On all kubernetes nodes (kmaster1, kmaster2, kworker1)
Disable Firewall
systemctl stop firewalld

systemctl disable firewalld
Disable swap
swapoff -a; sed -i '/swap/d' /etc/fstab
Update sysctl settings for Kubernetes networking
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl –system

Install docker engine
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

Kubernetes Setup
Add yum repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
Install Kubernetes components
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

On any one of the Kubernetes master node (Eg: kmaster1)
Initialize Kubernetes Cluster
kubeadm init --control-plane-endpoint="172.16.16.100:6443" --upload-certs --apiserver-advertise-address=172.16.16.101 --pod-network-cidr=192.168.0.0/16


Copy the commands to join other master nodes and worker nodes.
Deploy Calico network
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml


Join other nodes to the cluster (kmaster2 & kworker1)
Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.
IMPORTANT: You also need to pass --apiserver-advertise-address to the join command when you join the other master node.

Downloading kube config to your local machine
On your host machine
mkdir ~/.kube
scp root@172.16.16.101:/etc/kubernetes/admin.conf ~/.kube/config

Verifying the cluster
kubectl cluster-info
kubectl get nodes
kubectl get cs


