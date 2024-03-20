使用 Kubectl 进行 Kubernetes 诊断的指南。
列出了 100 个 Kubectl 命令，这些命令对于诊断 Kubernetes 集群中的问题非常有用。这些问题包括但不限于：
* 集群信息
* Pod 诊断
* 服务诊断
* 部署诊断
* 网络诊断
* 持久卷和持久卷声明诊断
* 资源使用情况
* 安全和授权
* 节点故障排除
* 其他诊断命令：文章还提到了许多其他命令，如资源扩展和自动扩展、作业和定时作业诊断、Pod 亲和性和反亲和性规则、RBAC 和安全、服务账号诊断、节点排空和取消排空、资源清理等。

### 集群信息：

1. 显示 Kubernetes 版本：kubectl version
2. 显示集群信息：kubectl cluster-info
3. 列出集群中的所有节点：kubectl get nodes
4. 查看一个具体的节点详情：kubectl describe node <node-name>
5. 列出所有命名空间：kubectl get namespaces
6. 列出所有命名空间中的所有 pod：kubectl get pods --all-namespaces


### Pod 诊断：
1. 列出特定命名空间中的 pod：kubectl get pods -n <namespace>
2. 查看一个 Pod 详情：kubectl describe pod <pod-name> -n <namespace>
3. 查看 Pod 日志：kubectl logs <pod-name> -n <namespace>
4. 尾部 Pod 日志：kubectl logs -f <pod-name> -n <namespace>
5. 在 pod 中执行命令：kubectl exec -it <pod-name> -n <namespace> -- <command>

### Pod 健康检查：

1. 检查 Pod 准备情况：`kubectl get pods <pod-name> -n <namespace> -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'`
2. 检查 Pod 事件：`kubectl get events -n <namespace> --field-selector involvedObject.name=<pod-name>`

### Service诊断：

1. 列出命名空间中的所有服务：kubectl get svc -n <namespace>
2. 查看一个服务详情：kubectl describe svc <service-name> -n <namespace>

### Deployment诊断：

1. 列出命名空间中的所有 Deployment：kubectl get deployments -n <namespace>
2. 查看一个Deployment详情：kubectl describe deployment <deployment-name> -n <namespace>
3. 查看滚动发布状态：kubectl rollout status deployment/<deployment-name> -n <namespace>
4. 查看滚动发布历史记录：kubectl rollout history deployment/<deployment-name> -n <namespace>

### StatefulSet诊断：

1. 列出命名空间中的所有 StatefulSet：kubectl get statefulsets -n <namespace>
2. 查看一个 StatefulSet详情：kubectl describe statefulset <statefulset-name> -n <namespace>

### ConfigMap 和 Secret 诊断：

1. 列出命名空间中的 ConfigMap：kubectl get configmaps -n <namespace>
2. 查看一个ConfigMap详情：kubectl describe configmap <configmap-name> -n <namespace>
3. 列出命名空间中的 Secret：kubectl get secrets -n <namespace>
4. 查看一个Secret详情：kubectl describe secret <secret-name> -n <namespace>

### 命名空间诊断：

1. 查看一个命名空间详情：kubectl describe namespace <namespace-name>
资源使用情况：

1. 检查 pod 的资源使用情况：kubectl top pod <pod-name> -n <namespace>
2. 检查节点资源使用情况：kubectl top nodes

### 网络诊断：

1. 显示命名空间中 Pod 的 IP 地址：kubectl get pods -n <namespace> -o custom-columns=POD:metadata.name,IP:status.podIP --no-headers
2. 列出命名空间中的所有网络策略：kubectl get networkpolicies -n <namespace>
3. 查看一个网络策略详情：kubectl describe networkpolicy <network-policy-name> -n <namespace>

### 持久卷 (PV) 和持久卷声明 (PVC) 诊断：

1. 列出PV：`kubectl get pv`
2. 查看一个PV详情：`kubectl describe pv <pv-name>`
3. 列出命名空间中的 PVC：`kubectl get pvc -n <namespace>`
4. 查看PVC详情：`kubectl describe pvc <pvc-name> -n <namespace>`

### 节点诊断：

1. 获取特定节点上运行的 Pod 列表：`kubectl get pods --field-selector spec.nodeName=<node-name> -n <namespace>`

### 资源配额和限制：

列出命名空间中的资源配额：kubectl get resourcequotas -n <namespace>
2. 查看一个资源配额详情：`kubectl describe resourcequota <resource-quota-name> -n <namespace>`

### 自定义资源定义 (CRD) 诊断：
1. 列出命名空间中的自定义资源：`kubectl get <custom-resource-name> -n <namespace>`
2. 查看自定义资源详情：`kubectl describe <custom-resource-name> <custom-resource-instance-name> -n <namespace>`
使用这些命令时，请记住将`<namespace>, <pod-name>, <service-name>, <deployment-name>, <statefulset-name>, <configmap-name>, <secret-name>, <namespace-name>, <pv-name>, <pvc-name>, <node-name>, <network-policy-name>, <resource-quota-name>, <custom-resource-name>`, 和替换为你的特定值。

`<custom-resource-instance-name>`这些命令应该可以帮助你诊断 Kubernetes 集群以及在其中运行的应用程序。

### 资源伸缩和自动伸缩

1. Deployment伸缩：`kubectl scale deployment <deployment-name> --replicas=<replica-count> -n <namespace>`
2. 设置Deployment的自动伸缩：`kubectl autoscale deployment <deployment-name> --min=<min-pods> --max=<max-pods> --cpu-percent=<cpu-percent> -n <namespace>`
3. 检查水平伸缩器状态：`kubectl get hpa -n <namespace>`

### 作业和 CronJob 诊断：

1. 列出命名空间中的所有作业：`kubectl get jobs -n <namespace>`
2. 查看一份工作详情：`kubectl describe job <job-name> -n <namespace>`
3. 列出命名空间中的所有 cron 作业：`kubectl get cronjobs -n <namespace>`
4. 查看一个 cron 作业详情：`kubectl describe cronjob <cronjob-name> -n <namespace>`

### 容量诊断：

1. 列出按容量排序的持久卷 (PV)：`kubectl get pv --sort-by=.spec.capacity.storage`
2. 查看PV回收策略：`kubectl get pv <pv-name> -o=jsonpath='{.spec.persistentVolumeReclaimPolicy}'`
3. 列出所有存储类别：`kubectl get storageclasses`

### Ingress和服务网格诊断：

1. 列出命名空间中的所有Ingress：`kubectl get ingress -n <namespace>`
2. 查看一个Ingress详情：`kubectl describe ingress <ingress-name> -n <namespace>`
3. 列出命名空间中的所有 VirtualServices (Istio)：`kubectl get virtualservices -n <namespace>`
4. 查看一个 VirtualService (Istio)详情：`kubectl describe virtualservice <virtualservice-name> -n <namespace>`

### Pod 网络故障排除：

1. 运行网络诊断 Pod（例如 busybox）进行调试：`kubectl run -it --rm --restart=Never --image=busybox net-debug-pod -- /bin/sh`
2. 测试从 Pod 到特定端点的连接：`kubectl exec -it <pod-name> -n <namespace> -- curl <endpoint-url>`
3. 跟踪从一个 Pod 到另一个 Pod 的网络路径：`kubectl exec -it <source-pod-name> -n <namespace> -- traceroute <destination-pod-ip>`
4. 检查 Pod 的 DNS 解析：`kubectl exec -it <pod-name> -n <namespace> -- nslookup <domain-name>`

### 配置和资源验证：

1. 验证 Kubernetes YAML 文件而不应用它：`kubectl apply --dry-run=client -f <yaml-file>`
2. 验证 pod 的安全上下文和功能：`kubectl auth can-i list pods --as=system:serviceaccount:<namespace>:<serviceaccount-name>`

### RBAC 和安全性：

1. 列出命名空间中的角色和角色绑定：`kubectl get roles,rolebindings -n <namespace>`
2. 查看角色或角色绑定详情：`kubectl describe role <role-name> -n <namespace>`

### 服务帐户诊断：

1. 列出命名空间中的服务帐户：`kubectl get serviceaccounts -n <namespace>`
2. 查看一个服务帐户详情：`kubectl describe serviceaccount <serviceaccount-name> -n <namespace>`

### 清空节点和解除封锁：

1. 清空节点以进行维护：`kubectl drain <node-name> --ignore-daemonsets`
2. 解除对节点的封锁：`kubectl uncordon <node-name>`

### 资源清理：

1. 强制删除 pod（不推荐）：`kubectl delete pod <pod-name> -n <namespace> --grace-period=0 --force`

### Pod 亲和性和反亲和性：

1. 列出 pod 的 pod 亲和性规则：`kubectl get pod <pod-name> -n <namespace> -o=jsonpath='{.spec.affinity}'`
2. 列出 pod 的 pod 反亲和性规则：`kubectl get pod <pod-name> -n <namespace> -o=jsonpath='{.spec.affinity.podAntiAffinity}'`

### Pod 安全策略 (PSP)：

1. 列出所有 Pod 安全策略（如果启用）：`kubectl get psp`

### 事件：

1. 查看最近的集群事件：`kubectl get events --sort-by=.metadata.creationTimestamp`
2. 按特定命名空间过滤事件：`kubectl get events -n <namespace>`

### 节点故障排除：

1. 检查节点情况：`kubectl describe node <node-name> | grep Conditions -A5`
2. 列出节点容量和可分配资源：`kubectl describe node <node-name> | grep -E "Capacity|Allocatable"`

### 临时容器（Kubernetes 1.18+）：

1. 运行临时调试容器：`kubectl debug -it <pod-name> -n <namespace> --image=<debug-image> -- /bin/sh`

### 资源指标（需要指标服务器）：

1. 获取 Pod 的 CPU 和内存使用情况：`kubectl top pod -n <namespace>`

### kuelet诊断：

1. 查看节点上的kubelet日志：`kubectl logs -n kube-system kubelet-<node-name>`

### 使用Telepresence 进行高级调试：

1. 使用 Telepresence 调试 pod：`telepresence --namespace <namespace> --swap-deployment <pod-name>`

### Kubeconfig 和上下文：

1. 列出可用的上下文：`kubectl config get-contexts`
2. 切换到不同的上下文：`kubectl config use-context <context-name>`

### Pod 安全标准（PodSecurity 准入控制器）：

1. 列出 PodSecurityPolicy (PSP) 违规行为：`kubectl get psp -A | grep -vE 'NAME|REVIEWED'`

### Pod 中断预算 (PDB) 诊断：

1. 列出命名空间中的所有 PDB：`kubectl get pdb -n <namespace>`
2. 查看一个PDB详情：`kubectl describe pdb <pdb-name> -n <namespace>`

### 资源锁诊断（如果使用资源锁）：

1. 列出命名空间中的资源锁：`kubectl get resourcelocks -n <namespace>`

### 服务端点和 DNS：

1. 列出服务的服务端点：`kubectl get endpoints <service-name> -n <namespace>`
2. 检查 Pod 中的 DNS 配置：`kubectl exec -it <pod-name> -n <namespace> -- cat /etc/resolv.conf`

### 自定义指标（Prometheus、Grafana）：

1. 查询Prometheus指标：用于 `kubectl port-forward` 访问Prometheus和Grafana服务来查询自定义指标。

### Pod 优先级和抢占：

1. 列出优先级：`kubectl get priorityclasses`

### Pod 开销（Kubernetes 1.18+）：

1. 列出 pod 中的开销：`kubectl get pod <pod-name> -n <namespace> -o=jsonpath='{.spec.overhead}'`

### 存储卷快照诊断（如果使用存储卷快照）：

1. 列出存储卷快照：`kubectl get volumesnapshot -n <namespace>`
2. 查看存储卷快照详情：`kubectl describe volumesnapshot <snapshot-name> -n <namespace>`

### 资源反序列化诊断：

1. 反序列化并打印 Kubernetes 资源：`kubectl get <resource-type> <resource-name> -n <namespace> -o=json`

### 节点污点：

1. 列出节点污点：`kubectl describe node <node-name> | grep Taints`

### 更改和验证 Webhook 配置：

1. 列出变异 webhook 配置：`kubectl get mutatingwebhookconfigurations`
2. 列出验证 Webhook 配置：`kubectl get validatingwebhookconfigurations`

### Pod 网络策略：

1. 列出命名空间中的 pod 网络策略：`kubectl get networkpolicies -n <namespace>`

### 节点条件（Kubernetes 1.17+）：

1. 自定义查询输出：`kubectl get nodes -o custom-columns=NODE:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status -l 'node-role.kubernetes.io/worker='`

### 审核日志：

1. 检索审核日志（如果启用）：检查 Kubernetes 审核日志配置以了解审核日志的位置。

### 节点操作系统详细信息：

1. 获取节点的操作系统信息：`kubectl get node <node-name> -o jsonpath='{.status.nodeInfo.osImage}'`

这些命令应该涵盖 Kubernetes 中的各种诊断场景。确保将`<namespace>、<pod-name>、<deployment-name>`等占位符替换为你的集群和用例的实际值。

***

## 常用命令

1、create、apply 命令：根据文件或者输入来创建资在这里插入代码片
>创建Deployment和Service资源
kubectl creat xxx.yaml #如果不存在则创建，如果存在，则报错
kubectl apply xxx.yaml #如果不存在则创建，如果存在，则更新

2、delete 命令：删除资源
>根据yaml文件删除对应的资源，但是yaml文件并不会被删除，这样更加高效
kubectl delete -f xxx.yaml kubectl delete -f xxx.yaml
也可以通过具体的资源名称来进行删除，使用这个删除资源，同时删除deployment、pod、service资源 kubectl delete 具体的资源名

3、get 命令 ：获得资源信息
>查看default命名空间的所有资源，显示运行中的Pod、Service、Deployment以及ReplicaSet的关键信息
kubectl get all
查看所有的命名空间的资源
kubectl get all --all-namespaces
查看集群命名空间（namespace = ns）
kubectl get ns
列出所有运行的Pod信息
kubectl get pods
查看Pod详细信息，可以查看 pod 的列表，其中 READY 列代表该 Pod 总共有多少（查看组件状态）个容器，并且该容器已经成功启动，可以对外提供服务了,也可以看到这个pod属于哪个node
kubectl get pods -o wide
以yaml格式查看Pod详细信息
kubectl get pod -o yaml
以json格式查看Pod详细信息
kubectl get pod -o json
显示pod节点的标签信息
kubectl get pod --show-labels
查看Pod标签
kubectl get pod -n nsName --show-labels
获取指定名称空间中的指定pod
kubectl get pod -n kube-system podName
查询集群中的所有pod，带有namespace
kubectl get pod --all-namespaces
查看所有命名空间POD详细状态，前四列查出结果与kubectl get pods -A相同
kubectl get pod --all-namespaces -o wide
查看集群节点信息
kubectl get node
显示node节点的标签信息
kubectl get node --show-labels
获取节点和服务版本，以及查看附加信息
kubectl get nodes -o wide
查看已经部署了的所有应用，可以看到容器，以及容器所用的镜像，标签等信息
kubectl get deploy -o wide kubectl get deployments -o wide
查看组件状态（componentstatuseskubectl = cs）
kubectl get cs
查看所有的pod的副本数，以及他们的可用数量以及状态等信息
kubectl get rs
查看服务的详细信息，显示了服务名称，类型，集群ip，端口，时间等信息
kubectl get svc
查看指定命名空间的服务
kubectl get svc -n kube-system
查看rc和service列表
kubectl get rc,service
查看指定命名空间service信息
kubectl get services -n namespacesName
查看pod,svc,ep能及标签信息
kubectl get pod,svc,ep --show-labels
查看指定命名空间域名信息
kubernetes get ingress -n namespacesName
查看pv(persistentvolumes)信息
kubectl get -n ops pv
查看其他健康端点
kubectl get --raw ‘/healthz?verbose’
查看集群事件，watch命令，挂起观察
kubectl get events -A
replicaset是用来管理实例数量的，可以看成是rc/deployment的一个对象
kubectl get replicaset
查看关联后端的节点
kubectl get endpoints
检查ingress域
kubectl get ingress -n namespacesName

4、top 命令：用于查看资源的cpu，内存磁盘等资源的使用率
>查看node的使用情况
kubectl top node
查看node的使用情况，–containers可以显示 pod 内所有的container
kubectl top pod
找CPU消耗最高的Pod
kubectl top pod -l -ncpu-loader --sort-by=cpu -A

5、logs命令：用于将容器中的日志导出
>查看日志，-c容器名，-f 指定是否持续输出日志
kubectl logs -n namespacesName -f [-c 容器名]
查看曾经运行郭，现在已经停止的容器的日志
kubectl logs -p -n namespacesName -f -c 容器名
查看最近20条日志
kubectl logs --tail=20 -n namespacesName -f -c 容器名
查看最近一小时内产生的所有日志
kubectl logs --since=1h -n nsName -f -c 容器名

6、run 命令：在集群中创建并运行一个或多个容器镜像
>启动一个实例
kubectl run NAME --image=image
启动一个有5个副本的nginx实例,并在容器中设置环境变量
kubectl run NAME --image=image --replicas=5
试运行，不创建他们的情况下，打印出所有相关的API对象
kubectl run NAME --image=image --dry-run
重启策略设置为Never，不自动重启；重启策略默认是always，总是自动拉取
kubectl run -i -t NAME --image=image --restart=Never

7、expose 命令：创建一个service服务，并且暴露端口让外部可以访问
>为deployment（无状态部署）的nginx创建service，并通过Service的80端口转发至容器的80端口上，Service的名称为nginx-service类型为NodePort
kubectl expose deployment nginx --port=88 --type=NodePort --target-port=80 --name=nginx-service

8、describe命令：显示特定资源的详细信息
>描述一个pod，查看非running 状态到具体可能原因
kubectl describe pods podName -n namespacesName
描述xxx.yaml中的资源类型和名称指定的pod
kubectl describe -f xxx.yaml
描述所有的pod
kubectl describe pods --all-namespaces
描述所有Labels为xxxx的pod
kubectl describe pods -l xxxx --all-namespaces

9、explain 命令：用于显示资源文档信息
>命令详解
kubectl explain pods.spec.containers.livenessProbe.httpGet

10、set 命令：配置应用的一些特定资源，也可以修改应用已有的资源
>这个命令用于设置资源的一些范围限制,可用资源对象包括replicationcontroller、deployment、daemonset、job、replicaset
kubectl set resources …
将deployment的指定容器cpu限制为“200m”，将内存设置为“512Mi”
kubectl set resources deployment 容器名 --limits=cpu=200m,memory=512Mi
删除deployment的指定容器的计算资源值
kubectl set resources deployment 容器名 --limits=cpu=0,memory=0 --requests=cpu=0,memory=0
用于更新现有资源的容器镜像
kubectl set image…
将所有deployment中的指定容器的镜像改为新镜像
kubectl set image deployment 容器名 原镜像=新镜像
将所有deployment和rc指定镜像的容器更新为新镜像
kubectl set image deployment,rc 指定镜像=新镜像 --all

11、label命令: 用于更新(增加、修改或删除)资源上的 label(标签) label
>必须以字母或数字开头，可以使用字母、数字、连字符、点和下划线，最长63个字符Pod添加label unhealthy=true
kubectl label pods podName unhealthy=true
给Pod修改label 为 ‘status’ / value ‘unhealthy’，且覆盖现有的value
kubectl label --overwrite pods podName status=unhealthy

12、exec命令：进入容器进行交互，在容器中执行命令
>连接容器，-c 容器、 -p pod名、-i 将控制台输入发送到容器、-t 将标准输入控制台作为容器的控制台输入
kubectl exec jenkins-864794d566-zj6rt -n ops -it – /bin/bash

13、api-servions命令：打印受支持的api版本信息
>打印当前集群支持的api版本信息
kubectl api-versions

14、rollout命令: 对指定容器进行重启
>重启容器
kubectl rollout restart statefulsets prometheus -n monitor

15、cp：用于pod和外部的文件交换，将文件和目录复制到容器或从容器复制到容器
>将本地目录或文件拷贝到默认命名空间的远端pod的目录下
kubectl cp 本地目录 some-podName:远端pod目录
复制本地文件到指定目录在远程pod在一个特定的容器
kubectl cp 本地目录 some-podName:远端pod目录 -c 容器名
