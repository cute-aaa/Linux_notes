Kubernetes 笔记

## 好东西

谷歌提供的在线kubernetes实验终端<https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/>

终端配套文档<https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/>

终端的文本不能复制，解决方法是在选中文本之后不要松开鼠标，用键盘alt+tab切换窗口，然后再右键选中的部分就出现复制选项了。





# 使用kubeadm管理

## 准备工作

以Ubuntu为例

### 安装docker

参考<https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html>

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```bash
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```

### 手动安装

由于 `apt` 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

```bash
$ sudo apt-get update

$ sudo apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

为了确认所下载软件包的合法性，需要添加软件源的 `GPG` 密钥。

```bash
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -


# 官方源
# $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

然后，我们需要向 `source.list` 中添加 Docker 软件源

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"


# 官方源
# $ sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"
```

>   以上命令会添加稳定版本的 Docker CE APT 镜像源，如果需要测试或每日构建版本的 Docker CE 请将 stable 改为 test 或者 nightly。

#### 安装 Docker CE

更新 apt 软件包缓存，并安装 `docker-ce`：

```bash
$ sudo apt-get update

$ sudo apt-get -y install docker-ce
```



### 使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套脚本安装：

```bash
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。

### 启动 Docker CE

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```



用途未知：

### 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```bash
$ sudo usermod -aG docker $USER
```

**修改docker cgroup驱动，与k8s一致**

# ！！！一定要先修改驱动再安kubeadm！！！！

kubeadm会通过docker决定kubelet使用的驱动，如果在kubelet被设为cgroupfs之后把docker设为systemd，那么kubelet会因为与docker驱动不一致而无法启动。详见https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node

```bashg
## 使用systemd
# 修改docker cgroup驱动：native.cgroupdriver=systemd
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],#只需要这一句就行了
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

systemctl restart docker  # 重启使配置生效
```

或更改k8s的驱动：

```
确保docker 的cgroup drive 和kubelet的cgroup drive一样：

docker info | grep -i cgroup

cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf


若显示不一样，则执行：
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload


### 别试了，kubeadm初始化时会失败，因为改了之后kubelet启动不了
## 使用cgroupfs
# 修改kubelet的配置，使用cgroupfs 参考https://www.cnblogs.com/hongdada/p/9771857.html
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl enable kubelet #开机启动
systemctl restart kubelet
```

![img](https://upload-images.jianshu.io/upload_images/938819-9d584bf92c8e70b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



### 安装kube全家桶

参考<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

**每个节点都需要安装这三个东西**

You will install these packages on all of your machines:

-   `kubeadm`: the command to bootstrap the cluster.	集群启动命令
-   `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.    这个组件在所有集群的机器中运行，大概执行启动pod和容器的操作
-   `kubectl`: the command line util to talk to your cluster.    与集群交互的命令行集合

```bash
#阿里源Centos
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum -y install kubelet kubeadm kubectl
# Ubuntu
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

#官方源
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

#### kubectl自动完成

```shell
apt-get -y install bash-completion || yum -y install bash-completion
source /usr/share/bash-completion/bash_completion
echo "source /usr/share/bash-completion/bash_completion" >> ~/.bashrc
kubectl completion bash >/etc/bash_completion.d/kubectl
kubeadm completion bash >/etc/bash_completion.d/kubeadm
source /etc/bash_completion.d/kubectl
source /etc/bash_completion.d/kubeadm
```



#### 关闭swap分区*

```bash
swapoff -a && sysctl -w vm.swappiness=0  # 关闭swap
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab  # 取消开机挂载swap
```

#### 关闭SElinux []

```bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

#### 开机启动 []

```bash
systemctl enable --now kubelet  # 开机启动kubelet
```

####  centos7用户还需要设置路由：[]

```bash
yum install -y bridge-utils.x86_64
modprobe  br_netfilter  # 加载br_netfilter模块，使用lsmod查看开启的模块
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

#### 关闭防火墙

```bash
ufw disable || systemctl disable --now firewalld  # 关闭防火墙
sysctl --system  # 重新加载所有配置文件
```



### 拉取镜像

所需镜像

![1564017238857](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1564017238857.png)

这里不拉，一会执行`kubeadm init` 时也会自己拉

```sh
# Master端：
kubeadm config images pull # 拉取集群所需镜像，这个需要翻墙

# --- 不能翻墙可以尝试以下办法 ---
kubeadm config images list # 列出所需镜像
#(不是一定是下面的,根据实际情况来)
# 根据所需镜像名字先拉取国内资源
docker pull mirrorgooglecontainers/kube-apiserver:v1.14.1
docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.1
docker pull mirrorgooglecontainers/kube-scheduler:v1.14.1
docker pull mirrorgooglecontainers/kube-proxy:v1.14.1
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker pull coredns/coredns:1.3.1  # 这个在mirrorgooglecontainers中没有

# 修改镜像tag
docker tag mirrorgooglecontainers/kube-apiserver:v1.14.1 k8s.gcr.io/kube-apiserver:v1.14.1
docker tag mirrorgooglecontainers/kube-controller-manager:v1.14.1 k8s.gcr.io/kube-controller-manager:v1.14.1
docker tag mirrorgooglecontainers/kube-scheduler:v1.14.1 k8s.gcr.io/kube-scheduler:v1.14.1
docker tag mirrorgooglecontainers/kube-proxy:v1.14.1 k8s.gcr.io/kube-proxy:v1.14.1
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1


# 把所需的镜像下载好，init的时候就不会再拉镜像，由于无法连接google镜像库导致出错

# 删除原来的镜像
docker rmi mirrorgooglecontainers/kube-apiserver:v1.14.1
docker rmi mirrorgooglecontainers/kube-controller-manager:v1.14.1
docker rmi mirrorgooglecontainers/kube-scheduler:v1.14.1
docker rmi mirrorgooglecontainers/kube-proxy:v1.14.1
docker rmi mirrorgooglecontainers/pause:3.1
docker rmi mirrorgooglecontainers/etcd:3.3.10
docker rmi coredns/coredns:1.3.1

# --- 不能翻墙可以尝试使用 ---

# Node端：
# 根据所需镜像名字先拉取国内资源
docker pull mirrorgooglecontainers/kube-proxy:v1.14.1
docker pull mirrorgooglecontainers/pause:3.1


# 修改镜像tag
docker tag mirrorgooglecontainers/kube-proxy:v1.14.1 k8s.gcr.io/kube-proxy:v1.14.1
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1

# 删除原来的镜像
docker rmi mirrorgooglecontainers/kube-proxy:v1.14.1
docker rmi mirrorgooglecontainers/pause:3.1
# 不加载镜像node节点不能


# 制作镜像
mkdir images
cd images
docker save k8s.gcr.io/kube-proxy:v1.15.1 > k8s.gcr.io#kube-proxy.tar
docker save k8s.gcr.io/kube-apiserver:v1.15.1 > k8s.gcr.io#kube-apiserver.tar
docker save k8s.gcr.io/kube-scheduler:v1.15.1 > k8s.gcr.io#kube-scheduler.tar
docker save k8s.gcr.io/kube-controller-manager:v1.15.1 > k8s.gcr.io#kube-controller-manager.tar
docker save k8s.gcr.io/coredns:1.3.1 > k8s.gcr.io#coredns.tar
docker save k8s.gcr.io/etcd:3.3.10 > k8s.gcr.io#etcd.tar
docker save k8s.gcr.io/pause:3.1 > k8s.gcr.io#pause.tar
docker save gcr.io/google_containers/etcd-amd64:3.0.17 > gcr.io#google_containers#etcd-amd64.tar

mkdir dashboard
cd dashboard/
docker pull k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta2/aio/deploy/recommended.yaml
docker save k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1 > k8s.gcr.io#kubernetes-dashboard-amd64.tar
docker save kubernetesui/dashboard:v2.0.0-beta2 > kubernetesui#dashboard.tar
docker save kubernetesui/metrics-scraper:v1.0.1 > kubernetesui#metrics-scraper.tar
cd ..

cd add-on/

mkdir calico
cd calico/
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
docker save calico/node:v3.8.0 > calico#node.tar
docker save calico/cno:v3.8.0 > calico#cno.tar
docker save calico/cni:v3.8.0 > calico#cni.tar
docker save calico/pod2daemon-flexvol:v3.8.0 > calico#pod2daemon-flexvol.tar
cd ..

mkdir canal
cd canal/
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/canal.yaml
#这个没有镜像
cd ..

mkdir cilium
cd cilium/
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.5/examples/kubernetes/1.14/cilium.yaml
docker save cilium/cilium-etcd-operator:v2.0.6 > cilium#cilium-etcd-operator.tar
docker save cilium/cilium-init:2019-04-05 > cilium#cilium-init.tar
docker save cilium/cilium:v1.5.5 > cilium#cilium.tar
cd ..

mkdir flannel && cd flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
docker save quay.io/coreos/flannel:v0.11.0 > quay.io#coreos#flannel.tar
cd ..

mkdir romana && cd romana/
kubectl apply -f https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml
docker save quay.io/romana/daemon:v2.0.2 > quay.io#romana#daemon.tar
docker save quay.io/romana/listener:v2.0.2 > quay.io#romana#listener.tar
docker save quay.io/romana/agent:v2.0.2 > quay.io#romana#agent.tar
cd ..

mkdir weavenet && cd weavenet
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
docker save weaveworks/weave-kube:2.5.2 > weaveworks#weave-kube.tar
docker save weaveworks/weave-npc:2.5.2 > weaveworks#weave-npc.tar
cd ..
```



## 创建集群

参考

博客

<https://juejin.im/post/5cb7dde9f265da034d2a0dba#heading-1>

安装全家桶

<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

kubectl、kubeadm自动补全

<https://kubernetes.io/docs/tasks/tools/install-kubectl/>

创建集群

<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

初始化步骤：初始化、指定配置文件、应用网络附件

加入集群：拉取镜像，检查docker、kubelet状态，加入

### 初始化

```bash
ipaddrs=($(hostname -I))
kubeadm init --apiserver-advertise-address ${ipaddrs[0]} --pod-network-cidr 10.244.0.0/16
# --kubernetes-version 1.14.1
# --apiserver-advertise-address 指定与其它节点通信的接口
# --pod-network-cidr 指定pod网络子网，使用fannel网络必须使用这个CIDR
# 其他cidr见https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes
```

**输出信息最好保存一下**，最后输出的kubeadm join.....可以直接在其他节点上运行并加入此节点。

#### 选择配置文件 & 使kubectl对非root用户可用

（不指定配置文件会报错：The connection to the server localhost:8080 was refused - did you specify the right host or port?）

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

如果你是root用户，可以执行：

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### 

如果觉得不好，可以重置：

## Tear down拆除

To undo what kubeadm did, you should first [drain the node](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#drain) and make sure that the node is empty before shutting it down.

撤销kubeadm，你需要先删除所有节点，确保节点在关闭时是空的

Talking to the control-plane node with the appropriate credentials, run:

使用适当的凭证与控制节点通信

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

Then, on the node being removed, reset all kubeadm installed state:

然后，在清理后的节点上，重置kubeadm的安装状态

```bash
kubeadm reset
```

The reset process does not reset or clean up iptables rules or IPVS tables. If you wish to reset iptables, you must do so manually:

重置进程不会重置和清理iptables规则和IPVS表，如果你想重置iptables，需要手动执行：

```bash
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

If you want to reset the IPVS tables, you must run the following command:

重置IPVS表：

```bash
ipvsadm -C
```

If you wish to start over simply run `kubeadm init` or `kubeadm join` with the appropriate arguments.

More options and information about the [`kubeadm reset command`](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-reset/).



### 节点配置

#### 将master作为工作节点

默认情况下，处于安全考虑，master不会作为工作节点。可以手动将其设为工作节点

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

With output looking something like:	输出大致是：

```bash
node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

这将会在所有节点上删除 `node-role.kubernetes.io/master` ，这里面包含了控制面板节点。这将导致调度器能够在任何节点上对pod进行调度。

### 令牌

令牌用于控制节点和加入节点之间相互的身份认证。任何拥有令牌的人都可以向集群添加经过身份认证的节点。

令牌可以使用`kubeadm token` 命令进行创建、列出、删除。See the [kubeadm reference guide](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/).

### 安装一个pod网络附加组件

>   **Caution:** This section contains important information about installation and deployment order. Read it carefully before proceeding.

必须安装一个pod网络附件来使pod之间可以通信。

**不同的附件有不同的cidr配置，注意！**

网络必须在应用之前部署。同样的，在网络安装完成之前CoreDNS不会启动。在安装附件之前，可以看到kube-dns是阻塞状态

![img](https://upload-images.jianshu.io/upload_images/6709127-08b9475feef3fa8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/727/format/webp)

kubeadm只支持基于网络的容器网络接口（CNI），（并且不支持kubenet）

有几个项目使用CNI提供Kubernetes pod网络，其中一些还支持网络策略[Network Policy](https://kubernetes.io/docs/concepts/services-networking/networkpolicies/)。可以使用的网络附加组件的完整列表参阅 [附加组件页面](https://kubernetes.io/docs/concepts/cluster-administration/addons/) 。在CNI v0.6.0版本支持添加IPV6。CNI bridge和local-ipam是Kubernetes 1.9版本中唯一支持的IPv6网络插件。

注意，kubeadm默认设置了一个更安全的集群，并强制使用[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)（基于角色的访问控制）。确保您的网络清单支持RBAC。

注意，pod网络绝对不能与任何主机网络重叠，否则会引发问题。

如果您发现您的网络插件首选的Pod网络与您的一些主机网络之间存在冲突，您应该考虑一个合适的CIDR替换，在kubeadm init with——Pod -network- CIDR期间使用它，来替换你的网络插件的yaml

安装网络附件：

```bash
kubectl apply -f <add-on.yaml>
```

附件只能安装一个，这里使用Canal，只在amd64上有效。

```shell
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/canal.yaml
```

更多附件参考<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes>

查看附件是否安装成功（未测试）

```bash
kubectl get pods -n kube-system --selector=k8s-app=cilium
```



使用 `kubectl get pods --all-namespaces` 来确认CoreDNS已经运行。

没有运行的话，查看 [troubleshooting docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/).



### 加入集群

使用刚才master初始化时输出的命令：

```bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

加入成功输出大致是：

```console
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

如果要指定IPV6，需要用方括号括起来：`[fd00::101]:2073`.

如果没有令牌（token），可以使用一下命令在控制节点中获取：

```bash
kubeadm token list
```

输出大致是：

![1563953223791](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1563953223791.png)

```console
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```

默认情况下，令牌在24小时后过期。如果在当前令牌过期后将节点连接到集群，则可以在控制平面节点上运行以下命令创建一个新令牌:

```bash
kubeadm token create
```

The output is similar to this:

```console
5didvk.d09sbcov8ph2amjw
```

如果没有 `--discovery-token-ca-cert-hash` ，可以在控制节点中执行：

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

输出大致是：

```console
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```



再次运行 `kubectl get nodes` ，就能看到新成员了。



### 从控制节点之外的机器上控制集群

将管理员kubeconfig文件从控制平面节点复制到您的工作站：

```bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

上面的示例假设为root启用了SSH访问。如果不是这样，您可以复制admin.conf文件，让其他用户访问，然后使用该用户执行scp。

但是，admin.conf文件为用户提供了集群上的超级用户特权。这个文件应该谨慎使用。对于普通用户，建议生成一个具有白名单特权的惟一凭据。你可以用 `kubeadm alpha kubeconfig user --client-name <CN>`  命令，

该命令将输出一个KubeConfig文件到STDOUT，您应该将该文件保存到一个文件中并分发给用户。然后，通过 `kubectl create (cluster)rolebinding` 命令来实现白名单特权。



### 将API服务器代理到本地主机

如果你想从集群外部连接到API服务器，你可以使用`kubectl proxy`:

```bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

现在可以在本地访问API服务器： `http://localhost:8001/api/v1`



### Web UI （Dashboard）

参考

发挥作用的

<https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/>

创建token

<https://github.com/kubernetes/dashboard/wiki/Creating-sample-user>

开始

<https://github.com/kubernetes/dashboard/>

发行版本

<https://github.com/kubernetes/dashboard/releases>

推荐设置

<https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup>

没用到

<https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above>

dashboard不是自带的，执行一下命令来导入它

```bash
kubectl delete ns kubernetes-dashboard #删除旧版本
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
```

2.0.0-beta2起支持k8s的1.15版本

执行

```bash
kubectl proxy
```

访问<http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/>

会有一个验证身份的页面，可以输入

```bash
kubeadm token list
# 如果显示的token已过期，可以创建一个
kubeadm token create

#或者这里的token，具体看https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
kubectl -n kube-system describe secret

```

只能在执行命令的机器上访问，更多参考 `kubectl proxy --help`







## 使用Minikube管理

参考

官方文档<https://kubernetes.io/docs/tasks/tools/install-minikube/>

个人博客<https://blog.csdn.net/qq_26819733/article/details/83591891>

云栖社区<https://yq.aliyun.com/articles/221687/>



阿里minikube<https://github.com/AliyunContainerService/minikube>



### 准备

-   安装kubectl（1）  <https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux>
-   安装所有（2） <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node>

```bash
方法（1）
root@sure:~/下载# curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
root@sure:~/下载# chmod +x ./kubectl
root@sure:~/下载# sudo mv ./kubectl /usr/local/bin/kubectl
#此时由于服务端的kubernetes没有开启，所以会提示访问被拒绝
root@sure:~/下载# kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?

方法（2）
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
apt-get update
apt-get install -y kubelet kubeadm kubectl
# 标记软件包使其不自动更新
apt-mark hold kubelet kubeadm kubectl
```



##### 启用kubectl自动完成功能

您现在需要确保kubectl完成脚本在所有shell会话中获得。有两种方法可以做到这一点：

-   获取`~/.bashrc`文件中的完成脚本：

    ```shell
    echo 'source <(kubectl completion bash)' >>~/.bashrc
    ```

或

-   将完成脚本添加到`/etc/bash_completion.d`目录：

    ```shell
    kubectl completion bash >/etc/bash_completion.d/kubectl
    ```

>   **注意：** bash-completion来源于所有完成脚本`/etc/bash_completion.d`。

两种方法都是等价的。重新加载shell后，kubectl自动完成应该正常工作。



Minikube在不同操作系统上支持不同的驱动

-   macOS
    -   [xhyve driver](https://yq.aliyun.com/go/articleRenderRedirect?url=https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#xhyve-driver), [VirtualBox](https://yq.aliyun.com/go/articleRenderRedirect?url=https://www.virtualbox.org/wiki/Downloads) 或 [VMware Fusion](https://yq.aliyun.com/go/articleRenderRedirect?url=https://www.vmware.com/products/fusion)
-   Linux
    -   [VirtualBox](https://yq.aliyun.com/go/articleRenderRedirect?url=https://www.virtualbox.org/wiki/Downloads) 或 [KVM](https://yq.aliyun.com/go/articleRenderRedirect?url=https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm-driver)
    -   **NOTE:** Minikube 也支持 `--vm-driver=none` 选项来在本机运行 Kubernetes 组件，这时候需要本机安装了 Docker。在使用 0.27版本之前的 none 驱动时，在执行 `minikube delete` 命令时，会移除 /data 目录，请注意，[问题说明](https://yq.aliyun.com/go/articleRenderRedirect?url=https://github.com/kubernetes/minikube/issues/2794)；另外 none 驱动会运行一个不安全的API Server，会导致安全隐患，不建议在个人工作环境安装。
-   Windows
    -   [VirtualBox](https://yq.aliyun.com/go/articleRenderRedirect?url=https://www.virtualbox.org/wiki/Downloads) 或 [Hyper-V](https://yq.aliyun.com/go/articleRenderRedirect?url=https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperV-driver) - 请参考下文

注：

-   由于minikube复用了docker-machine，在其软件包中已经支持了相应的VirtualBox, VMware Fusion驱动
-   VT-x/AMD-v 虚拟化必须在 BIOS 中开启
-   在Windows环境下，如果开启了Hyper-V，不支持VirtualBox方式



### 安装minikube

阿里提供了最新的Minikube修改版的文件，可以直接下载使用

**Mac OSX**

```bash
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

**Linux**

```bash
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

**Windows**

下载 [minikube-windows-amd64.exe](https://yq.aliyun.com/go/articleRenderRedirect?url=http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-windows-amd64.exe) 文件，并重命名为 `minikube.exe`

#### 自己构建

也可以从Github上获取相应的项目自行构建。

注：需要本地已经安装配置好 Golang 开发环境和Docker引擎

```bash
git clone https://github.com/AliyunContainerService/minikube
cd minikube
git checkout aliyun-v1.2.0
make
sudo cp out/minikube /usr/local/bin/
```

#### 另一种下载安装方式

使用docker仓库，参考个人博客<https://blog.csdn.net/u010652906/article/details/86075437>

#### 不使用阿里的后果

```bash
root@sure:~/下载# minikube start
😄  minikube v1.2.0 on linux (amd64)
⚠️  Please don't run minikube as root or with 'sudo' privileges. It isn't necessary.
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🏃  Re-using the currently running virtualbox VM for "minikube" ...
⌛  Waiting for SSH access ...
🐳  Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
E0703 12:04:44.516115   22020 start.go:403] Error caching images:  Caching images for kubeadm: caching images: caching image /root/.minikube/cache/images/k8s.gcr.io/coredns_1.3.1: fetching remote image: Get https://k8s.gcr.io/v2/: dial tcp 74.125.203.82:443: connect: connection refused
❌  Unable to load cached images: loading cached images: loading image /root/.minikube/cache/images/gcr.io/k8s-minikube/storage-provisioner_v1.8.1: stat /root/.minikube/cache/images/gcr.io/k8s-minikube/storage-provisioner_v1.8.1: no such file or directory
💾  Downloading kubelet v1.15.0
💾  Downloading kubeadm v1.15.0
🔄  Relaunching Kubernetes v1.15.0 using kubeadm ... 

💣  Error restarting cluster: waiting for apiserver: timed out waiting for the condition

😿  Sorry that minikube crashed. If this was unexpected, we would love to hear from you:
👉  https://github.com/kubernetes/minikube/issues/new
```



### 查看是否安装完成

```bash
#查看版本
root@sure:~/下载# minikube version
minikube version: v1.2.0

#启动
root@sure:~/下载# minikube start --registry-mirror=https://registry.docker-cn.com
😄  minikube v1.2.0 on linux (amd64)
⚠️  Please don't run minikube as root or with 'sudo' privileges. It isn't necessary.
✅  using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
🐳  Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
🚜  Pulling images ...
🚀  Launching Kubernetes ... 
⌛  Verifying: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"

#打开Kubernetes控制台
root@sure:~/下载# minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:36807/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
#停在这里之后打开上面的连接就能看到控制台了

#启动之后再去查看kubectl version
root@sure:~/下载# kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```



## 运行

关系：

Pod总是在**节点**上运行。Node是Kubernetes中的工作计算机，可以是虚拟机或物理计算机，具体取决于集群。每个节点由Master管理。节点可以有多个pod，Kubernetes master会自动处理在群集中的节点上调度pod。Master的自动调度考虑了每个节点上的可用资源。

每个Kubernetes节点至少运行：

-   Kubelet，负责Kubernetes Master和Node之间通信的过程; 它管理Pod和机器上运行的容器。
-   容器运行时（如Docker，rkt）负责从注册表中提取容器映像，解压缩容器以及运行应用程序。

*如果容器紧密耦合并且需要共享磁盘等资源，那么容器只应在一个Pod中一起安排*



集群（cluster)

节点（Nodes）（一个虚拟机或物理机）（可包含多个pod）

​	pod：具有自己的 IP地址，开启服务后公开

​		数据卷（volume）

​		容器app（containerized app）

![img](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

服务（通过一组pod路由流量）（服务是抽象的，使用[标签和选择器](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels)匹配一组Pod）

​	pod（依赖pod之间的发现和路由由Kubernetes Service处理）

![img](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)

![img](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)



标签是一个允许对Kubernetes中的对象进行逻辑操作的分组原语。标签是附加到对象的键/值对，可以以多种方式使用：

-   指定用于开发，测试和生产的对象
-   嵌入版本标签
-   使用标记对对象进行分类

![img](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)

### 创建部署

1.  现在，您可以使用kubectl与群集进行交互。有关更多信息，请参阅[与群集交互](https://kubernetes.io/docs/setup/learning-environment/minikube/#interacting-with-your-cluster)。

    让我们使用名为的现有映像创建Kubernetes部署`echoserver`，该映像是一个简单的HTTP服务器，并使用在端口8080上公开它`--port`。

    ```shell
    # 这个镜像挂掉了
    kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
    # 可以使用这个
    kubectl run hello-minikube --image=training/webapp --port=5000
    ```

    输出类似于：

    ```
    deployment.apps/hello-minikube created
    ```

2.  要访问`hello-minikube`部署，请将其公开为服务：

    ```shell
    kubectl expose deployment hello-minikube --type=NodePort
    ```

    该选项`--type=NodePort`指定服务的类型。

    -   *ClusterIP*（默认） - 在群集中的内部IP上公开服务。此类型使服务只能从群集中访问。
    -   *NodePort* - 使用NAT在集群中每个选定节点的同一端口上公开服务。使用可从群集外部访问服务`<NodeIP>:<NodePort>`。ClusterIP的超集。
    -   *LoadBalancer* - 在当前云中创建外部负载均衡器（如果支持），并为服务分配固定的外部IP。NodePort的超集。
    -   *ExternalName* - `externalName`通过返回带有名称的CNAME记录，使用任意名称（在规范中指定）公开服务。没有代理使用。此类型需要v1.7或更高版本`kube-dns`。

    输出类似于：

    ```
    service/hello-minikube exposed
    ```

3.  该`hello-minikube` pod已经推出，但你必须要等到pod部署完成之后通过公开的服务访问它。

    检查Pod是否已启动并运行：

    ```shell
    kubectl get pod
    ```

    如果输出显示`STATUS`为`ContainerCreating`，则仍在创建Pod：

    ```
    NAME                              READY     STATUS              RESTARTS   AGE
    hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
    ```

    如果输出显示`STATUS`为`Running`，则Pod现在已启动并正在运行：

    ```
    NAME                              READY     STATUS    RESTARTS   AGE
    hello-minikube-3383150820-vctvh   1/1       Running   0          13s
    ```

4.  获取公开的服务的URL以查看服务详细信息：

    ```shell
    minikube service hello-minikube --url
    ```

5.  要查看本地群集的详细信息，请在浏览器中复制并粘贴您作为输出的URL。

    输出类似于：

    ```bash
    # 官方镜像：
    Hostname: hello-minikube-7c77b68cff-8wdzq
    
    Pod Information:
        -no pod information available-
    
    Server values:
        server_version=nginx: 1.13.3 - lua: 10008
    
    Request Information:
        client_address=172.17.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://192.168.99.100:8080/
    
    Request Headers:
        accept=*/*
        host=192.168.99.100:30674
        user-agent=curl/7.47.0
    
    Request Body:
        -no body in request-
    ```

    ```bash
    # training/webapp
    Hello World!
    ```

    如果您不再希望运行服务和群集，则可以删除它们。

6.  删除`hello-minikube`服务：

    ```shell
    kubectl delete services hello-minikube
    ```

    输出类似于：

    ```
    service "hello-minikube" deleted
    ```

7.  删除`hello-minikube`部署：

    ```shell
    kubectl delete deployment hello-minikube
    ```

    输出类似于：

    ```
    deployment.extensions "hello-minikube" deleted
    ```

8.  停止本地Minikube群集：

    ```shell
    minikube stop
    ```

    输出类似于：

    ```
    Stopping "minikube"...
    "minikube" stopped.
    ```

    有关更多信息，请参阅[停止群集](https://kubernetes.io/docs/setup/learning-environment/minikube/#stopping-a-cluster)。

9.  删除本地Minikube群集：

    ```shell
    minikube delete
    ```

    输出类似于：

    ```
    Deleting "minikube" ...
    The "minikube" cluster has been deleted.
    ```

    有关更多信息，请参阅[删除群集](https://kubernetes.io/docs/setup/learning-environment/minikube/#deleting-a-cluster)。




## 远程访问

kubectl命令可以创建一个代理，将通信转发到集群范围的私有网络。按control-C可以终止代理，并且在运行时不会显示任何输出。

建议打开第二个终端窗口来运行代理。

```bash
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy
```

现在，我们的主机(在线终端)和Kubernetes集群之间有了连接。代理允许从这些终端直接访问API。

您可以在http://localhost:8001上看到所有通过代理端点承载的api。例如，我们可以使用curl命令直接通过API查询版本:

```bash
curl http://localhost:8001/version
```

输错了会打印帮助信息

```
$ curl http://localhost:8001/a
{
  "paths": [
    "/apis",
    "/apis/",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/healthz",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/metrics",
    "/openapi/v2",
    "/version"
  ]
}
```

API服务器将根据pod名称自动为每个pod创建端点，该端点也可以通过代理访问。

运行也会出现容器的地址，可以访问到服务主页。权限未知。

```bash
root@sure:~# minikube service hello-minikube --url
http://192.168.99.106:30370
```

首先我们需要获得Pod名称，我们将存储在环境变量POD_NAME中:

```bash
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') echo Name of the Pod: $POD_NAME
```

现在我们可以对在那个pod中运行的应用程序发出HTTP请求:

```bash
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
```

url是到Pod API的路由。

注意:检查终端顶部。代理在一个新选项卡(终端2)中运行，最近的命令在原始选项卡(终端1)中执行。

查看日志

```bash
kubectl logs POD_NAME
```

一旦Pod启动并运行，我们就可以直接在容器上执行命令。为此，我们使用exec命令并使用Pod的名称作为参数。让我们列出环境变量:

```bash
kubectl exec $POD_NAME env
```

同样值得一提的是，容器本身的名称可以省略，因为我们在Pod中只有一个容器。

接下来让我们在Pod的容器中启动一个bash会话:

```bash
kubectl exec -ti $POD_NAME bash
```

现在在运行NodeJS应用程序的容器上有一个打开的控制台。app的源代码在server.js文件中:（就是测试一下输入输出）

```bash
cat server.js
```

您可以通过运行curl命令来检查应用程序是否已经启动:

```bash
curl localhost:8080
```



## 绑定docker

在Mac / Linux主机上使用Docker守护程序，请使用`docker-env command`shell中的：

```shell
eval $(minikube docker-env)
```

您现在可以在主机Mac / Linux机器的命令行中使用Docker与Minikube VM内的Docker守护程序进行通信：

```shell
docker ps
```



## 扩展服务



![img](https://d33wubrfki0l68.cloudfront.net/043eb67914e9474e30a303553d5a4c6c7301f378/0d8f6/docs/tutorials/kubernetes-basics/public/images/module_05_scaling1.svg)

![img](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/b5f56/docs/tutorials/kubernetes-basics/public/images/module_05_scaling2.svg)

服务扩展是通过更改部署中的副本数量来实现的。扩展部署将确保创建新的pod并将其调度到具有可用资源的节点。Kubernetes还支持pod的自动缩放，但这超出了本教程的范围。还可以将其扩展到零，它将终止指定部署的所有pod。

运行应用程序的多个实例将需要一种方法来将流量分配给所有实例。服务有一个集成的负载平衡器，它将把网络流量分配给公开部署的所有吊舱。服务将使用端点连续监视正在运行的pod，以确保流量只发送到可用的Pods。一旦应用程序运行了多个实例，就可以进行滚动更新，而不需要停机。

```bash
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```



## 更新应用

摘要:通过使用新实例增量更新Pods实例，更新应用程序滚动更新允许部署“零停机时间更新”。

滚动更新允许使用新实例逐渐地（incrementally）更新Pods实例，从而实现部署更新，停机时间为零。新pod将安排在具有可用资源的节点上。在前面的模块中，我们将应用程序扩展为运行多个实例。这要求在不影响应用程序可用性的情况下执行更新。默认情况下，更新期间不可用的最大pod数和可以创建的新pod数都是1。这两个选项都可以配置为(pod的)数量或百分比。在Kubernetes中，更新是经过版本控制的，任何部署更新都可以恢复到以前的(稳定的)版本。



![img](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/fa906/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates1.svg)

![img](https://d33wubrfki0l68.cloudfront.net/678bcc3281bfcc588e87c73ffdc73c7a8380aca9/703a2/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates2.svg)

![img](https://d33wubrfki0l68.cloudfront.net/9b57c000ea41aca21842da9e1d596cf22f1b9561/91786/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates3.svg)

![img](https://d33wubrfki0l68.cloudfront.net/6d8bc1ebb4dc67051242bc828d3ae849dbeedb93/fbfa8/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates4.svg)

如果部署是公开（exposed）的，则服务将仅在更新期间对可用的pod进行负载平衡。

可用pod是指用户可用的应用程序实例。滚动更新允许以下操作:

-   促进（Promote）应用程序从一个环境到另一个环境(通过容器映像更新).
-   回滚到以前的版本。
-   持续集成和持续交付应用程序，零停机时间。

#### 更新镜像

查看当前pod

```bash
kubectl get pods
```

要查看应用程序的当前镜像版本，请对Pods运行describe命令(查看镜像字段):

```bash
kubectl describe pods
```

要将应用程序的映像更新到版本2，请使用set image命令，然后是部署名称和新的映像版本:

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```

该命令通知部署为应用程序使用不同的映像，并启动滚动更新。检查新pod的状态，查看以get Pods命令结束的旧pod:

```bash
kubectl get pods
```



#### 验证更新

首先，让我们检查应用程序是否正在运行。为了找出公开的IP和端口，我们可以使用描述服务:

```bash
kubectl describe services/kubernetes-bootcamp
```

创建一个名为NODE_PORT的环境变量，该变量具有指定的节点端口的值:

```bash
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') echo NODE_PORT=$NODE_PORT
```

接下来，我们将对暴露的IP和端口做一个“curl”:

```bash
curl $(minikube ip):$NODE_PORT
```

每个请求都命中一个不同的Pod，我们看到所有的Pod都运行最新版本(v2)。

更新也可以通过运行rollout status命令来确认:

```bash
kubectl rollout status deployments/kubernetes-bootcamp
```

要查看应用程序的当前图像版本，请对pod运行描述命令:

```bash
kubectl describe pods
```

现在在运行应用程序的版本2(查看image字段)



#### 回滚

让我们执行另一个更新，并部署标记为v10的图像:

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
```

使用get deployment查看部署状态:

```bash
kubectl get deployments
```

似乎哪里不对… 我们没有得到期望的可用pods数。再次产看pod列表：

```bash
kubectl get pods
```

 `describe` 命令或许描述的更清晰些

```bash
kubectl describe pods
```

存储库中没有名为 `v10` 的镜像. 让我们回滚到上一个工作版本。使用 `rolloutundo` 命令:

```bash
kubectl rollout undo deployments/kubernetes-bootcamp
```

 `rollout` 命令将部署恢复到先前的已知状态(image的 v2版本)。更新是经过版本控制的，您可以恢复到任何先前已知的部署状态。再次列出pods :

```bash
kubectl get pods
```

四个pod在运行，再次查看部署在上面的镜像：

```bash
kubectl describe pods
```

可以看到部署正在使用稳定版本（v2），回滚成功。



更多参照<https://kubernetes.io/docs/setup/learning-environment/minikube/>




