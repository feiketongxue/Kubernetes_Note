# `Pod控制器`

## 介绍

`Pod`是 `kubernetes`的最小管理单元，在 `kubernetes`中，按照 `pod`的创建方式可以将其分为两类：

- 自主式 `pod`：`kubernetes`直接创建出来的 `Pod`，这种 `pod`删除后就没有了，也不会重建
- 控制器创建 `pod`：`kubernetes`通过控制器创建 `pod`，这种 `pod`删除了之后还会自动重建

> **`什么是Pod控制器`**
> 
> `Pod`控制器是管理 `pod`的中间层，使用 `Pod`控制器之后，只需要告诉 `Pod`控制器，想要多少个什么样的 `Pod`就可以了，它会创建出满足条件的 `Pod`并确保每一个 `Pod`资源处于用户期望的目标状态。如果 `Pod`资源在运行中出现故障，它会基于指定策略重新编排 `Pod`。
> 
> **`在kubernetes中，有很多类型的pod控制器，每种都有自己的适合的场景，常见的有下面这些：`**
> 
> - `ReplicationController`：比较原始的 `pod`控制器，~~已经被废弃~~，由 `ReplicaSet`替代
> - `ReplicaSet`：保证副本数量一直维持在期望值，并支持 `pod`数量扩缩容，镜像版本升级
> - `Deployment`：通过控制 `ReplicaSet`来控制 `Pod`，并支持滚动升级、回退版本，是 `ReplicaSet`的升级版，拥有 `ReplicaSet`的全部功能
> - `Horizontal Pod Autoscaler`：可以根据集群负载自动水平调整 `Pod`的数量，实现削峰填谷
> - `DaemonSet`：在集群中的指定 `Node`上运行且仅运行一个副本，一般用于守护进程类的任务
> - `Job`：它创建出来的 `pod`只要完成任务就立即退出，不需要重启或重建，用于执行一次性任务
> - `Cronjob`：它创建的 `Pod`负责周期性任务控制，不需要持续后台运行
> - `StatefulSet`：管理有状态应用

## ReplicaSet（RS）

`ReplicaSet`的主要作用是**保证一定数量的pod正常运行**，它会持续监听这些 `Pod`的运行状态，一旦 `Pod`发生故障，就会重启或重建。同时它还支持对 `pod`数量的扩缩容和镜像版本的升降级。

![20240321113605628](../Images/image-20240321113605628.png)

```shell
# ReplicaSet的资源清单文件
apiVersion: apps/v1       # 版本号
kind: ReplicaSet          # 类型   
metadata:                 # 元数据
  name:                   # rs名称 
  namespace:              # 所属命名空间 
  labels: 	              # 标签
    controller: rs
spec:                     # 详情描述
  replicas: 3             # 副本数量
  selector:               # 选择器，通过它指定该控制器管理哪些pod
    matchLabels:          # Labels匹配规则
      app: nginx-pod
    matchExpressions:     # Expressions匹配规则
      - {key: app, operator: In, values: [nginx-pod]}
  template:               # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

> 在这里面，需要新了解的配置项就是 `spec`下面几个选项：
> 
> - `replicas`：指定副本数量，其实就是当前 `rs`创建出来的 `pod`的数量，默认为1
> - `selector`：选择器，它的作用是建立 `pod`控制器和 `pod`之间的关联关系，采用的 `Label Selector`机制
>   
>   在 `pod`模板上定义 `label`，在控制器上定义选择器，就可以表明当前控制器能管理哪些 `pod`了
> - `template`：模板，就是当前控制器创建 `pod`所使用的模板板，里面其实就是前一章学过的 `pod`的定义


### 创建 ReplicaSet

```shell
# 创建pc-replicaset.yaml文件
apiVersion: apps/v1
kind: ReplicaSet   
metadata:
  name: pc-replicaset
  namespace: dev
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
```

```sh
# 清除之前的 namespace
[root@master Download]# kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   9d    v1.17.4
node1    Ready    <none>   9d    v1.17.4
node2    Ready    <none>   9d    v1.17.4
[root@master Download]# kubectl delete ns dev
namespace "dev" deleted
[root@master Download]# kubectl get ns
NAME              STATUS   AGE
default           Active   9d
kube-flannel      Active   9d
kube-node-lease   Active   9d
kube-public       Active   9d
kube-system       Active   9d

[root@master ~]# kubectl create ns dev
namespace/dev created

# 创建rs
[root@master ~]# kubectl create -f pc-replicaset.yaml
replicaset.apps/pc-replicaset created

# 查看rs
# DESIRED:期望副本数量  
# CURRENT:当前副本数量  
# READY:已经准备好提供服务的副本数量
[root@master ~]# kubectl get rs pc-replicaset -n dev -o wide
NAME            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
pc-replicaset   3         3         3       4s    nginx        nginx:1.17.1   app=nginx-pod

# 查看当前控制器创建出来的pod
# 这里发现控制器创建出来的pod的名称是在控制器名称后面拼接了-xxxxx随机码
[root@master ~]# kubectl get pod -n dev
NAME                  READY   STATUS    RESTARTS   AGE
pc-replicaset-6zw7d   1/1     Running   0          8s
pc-replicaset-tzh6l   1/1     Running   0          8s
pc-replicaset-vs7d6   1/1     Running   0          8s
```

### 扩缩容

### 镜像升级

### 镜像升级

### 删除ReplicaSet


## Deployment(Deploy)

### 创建 deployment

### 扩缩容

### 版本回退

### 金丝雀发布

## Horizontal Pod Autoscaler(HPA)

### 安装metrics-server

### 准备 deployment 和 servie

### 部署 HPA

### 测试

## DaemonSet(DS) 

## Job

## CronJob(CJ)