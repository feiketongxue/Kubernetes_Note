# Kubernetes 官网（中文）

[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)

## Kubernetes 组件

一个kubernetes 集群主要是由 控制节点（master），工作节点（node）节点构成，每个节点都会安装不同的组件。

### master：负责集群的控制平面，负责集群的决策（管理）

> ApiServer: 资源操作的唯一入口，接受用户输入的命令，提供认证、授权、API注册和发现等机制

> Scheduler: 负责整个集群的资源调度，按照预定的调度策略将Pod 调度到相应的node节点上

> ControllerManager: 组册维护集群的状态，比如 程序部署安排、故障检测、自动扩展、滚动更新等

> Etcd: 负责存储集群中各种资源对象的信息

### node:  集群的数据平面，负责为容器提供运行环境（干活）

> Kubelet: 负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器

> KubeProxy: 负责提供集群内部的服务发现和负载均衡

> Docker: 负责节点上容器的各种操作

![image-20240223100857698](/Users/fico/Library/Application%20Support/typora-user-images/image-20240223100857698.png)

 下面，以部署一个nginx 服务来说明kubernetes系统各个组件调用关系：

1. 首先要明确，一旦kubernetes 环境启动之后，master和node都会将自身的信息存储到etcd 数据库中，也就是持久化处理；

2. 一个nginx 服务的安装请求会先被发送到master 节点的 apiServer 组件；

3. apiServer 组件会调用 scheduler 组件来决定到底应该吧这个服务安装到呢个node节点上，在此时，他会从etcd 中读取各个node 节点的信息，然后按照一定的算法进行选择，并将结果告知apiServer；

4. apiServer 调用 controller-manager 去调度Node节点安装 nginx 服务；

5. kubelet 接收到指令后，会通知docker ，然后由 docker 来启动一个 nginx 的pod；（pod是kubernetes的最小操作单元，容器必须跑在pod中）

6. 这样，一个 nginx 服务就运行了，如果需要访问 nginx，就需要通过 kube-proxy 来对 pod 产生的代理；

   **外界用户就可以访问集群中的 nginx 服务了**

## kubernetes 概念

   -  Master：集群控制节点，每个集群需要至少一个master 节点负责集群的管控；
   -  Node：工作负载节点，由 master 分配容器到这些 node 工作点上，然后 node 节点上的 docker 负责容器的运行；
   -  Pod：kubernetes 的最小控制单元，容器都是运行在 pod 中的，一个pod 中，可以有 1 个或者多个容器
   -  Controller：多种控制器，通过他来实现对 pod 的管理，比如 启动 pod、停止 pod 的数量等；
   -  Service：pod对外服务的统一福口，下面可以维护同一类的多个 pod
   -  Label：标签，用于对 pod 进行分类，同一类 pod 会拥有相同的标签，比如下图中的 app:tomcat 、app:tom标签 
   -  NameSpace：命名空间，用来隔离pod的运行环境 ，比如 dev、prod，test 环境

![image-20240223104622087](/Users/fico/Library/Application%20Support/typora-user-images/image-20240223104622087.png)





   