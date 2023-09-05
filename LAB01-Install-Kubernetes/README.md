# LAB01-Install-Kubernetes

## a. Nội dung lab

Hostname & Vai trò | Thông tin | IP
:---------:|:----------:|:---------:
 master | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 172.16.10.100
 worker1 | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 172.16.10.101
 worker2 | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 172.16.10.102

## b. Các cài đặt chung

* Update

```bash
apt update
apt upgrade -y
```

* Cài đặt & start & enable docker

```bash
apt install -y docker.io
systemctl enable docker --now
```

* Tắt swap

```bash
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
swapoff -a
mount -a
```

* Enable kernel modules and configure sysctl.

```bash
# Enable kernel modules
modprobe overlay
modprobe br_netfilter

# Add some settings to sysctl
tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sysctl --system
```

* Cài đặt kubernetes

```bash
# Thêm Kubernetes GPG key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

# Thêm Kubernetes repo
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

apt update

# Cài đặt Docker và Kubernetes
apt install -y docker-ce kubelet kubeadm kubectl
systemctl enable kubelet --now
```

## 1. Master

* Khởi tạo

```bash
kubeadm init --apiserver-advertise-address=172.16.10.100 --pod-network-cidr=192.168.0.0/16
```

* Sau khi quá trình cài đặt hoàn thành, hãy thiết lập kubectl:

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

* Cài đặt plugin mạng Calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

* Kiểm tra lại Cluster

```bash
kubectl get nodes
```

* Bạn sẽ thấy danh sách các node và trạng thái của chúng.

* Với đó, bạn đã cài đặt thành công một cluster Kubernetes với 1 node master và 2 node worker, sử dụng plugin mạng Calico.

## 2. worker1 & worker2

* Thêm các Node vào Cluster

Chạy lệnh này trên cả hai node worker (worker1 và worker2). Lệnh này sẽ được hiển thị ở cuối của quá trình kubeadm init ở node master:

```bash
kubeadm join 172.16.10.100:6443 --token YOUR-TOKEN --discovery-token-ca-cert-hash YOUR-HASH
```
