# 手把手搭建Kubernetes集群



## 1.环境准备

由这三台物理机组成，一个master节点，两个node节点，如下所示。

| 节点              | 主机IP(bond0.168) |
| ----------------- | ----------------- |
| k8s-node1(master) | 172.26.68.169     |
| k8s-node2(master) | 172.26.68.170     |
| k8s-node3(master) | 172.26.68.171     |

## 2.设置主机名

```sh
hostnamectl set-hostname k8s-node1
hostnamectl set-hostname k8s-node2
hostnamectl set-hostname k8s-node3
```



## 3.主机名与ip映射

在master上添加主机名和ip对应关系：

```
vi /etc/hosts
```



添加如下

```
172.26.68.169  k8s-node1

172.26.68.170  k8s-node2

172.26.68.171  k8s-node3
```



## 4.禁用swap分区

为了保证 kubelet 正常工作，必须禁用swap分区。

```
# 临时
swapoff -a

# 永久
vi /etc/fstab
注释掉了/etc/fstab配置文件中的/swapfile none swap defaults 00

使用 free -m 命令查看是否禁用成功
```



## 5.禁用selinux

1.使用 vim /etc/sysconfig/selinux。  

2.将SELINUX=enforcing改为SELINUX=disabled。  

3.修改完成后,重启计算机 reboot 或 init 6。

## 6.禁用防火墙

```sh
systemctl stop firewalld
systemctl disable firewalld
```



## 7.允许iptables检查桥接流量

为了让节点上的 iptables 能够正确地查看桥接流量，确保在sysctl 配置中将 net.bridge.bridge-nf-call-iptables 设置为 1

将桥接的IPv4流量传递到iptables的链(流量统计作用)：

```
cat<<EOF | sudotee/etc/modules-load.d/k8s.confbr_netfilterEOFcat<<EOF | sudotee/etc/sysctl.d/k8s.confnet.bridge.bridge-nf-call-ip6tables = 1net.bridge.bridge-nf-call-iptables = 1EOFsudosysctl --system
```

可能遇到的问题：

遇见提示是只读的文件系统，运行命令：mount-o remount rw /

## 8.时间同步

所有节点同步时间是必须的，如果节点时间不同步，会造成Etcd存储Kubernetes信息的键－值数据库同步数据不正常，也会造成证书出现问题。

```
yum install ntpdate -yntpdate time.windows.com
```



## 9.安装docker

```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repoyum -y install docker-ce-18.06.1.ce-3.el7systemctl enable docker && systemctl start dockerdocker --version

给 docker设置阿里源

vi /etc/docker/daemon.json

添加下面内容:

{
	"registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
```



## 10.添加yum源

```
vi /etc/yum.repos.d/kubernetes.repo

添加下面内容：

[kubernetes]

name=Kubernetes

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=0

repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg

https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```



## 11.安装 kubeadm，kubelet 和 kubectl

```
yum install -y kubelet-1.18.1-0 kubeadm-1.18.1-0 kubectl-1.18.1-0systemctl enable kubelet
```

##  12.部署节点

### 1 部署Master节点

1.1 初始化kubeadm

在172.26.68.169

上执行下面的命令：

```
kubeadm init \--apiserver-advertise-address=172.26.68.169 \--image-repository registry.aliyuncs.com/google_containers \--kubernetes-version v1.18.1 \--service-cidr=10.96.0.0/12 \--pod-network-cidr=10.244.0.0/16
```

执行结果如下：



### 2 kubectl工具

接下来使用kubectl工具，分别执行以下命令：

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

查看节点：

NAME       STATUS        ROLES        AGE        VERSION

k8s-node1  Ready           master        3m52s     v1.18.1

### 3 安装pod网络插件（CNI）

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 4添加k8s node节点

在执行kubeadm init后，会打印出添加节点的命令：

```
kubeadm join 172.26.68.169:6443 --token gr4xxxxxxxxxxrrvq --discovery-token-ca-cert-hash sha256:e9bcxef2ecxxxxxxxxxxxxxxx
```

若果没有记录下来可以使用以下命令查看:

```
kubeadm token create --print-join-command
```

如果出现如下问题，可以使用kubeadm reset命令重置：

[preflight] Running pre-flight checks        [WARNING Hostname]: hostname "k8s-node1" could not be reached        [WARNING Hostname]: hostname "k8s-node1": lookup k8s-node1 on172.26.68.169:53: no such host

error execution phase preflight: [preflight] Some fatal errors occurred:        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`

### 5查看节点

```
kubectl get nodes
```

卸载K8s及组件

```
kubeadm reset

yum erase -y kubelet kubectl kubeadm kubernetes-cni
```

