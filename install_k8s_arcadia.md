## 安装前的检查
1、确认操作系统版本
```root@k8s-node2:~# lsb_release -a
LSB Version:    core-11.1.0ubuntu4-noarch:security-11.1.0ubuntu4-noarch
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy
root@k8s-node2:~# uname -a
Linux k8s-node2 5.15.0-92-generic #102-Ubuntu SMP Wed Jan 10 09:33:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux`
```

2、apt update
3、修改 /etc/hosts
```
192.168.0.245  k8s-master1
192.168.0.247  k8s-node1
192.168.0.246  k8s-node2
192.168.0.241  k8s-node3-gpu1
```
4、暂时禁用交换分区
> sudo swapoff -a

5、确保每个节点上 MAC 地址和 product_uuid 的唯一性 
> 使用命令 ip link 或 ifconfig -a 来获取网络接口的 MAC 地址 ，可以使用 sudo cat /sys/class/dmi/id/product_uuid 命令对 product_uuid 校验 。一般来讲，硬件设备会拥有唯一的地址，但是有些虚拟机的地址可能会重复。 Kubernetes 使用这些值来唯一确定集群中的节点。 如果这些值在每个节点上不唯一，可能会导致安装失败。

6、关闭防火墙
```
ufw status
ufw disable
```
7、转发 IPv4 并让 iptables 看到桥接流量 
执行下述命令：
```
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter

$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

$ sudo sysctl --system
```


## 安装 kubeadm
1、安装 kubeadm、kubelet 和 kubectl


以下指令适用于 Kubernetes 1.29. 
1）更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包： 
```
sudo apt-get update 
# apt-transport-https 可能是一个虚拟包（dummy package）；如果是的话，你可以跳过安装这个包 
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
2）下载用于 Kubernetes 软件包仓库的公共签名密钥。所有仓库都使用相同的签名密钥，因此你可以忽略URL中的版本： 
```
# 如果 `/etc/apt/keyrings` 目录不存在，则应在 curl 命令之前创建它，请阅读下面的注释。
# sudo mkdir -p -m 755 /etc/apt/keyrings 
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
3）添加 Kubernetes apt 仓库。 
>请注意，此仓库仅包含适用于 Kubernetes 1.29 的软件包； 对于其他 Kubernetes 次要版本，则需要更改 URL 中的 Kubernetes 次要版本以匹配你所需的次要版本 （你还应该检查正在阅读的安装文档是否为你计划安装的 Kubernetes 版本的文档）。
```
# 此操作会覆盖 /etc/apt/sources.list.d/kubernetes.list 中现存的所有配置。 
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
 
2、更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本： 
```
sudo apt-get update 
sudo apt-get install -y kubelet kubeadm kubectl 
sudo apt-mark hold kubelet kubeadm kubectl
```

## 安装容器运行时 containerd

在最新的 kubernetes 中已经使用 containerd 作为默认的容器运行时，所以我们需要安装 containerd 。先安装依赖：
```
$ sudo apt-get install ca-certificates curl gnupg
```
下载用于 docker 软件包仓库的公共签名密钥：
```
$ sudo install -m 0755 -d /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
添加 docker 仓库：
```
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
更新 apt 包索引，并安装 containerd：
```
$ sudo apt-get update
$ sudo apt-get install containerd.io
```
创建 containerd的默认配置文件，保存到 /etc/containerd/config.toml：
```
$ containerd config default > /etc/containerd/config.toml
```
由于一些众所周知的原因，部分容器镜像很难拉取到，所以这里我们需要修改默认配置：
```
# sandbox_image修改为aliyun源
sandbox_image = "registry.aliyuncs.com/k8sxio/pause:3.8"
# 启动 systemd cgroup 驱动
SystemdCgroup = true
```
使配置生效：
```
systemctl daemon-reload
systemctl restart containerd.service
```
## 使用 kubeadm 创建集群

### 创建 master

1、这里我们需要提供 --image-repository的参数，让 kubeadm 去指定的镜像仓库拉取镜像，一般这一步执行失败的原因，都是无法拉取镜像。 
>这里的镜像仓库只会影响 api-server、coredns等镜像的拉取，像 pause容器的镜像由于是与容器运行时强关联的，所以必须是在 containerd 的配置里调整，具体可以参考刚才的对 containerd.toml的修改。
```
$ sudo kubeadm init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'
```
安装完毕后，根据提示，执行如下命令。这样 kubectl 才有权限访问该 kubernetes 集群：
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
安装成功后显示如下：
```
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.243:6443 --token wemz3u.u4xkcsk16ono53yd \
        --discovery-token-ca-cert-hash sha256:503769f71aa9faf5cc699d1aa36c445f2a938f562571cf43a7051e535ed2ccd6 
```

### 安装 worker

master 的 kubeadm 安装结束后，会输出一条 kubeadm join 的命令，直接拷贝后在 worker 执行即可：
```
kubeadm join 192.168.0.243:6443 --token wemz3u.u4xkcsk16ono53yd \
        --discovery-token-ca-cert-hash sha256:503769f71aa9faf5cc699d1aa36c445f2a938f562571cf43a7051e535ed2ccd6 
```
在 master 和 worker 都执行完 kubeadm 命令后，我们在 master 上就可以查看 node 的信息，可以看到目前 node1 & node2 都已经加入 kubernetes 集群，STATUS 都是 NotReady 的状态：

```
root@k8s-master1:~# kubectl get no -A -owide
NAME             STATUS     ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-master1      NotReady   control-plane   50m   v1.29.2   192.168.0.245   <none>        Ubuntu 22.04.3 LTS   5.15.0-92-generic   containerd://1.6.28
k8s-node1        NotReady   <none>          36s   v1.29.2   192.168.0.247   <none>        Ubuntu 22.04.3 LTS   5.15.0-92-generic   containerd://1.6.28
k8s-node2        NotReady   <none>          20s   v1.29.2   192.168.0.246   <none>        Ubuntu 22.04.3 LTS   5.15.0-92-generic   containerd://1.6.28
k8s-node3-gpu1   NotReady   <none>          40s   v1.29.2   192.168.0.241   <none>        Ubuntu 22.04.3 LTS   5.15.0-92-generic   containerd://1.6.28
```
同时我们可以看到 pod 的信息，kubernetes 的核心组件都已经在 Running 状态。其中 coredns 仍然处在 Pending 状态， 这是正常的：

```
root@k8s-master1:~# kubectl get po -A -owide
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE   READINESS GATES
kube-system   coredns-5f98f8d567-2gnxq              0/1     Pending   0          50m   <none>          <none>           <none>           <none>
kube-system   coredns-5f98f8d567-zk6q7              0/1     Pending   0          50m   <none>          <none>           <none>           <none>
kube-system   etcd-k8s-master1                      1/1     Running   0          50m   192.168.0.245   k8s-master1      <none>           <none>
kube-system   kube-apiserver-k8s-master1            1/1     Running   0          50m   192.168.0.245   k8s-master1      <none>           <none>
kube-system   kube-controller-manager-k8s-master1   1/1     Running   0          50m   192.168.0.245   k8s-master1      <none>           <none>
kube-system   kube-proxy-c4m4m                      1/1     Running   0          68s   192.168.0.246   k8s-node2        <none>           <none>
kube-system   kube-proxy-dsf5h                      1/1     Running   0          84s   192.168.0.247   k8s-node1        <none>           <none>
kube-system   kube-proxy-ftbx5                      1/1     Running   0          50m   192.168.0.245   k8s-master1      <none>           <none>
kube-system   kube-proxy-pkqhl                      1/1     Running   0          88s   192.168.0.241   k8s-node3-gpu1   <none>           <none>
kube-system   kube-scheduler-k8s-master1            1/1     Running   0          50m   192.168.0.245   k8s-master1      <none>           <none>
```
          
为了解决节点 NotReady 和 coredns Pending的问题，我们需要安装网络插件，可使用Calico、terway或者kube-ovn，由于我们的云主机使用了阿里云VPC，所以这里推荐使用阿里巴巴的 terway。

### 网络插件方案一：安装 terway 插件
1、将iptables的policy换成ACCEPT，`iptables -P FORWARD ACCEPT`。
2、检查节点上的 `"rp_filter"` 内核参数，并在每个节点上将其设置为"0"。
3、通过 `kubectl get cs` 验证集群安装完成

### 网络插件方案二：安装 Calico 插件
请参考 [Install Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

### 网络插件方案三：安装 kube-ovn 插件

我们到官网下载 kube-ovn 的安装脚本：
```
$ wget https://raw.githubusercontent.com/kubeovn/kube-ovn/release-1.12/dist/images/install.sh
```
执行安装即可（如果执行失败，可以从官网下载 cleanup.sh 的脚本执行回退，再重复安装几次，一般失败都是因为镜像拉取的问题。）：
```
$ bash install.sh
```
执行成功后可以看到 kube-ovn 的 logo，之后就可以重新查看节点和 pod 的信息：
```
$ kubectl get node
NAME    STATUS   ROLES           AGE     VERSION
node1   Ready    control-plane   4h      v1.28.2
node2   Ready              3h59m   v1.28.2

$ kubectl get po -A -owide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE     IP              NODE    NOMINATED NODE   READINESS GATES
kube-system   coredns-6554b8b87f-xx59w               1/1     Running   0          88s     10.16.0.5       node2              
kube-system   coredns-6554b8b87f-zzdsb               1/1     Running   0          88s     10.16.0.4       node2              
kube-system   etcd-node1                             1/1     Running   0          4h1m    192.168.31.29   node1              
kube-system   kube-apiserver-node1                   1/1     Running   0          4h1m    192.168.31.29   node1              
kube-system   kube-controller-manager-node1          1/1     Running   0          4h1m    192.168.31.29   node1              
kube-system   kube-ovn-cni-bvj4b                     1/1     Running   0          96s     192.168.31.29   node1              
kube-system   kube-ovn-cni-hqqvn                     1/1     Running   0          96s     192.168.31.68   node2              
kube-system   kube-ovn-controller-65f6f75847-xjkgv   1/1     Running   0          96s     192.168.31.68   node2              
kube-system   kube-ovn-monitor-bd6bdf97-nmlbl        1/1     Running   0          96s     192.168.31.29   node1              
kube-system   kube-ovn-pinger-8cmjp                  1/1     Running   0          86s     10.16.0.6       node2              
kube-system   kube-proxy-q2td8                       1/1     Running   0          4h      192.168.31.68   node2              
kube-system   kube-proxy-w72hz                       1/1     Running   0          4h1m    192.168.31.29   node1              
kube-system   kube-scheduler-node1                   1/1     Running   0          4h1m    192.168.31.29   node1              
kube-system   ovn-central-745ff54dc-kzptf            1/1     Running   0          2m28s   192.168.31.29   node1              
kube-system   ovs-ovn-5gw99                          1/1     Running   0          2m28s   192.168.31.29   node1              
kube-system   ovs-ovn-qcxr8                          1/1     Running   0          2m27s   192.168.31.68   node2              
```
可以看到这里节点都已经 Ready，并且所有核心组件都处在 Running状态。到此位置 Kubernetes 集群就已经搭建完成

## 安装 stroageclass
安装openebs-localpv
是没有stroageclass，应该创建一个这个资源例如openhost这些，能提供pvc，minio需要pvc来存储文件。

## 安装Kubebb
1、安装 helm
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
2、安装 kubebb
```
git clone https://github.com/kubebb/core.git
./hack/quick-install.sh
```

## 安装nvidia gpu operator
1、在GPU节点安装helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
    && chmod 700 get_helm.sh \
    && ./get_helm.sh
```
Configuring containerd (for Kubernetes)

>Installing  nvidia-ctk with Apt，查看 https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#configuring-containerd-for-kubernetes

1、Configure the production repository:
```
$ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
Optionally, configure the repository to use experimental packages:
```
$ sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
2、Update the packages list from the repository: 
```
$ sudo apt-get update
```
3、Install the NVIDIA Container Toolkit packages: 
```
$ sudo apt-get install -y nvidia-container-toolkit
```
4、Configure the container runtime by using the nvidia-ctk command: 
```
$ sudo nvidia-ctk runtime configure --runtime=containerd
```

The nvidia-ctk command modifies the `/etc/containerd/config`.toml file on the host. The file is updated so that containerd can use the NVIDIA Container Runtime.

5、Restart containerd: 
```
$ sudo systemctl restart containerd
```

2、安装 gpu-operatpor
在master节点运行
```
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
helm install --generate-name \
 -n gpu-operator --create-namespace \
 nvidia/gpu-operator --set driver.enabled=false
```

## 安装 arcadia
1、克隆arcadia代码
```
git clone https://github.com/kubeagi/arcadia.git 
```
2、进入工作目录
```
cd arcadia/deploy/charts/arcadia 
```
3、编辑 values.yaml
替换 `<replaced-ingress-nginx-ip>` 为kubebb安装过程中部署的`ingress node IP`
>替换命令 :%s/old/new/g         

4、安装
```
helm install arcadia -n kubeagi-system --create-namespace  . 
```
5、查看安装状态
```
kubectl get pods -n kubeagi-system 
```
6、访问arcadia门户
```
https://portal.<replaced-ingress-nginx-ip>.nip.io
```
### 添加管理集群
1、为集群管理创建一个 namespace，可以使用 cluster-system，用来保存集群信息
```
kubectl create ns cluster-system  
```
2、获取添加集群的 token
```
export TOKENNAME=$(kubectl get serviceaccount/host-cluster-reader -n u4a-system -o jsonpath='{.secrets[0].name}')  
kubectl get secret $TOKENNAME -n u4a-system -o jsonpath='{.data.token}' | base64 -d  
```
3、登录管理平台，进入 “集群管理”，点击“添加集群”。 
4、输入集群名称，按需修改集群后缀，这里使用“API Token”方式接入集群。 
```
API Host，使用支持 OIDC 协议的 K8s API 地址，可以通过 kubectl get ingress -nu4a-system 查看kube-oidc-proxy-server-ingress 对应的 Host 信息，比如 https://k8s.172.22.96.136.nip.io
API Token，输入第 2 步获取的 token 信息
```
5、添加成功后，可以在列表上看到集群信息及其状态；选择“租户管理”，会看到名称为 "system-tenant" 的一个系统租户


## 【可选】安装ingress-nginx插件 
ingress-nginx的yaml文件下载地址：https://kubernetes.github.io/ingress-nginx/deploy/
使用 curl -O https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
下载到本地之后，修改里面的文件，需要将LoadBlance修改为NodePort,里面的镜像可参照对应版本换成阿里云仓库的方便下载registry.cn-hangzhou.aliyuncs.com/google_containers/，里面的部署方式为deployment,也可换成daemonset,service也可自行设置成固定的，若未设置，将自动分配。

完成后，访问服务如下：



## 【可选】安装dashboard 
dashboard下载地址：GitHub - kubernetes/dashboard: General-purpose web UI for Kubernetes clusters
下载yaml文件：curl -O https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
注意：有时候网络不好下载不下来，建议多试几次，会下载下来，下载下来之后将里面射击到的镜像标签改为阿里云仓库的，将dashboard的暴露类型改为nodePort。然后执行 kubectl apply -f recommended.yaml

部署完成后访问查看


创建一个登录dashboard的账户

kubectl get clusterrole #查看集群角色

kubectl get role -n kubernetes-dashboard #查看角色

kubectl -n kube-system describe clusterrole admin #描述角色

kubectl create serviceaccount dashboard-admin -n kube-system #在kube-system命名空间创建账户

kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin #将创建的账户绑定到集群角色cluster-admin

kubectl -n kube-system create token dashboard-admin 创建账户的token
粘贴token登录
将dashboard添加为Ingress服务
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: dashboard-ingress
namespace: kubernetes-dashboard
annotations:
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
ingress.kubernetes.io/ssl-passthrough: "true"
nginx.ingress.kubernetes.io/use-regex: "true"
nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
ingressClassName: nginx
rules:
- host: www.dashboard.com
http:
paths:
- path: /dashboard(/|$)(.*)
pathType: Prefix
backend:
service:
name: kubernetes-dashboard
port:
number: 443
```
执行kubectl apply -f 完成后，访问如下：
用的端口是ingress的https映射端口，若用ingress的http映射端口，则无法认证登录。