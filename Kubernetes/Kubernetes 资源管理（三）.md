# `Kubernetes` 资源管理

## 资源管理介绍

在 `kubernetes` 中，所有的内容都抽象为资源，用户通过操作资源来管理 `kuernetes`

> ​	`kubernetes`的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在`kubernetes`集群中运行一个个的容器，并将指定的程序跑在容器中。
>
> ​	`kubernetes`的最小管理单元是 `pod`而不是容器，所以只能将容器放在`Pod`中，而 `kubernetes`一般也不会直接管理`Pod`，而是通过`Pod控制器`来管理 `Pod`。
>
> ​	`Pod`可以提供服务，就要考虑如何访问`Pod`中服务，`kubernetes`提供了`Service`资源实现这个功能，
>
> ​	当然，如果`Pod`中程序的数据需要持久化，`kubernetes`还提供了 各种`存储`系统。

<img src="/Users/fico/Library/Application%20Support/typora-user-images/image-20240228112944374.png" alt="image-20240228112944374" style="zoom:80%;" />

## `YAML`语言介绍

​	`YAML` 是一个类似于`XML`、`JSON`的标记性语言，强调以`数据`为中心，并不是以标记语言为重点，因而 `YAML`本身的定义比较简单，号称 `一种人性化的数据格式语言`

例如：

```xml
<name>
  <age>15</age>
  <class>2</class>
</name>
```

```yaml
name:
	age: 15
	class: 2
```

​	`YAML`的语法较为简单，主要有下面几个：

		- 大小写敏感
		- 使用缩紧表示层级关系
		- 缩紧不允许使用`tab`，只允许空格
		- 缩紧的空格数不重要，只要相同层级的元素左对齐即可
		- `#`表示注释

​	`YAML`支持以下几种数据类型：

- 纯量：单个的，不可再分的值
- 对象：键值对的集合，又称为 映射（Mapping）/哈希（Hash）/字典（Dictionary）
- 数组：一组按次序排列的值，又称为序列（Sequnce）/列表（List）

> PS：小提示
>
> 1. 书写 `YAML`切记：后面要加一个空格
> 2. 如果需要将多段`YAML`配置放在一个文件中，中间要使用 --- 分隔

## 资源管理方式

- 命令式对象管理：直接使用命令去操作`kubernetes`资源

  ```shell
  kuberctl run nginx-pod --image=nginx:1.17.1 --port=80
  ```

- 命令式对象配置：通过命令配置和配置文件去操作`kubernetes`资源

  ```shell
  kuberctl create/patch -f nginx-pod.yaml
  ```

- 声明式对象配置：通过`apply`命令和配置文件去操作`kubernetes`资源

  ```shell
  kuberctl apply -f nginx-pod.yaml
  ```

  |      类型      | 操作 | 适用环境 | 优点           | 缺点                             |
  | :------------: | :--: | -------- | -------------- | -------------------------------- |
  | 命令式对象管理 | 对象 | 测试     | 简单           | 只能操作活动对象，无法审计，跟踪 |
  | 命令式对象配置 | 文件 | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
  | 声明式对象配置 | 目录 | 开发     | 支持目录操作   | 意外情况下难以调试               |

  ### 命令式对象管理
  
  **kubectl 命令**
  
  ​	`kubectl`是`kubernetes`集群的命令行工具，可以对集群本身进行管理，并能够在集群上进行容器话应用的安装部署。`kubectl` 命令的语法如下：
  
  ```shell
  kubectl [command] [type] [name] [flags]
  ```
  
  **command**：指定要对资源执行的做操，例如 create、get、delete
  
  **type**：指定的资源类型，比如 deployment、pod、service
  
  **name**：指定资源的名称，名称大小写敏感
  
  **flags**：指定额外的可选参数
  
  ```shell
  # 查看所有 pod
  [root@master ~]# kubectl get pod
  NAME                     READY   STATUS    RESTARTS   AGE
  nginx-6867cdf567-72rjc   1/1     Running   0          23h
  
  # 查看某个 pod
  [root@master ~]# kubectl get pod nginx-6867cdf567-72rjc
  NAME                     READY   STATUS    RESTARTS   AGE
  nginx-6867cdf567-72rjc   1/1     Running   0          23h
  
  # 查看某个 pod，以 yaml 格式展示结果
  [root@master ~]# kubectl get pod nginx-6867cdf567-72rjc -o yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2024-02-27T07:05:22Z"
    generateName: nginx-6867cdf567-
    labels:
      app: nginx
      pod-template-hash: 6867cdf567
    name: nginx-6867cdf567-72rjc
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: nginx-6867cdf567
      uid: b4051001-4315-47a7-b0e4-582af7a4f76f
    resourceVersion: "2186"
    selfLink: /api/v1/namespaces/default/pods/nginx-6867cdf567-72rjc
    uid: a8541f81-4b7b-4feb-8172-eba576743f25
  spec:
    containers:
    - image: nginx:1.14-alpine
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: default-token-ffbqn
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: node2
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: default-token-ffbqn
      secret:
        defaultMode: 420
        secretName: default-token-ffbqn
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2024-02-27T07:05:22Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2024-02-27T07:05:45Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2024-02-27T07:05:45Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2024-02-27T07:05:22Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: docker://c969662075660f774478c997acb188d0eff9224d72a9da193df90c006c51a5d8
      image: nginx:1.14-alpine
      imageID: docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
      lastState: {}
      name: nginx
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2024-02-27T07:05:44Z"
    hostIP: 192.168.2.108
    phase: Running
    podIP: 10.244.2.2
    podIPs:
    - ip: 10.244.2.2
    qosClass: BestEffort
    startTime: "2024-02-27T07:05:22Z"
    
  # 查看某个 pod，以 json 格式展示结果
  [root@master ~]# kubectl get pod nginx-6867cdf567-72rjc -o json
  {
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
          "creationTimestamp": "2024-02-27T07:05:22Z",
          "generateName": "nginx-6867cdf567-",
          "labels": {
              "app": "nginx",
              "pod-template-hash": "6867cdf567"
          },
          "name": "nginx-6867cdf567-72rjc",
          "namespace": "default",
          "ownerReferences": [
              {
                  "apiVersion": "apps/v1",
                  "blockOwnerDeletion": true,
                  "controller": true,
                  "kind": "ReplicaSet",
                  "name": "nginx-6867cdf567",
                  "uid": "b4051001-4315-47a7-b0e4-582af7a4f76f"
              }
          ],
          "resourceVersion": "2186",
          "selfLink": "/api/v1/namespaces/default/pods/nginx-6867cdf567-72rjc",
          "uid": "a8541f81-4b7b-4feb-8172-eba576743f25"
      },
      "spec": {
          "containers": [
              {
                  "image": "nginx:1.14-alpine",
                  "imagePullPolicy": "IfNotPresent",
                  "name": "nginx",
                  "resources": {},
                  "terminationMessagePath": "/dev/termination-log",
                  "terminationMessagePolicy": "File",
                  "volumeMounts": [
                      {
                          "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                          "name": "default-token-ffbqn",
                          "readOnly": true
                      }
                  ]
              }
          ],
          "dnsPolicy": "ClusterFirst",
          "enableServiceLinks": true,
          "nodeName": "node2",
          "priority": 0,
          "restartPolicy": "Always",
          "schedulerName": "default-scheduler",
          "securityContext": {},
          "serviceAccount": "default",
          "serviceAccountName": "default",
          "terminationGracePeriodSeconds": 30,
          "tolerations": [
              {
                  "effect": "NoExecute",
                  "key": "node.kubernetes.io/not-ready",
                  "operator": "Exists",
                  "tolerationSeconds": 300
              },
              {
                  "effect": "NoExecute",
                  "key": "node.kubernetes.io/unreachable",
                  "operator": "Exists",
                  "tolerationSeconds": 300
              }
          ],
          "volumes": [
              {
                  "name": "default-token-ffbqn",
                  "secret": {
                      "defaultMode": 420,
                      "secretName": "default-token-ffbqn"
                  }
              }
          ]
      },
      "status": {
          "conditions": [
              {
                  "lastProbeTime": null,
                  "lastTransitionTime": "2024-02-27T07:05:22Z",
                  "status": "True",
                  "type": "Initialized"
              },
              {
                  "lastProbeTime": null,
                  "lastTransitionTime": "2024-02-27T07:05:45Z",
                  "status": "True",
                  "type": "Ready"
              },
              {
                  "lastProbeTime": null,
                  "lastTransitionTime": "2024-02-27T07:05:45Z",
                  "status": "True",
                  "type": "ContainersReady"
              },
              {
                  "lastProbeTime": null,
                  "lastTransitionTime": "2024-02-27T07:05:22Z",
                  "status": "True",
                  "type": "PodScheduled"
              }
          ],
          "containerStatuses": [
              {
                  "containerID": "docker://c969662075660f774478c997acb188d0eff9224d72a9da193df90c006c51a5d8",
                  "image": "nginx:1.14-alpine",
                  "imageID": "docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7",
                  "lastState": {},
                  "name": "nginx",
                  "ready": true,
                  "restartCount": 0,
                  "started": true,
                  "state": {
                      "running": {
                          "startedAt": "2024-02-27T07:05:44Z"
                      }
                  }
              }
          ],
          "hostIP": "192.168.2.108",
          "phase": "Running",
          "podIP": "10.244.2.2",
          "podIPs": [
              {
                  "ip": "10.244.2.2"
              }
          ],
          "qosClass": "BestEffort",
          "startTime": "2024-02-27T07:05:22Z"
      }
  }
  
  
  # 查看 node 的详细信息
  [root@master ~]# kubectl get pod nginx-6867cdf567-72rjc -o wide
  NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE   READINESS GATES
  nginx-6867cdf567-72rjc   1/1     Running   0          23h   10.244.2.2   node2   <none>           <none>
  ```

> `kubernetes` 允许对资源进行多种操作，可以通过 -help 查看详细的操作命令
>
> ```shell
> kubectl --help
> ```
>
> `kubernetes` 中所有的内容都抽象为资源，可以通过下面的命令进行查看:
>
> ```shell
> kubectl api-resources
> ```

> 经常用到的命令有下面这些：

|  命令分类  | 命令         | 翻译                        | 命令作用                     |
| :--------: | ------------ | --------------------------- | ---------------------------- |
|  基本命令  | create       | 创建                        | 创建一个资源                 |
|            | edit         | 编辑                        | 编辑一个资源                 |
|            | get          | 获取                        | 获取一个资源                 |
|            | patch        | 更新                        | 更新一个资源                 |
|            | delete       | 删除                        | 删除一个资源                 |
|            | explain      | 解释                        | 展示资源文档                 |
| 运行和调试 | run          | 运行                        | 在集群中运行一个指定的镜像   |
|            | expose       | 暴露                        | 暴露资源为Service            |
|            | describe     | 描述                        | 显示资源内部信息             |
|            | logs         | 日志输出容器在 pod 中的日志 | 输出容器在 pod 中的日志      |
|            | attach       | 缠绕进入运行中的容器        | 进入运行中的容器             |
|            | exec         | 执行容器中的一个命令        | 执行容器中的一个命令         |
|            | cp           | 复制                        | 在Pod内外复制文件            |
|            | rollout      | 首次展示                    | 管理资源的发布               |
|            | scale        | 规模                        | 扩(缩)容Pod的数量            |
|            | autoscale    | 自动调整                    | 自动调整Pod的数量            |
|  高级命令  | apply        | rc                          | 通过文件对资源进行配置       |
|            | label        | 标签                        | 更新资源上的标签             |
|  其他命令  | cluster-info | 集群信息                    | 显示集群信息                 |
|            | version      | 版本                        | 显示当前Server和Client的版本 |

> 经常用到的资源都有下面这些：

| 资源分类      | 资源名称                 | 缩写    | 资源作用        |
| ------------- | ------------------------ | ------- | --------------- |
| 集群级别资源  | nodes                    | no      | 集群组成部分    |
| namespaces    | ns                       | 隔离Pod |                 |
| pod资源       | pods                     | po      | 装载容器        |
| pod资源控制器 | replicationcontrollers   | rc      | 控制pod资源     |
|               | replicasets              | rs      | 控制pod资源     |
|               | deployments              | deploy  | 控制pod资源     |
|               | daemonsets               | ds      | 控制pod资源     |
|               | jobs                     |         | 控制pod资源     |
|               | cronjobs                 | cj      | 控制pod资源     |
|               | horizontalpodautoscalers | hpa     | 控制pod资源     |
|               | statefulsets             | sts     | 控制pod资源     |
| 服务发现资源  | services                 | svc     | 统一pod对外接口 |
|               | ingress                  | ing     | 统一pod对外接口 |
| 存储资源      | volumeattachments        |         | 存储            |
|               | persistentvolumes        | pv      | 存储            |
|               | persistentvolumeclaims   | pvc     | 存储            |
| 配置资源      | configmaps               | cm      | 配置            |
|               | secrets                  |         | 配置            |

**练习：下面以一个 namespace/pod 的创建和删除简单演示命令的使用：**

```shell
# 查看 kubectl 版本
[root@master ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.4", GitCommit:"8d8aa39598534325ad77120c120a22b3a990b5ea", GitTreeState:"clean", BuildDate:"2020-03-12T21:03:42Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.4", GitCommit:"8d8aa39598534325ad77120c120a22b3a990b5ea", GitTreeState:"clean", BuildDate:"2020-03-12T20:55:23Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}

# 查看 kubectl 集群信息
[root@master ~]# kubectl cluster-info
Kubernetes master is running at https://192.168.2.87:6443
KubeDNS is running at https://192.168.2.87:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


# 创建一个 namespace 这里简写为 kubectl create ns dev
[root@master ~]# kubectl create namespace dev
namespace/dev created

# 查看命名空间
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   24h
dev               Active   7s
kube-flannel      Active   24h
kube-node-lease   Active   24h
kube-public       Active   24h
kube-system       Active   24h

# 在此dev namespace下创建并运行一个nginx的Pod
[root@master ~]# kubectl run pod --image=nginx:1.17.1 -n dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/pod created

# 指定 namespace 查看pod
[root@master ~]# kubectl get pod -n dev
NAME                  READY   STATUS    RESTARTS   AGE
pod-cbb995bbf-xb4zb   1/1     Running   0          15s

# 指定 namespace 查看 pod 描述信息
[root@master ~]# kubectl describe pods pod-cbb995bbf-xb4zb -n dev
Name:         pod-cbb995bbf-xb4zb
Namespace:    dev
Priority:     0
Node:         node1/192.168.2.107
Start Time:   Tue, 27 Feb 2024 23:10:43 -0800
Labels:       pod-template-hash=cbb995bbf
              run=pod
Annotations:  <none>
Status:       Running
IP:           10.244.1.3
IPs:
  IP:           10.244.1.3
Controlled By:  ReplicaSet/pod-cbb995bbf
Containers:
  pod:
    Container ID:   docker://5437740803b92421c6a2278aae93eb0734270effc0510f1da94e9cc38777c08d
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 27 Feb 2024 23:10:58 -0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-929fh (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-929fh:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-929fh
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  51s   default-scheduler  Successfully assigned dev/pod-cbb995bbf-xb4zb to node1
  Normal  Pulling    50s   kubelet, node1     Pulling image "nginx:1.17.1"
  Normal  Pulled     37s   kubelet, node1     Successfully pulled image "nginx:1.17.1"
  Normal  Created    36s   kubelet, node1     Created container pod
  Normal  Started    36s   kubelet, node1     Started container pod

# 删除 pod 为 not found ,没有指定 namespace
[root@master ~]# kubectl delete pod pod-cbb995bbf-xb4zb
Error from server (NotFound): pods "pod-cbb995bbf-xb4zb" not found

# 指定 namespace 删除 pod
[root@master ~]# kubectl delete pod pod-cbb995bbf-xb4zb -n dev
pod "pod-cbb995bbf-xb4zb" deleted

# 删除 namespace dev 
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted

# 查看 namespace
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   24h
kube-flannel      Active   24h
kube-node-lease   Active   24h
kube-public       Active   24h
kube-system       Active   24h

```

### 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作 `kubernetes` 资源；

命令式对象配置的方式操作资源，可以简单的认为：命令+yaml 配置文件（yaml 是各种配置参数）

例如：

```shell
# 创建一个 nginxpod.yaml 文件
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:latest
    
# 创建一个 nginxpod.yaml 文件
[root@master Download]# vim nginxpod.yaml
[root@master Download]# ll
总用量 4
-rw-r--r-- 1 root root 199 2月  27 23:27 nginxpod.yaml

# 查看 nginxpod.yaml 文件
[root@master Download]# cat nginxpod.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:latest
# 执行create命令，创建资源（此时创建了两个资源对象，分别是namespace和pod）
[root@master Download]# kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 执行get命令，查看资源
[root@master Download]# kubectl get -f nginxpod.yaml
NAME            STATUS   AGE
namespace/dev   Active   62s

NAME           READY   STATUS    RESTARTS   AGE
pod/nginxpod   1/1     Running   0          61s

# 执行delete命令，删除资源
[root@master Download]# kubectl delete -f nginxpod.yaml
namespace "dev" deleted
pod "nginxpod" deleted

# 查看 ns pod
[root@master Download]# kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6867cdf567-72rjc   1/1     Running   0          24h
[root@master Download]# kubectl get ns
NAME              STATUS   AGE
default           Active   24h
kube-flannel      Active   24h
kube-node-lease   Active   24h
kube-public       Active   24h
kube-system       Active   24h

```

### 声明式对象配置

声明式对象配置跟命令是对象配置类似，但是只有一个命令 `apply`

```shell
# 执行yaml文件，创建资源
[root@master Download]# kubectl apply -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 查看
[root@master Download]# kubectl get ns dev
NAME   STATUS   AGE
dev    Active   13m

# 查看
[root@master Download]# kubectl get pods -n dev
NAME       READY   STATUS    RESTARTS   AGE
nginxpod   1/1     Running   0          13m


# 再次执行
#     其实声明式对象配置就是使用apply描述一个资源最终的状态（在yaml中定义状态）
#    使用apply操作资源：
#        如果资源不存在，就创建，相当于 kubectl create
 #       如果资源已存在，就更新，相当于 kubectl patch
[root@master Download]# kubectl apply -f nginxpod.yaml
namespace/dev unchanged
pod/nginxpod unchanged

# 通过create 创建，已存在
[root@master Download]# kubectl create -f nginxpod.yaml 
Error from server (AlreadyExists): error when creating "nginxpod.yaml": namespaces "dev" already exists
Error from server (AlreadyExists): error when creating "nginxpod.yaml": pods "nginxpod" already exists

```

> **PS：**`kubectl`可以在`node`节点上运行吗 ?
>
> ```shell
> [root@node2 ~]# kubectl get nodes
> The connection to the server localhost:8080 was refused - did you specify the right host or port?
> # 
> ```
>
> `kubectl`的运行是需要进行配置的，它的配置文件是`$HOME/.kube`，如果想要在`node`节点运行此命令，需要将`master`上的`.kube`文件复制到`node`节点上，即在`master`节点上执行下面操作：
>
> ```shell
> # scp -r ~/.kube/ node1:~/
> # scp -r ~/.kube/ node2:~/
> 
> 
> [root@master ~]# scp -r ~/.kube/ node1:~/
> root@node1's password: 
> config                                                                                   100% 5448     4.9MB/s   00:00    
> ee9dd967597b6ec53b14c2e5b3804177                                                         100%  244   296.9KB/s   00:00    
> ec0668be4b06b76488df73c61ec01d32                                                         100% 4308    75.7KB/s   00:00    
> 15c72d7a1f13477f649bb487fa4e6ef3                                                         100%  695   648.3KB/s   00:00    
> 386205621d02763e3b2585ac588098df                                                         100%  311   488.5KB/s   00:00    
> ffbf53d197f6cfe6efbfe6886003bc10                                                         100%  411   650.5KB/s   00:00    
> 472e20dd67b41740eccc3ff64519600a                                                         100%  753     1.3MB/s   00:00    
> f9246c6b5cbf686cd1130c6d7b7d537a                                                         100%  316   514.0KB/s   00:00    
> 690e2d9c028191515ce1773100dd43f8                                                         100%  417   699.8KB/s   00:00    
> ad4977a60b5b08ace824d34a5ef5d5fe                                                         100% 6021     9.4MB/s   00:00    
> a412fed29bbcb1b185ef58e1a456b03d                                                         100%  614   746.7KB/s   00:00    
> 643f50ffcddfa5275359b90c06aaf938                                                         100%  516   854.5KB/s   00:00    
> 66e51487acce593adbdc14b34a84f914                                                         100%  532   663.2KB/s   00:00    
> f2c9ad358ac899355c8ea2739a4feeb6                                                         100%  434   615.2KB/s   00:00    
> d06fda4fa584824cd9f261ac9236b6fc                                                         100%  439   794.9KB/s   00:00    
> da68d7270f61cf11cee1ad8c290ea4a3                                                         100%  398   700.3KB/s   00:00    
> d9d9c76707c7e5e59c5f9ab3f13308ad                                                         100%  700     1.3MB/s   00:00    
> 9fd6923991d13cc6f3eea4cbb4855044                                                         100%  613   523.5KB/s   00:00    
> 0267695836eb403239e341643393df1d                                                         100%  403   773.5KB/s   00:00    
> 5bda7c9f218794a4ae33bdf4330c1628                                                         100%  618   874.2KB/s   00:00    
> fb53b4fe89cfc2da20940de338b7d6a6                                                         100%  537   750.4KB/s   00:00    
> 2b330d6d96516ec65367ec910e249a9b                                                         100%  705     1.1MB/s   00:00    
> 8a9eb92e7ea7ab5dc875634da105225e                                                         100% 2306     3.8MB/s   00:00    
> da06049ab31c8e8264906abe930fd25c                                                         100%  700     1.3MB/s   00:00    
> 68c579fa42689ffb02e2b92f1a46839e                                                         100%  619   860.5KB/s   00:00    
> 264cbb7c28bc64f8e77868d45ff1cf4a                                                         100%  416   705.1KB/s   00:00    
> 39895f16a5e09723f378b7e4454991f3                                                         100%  813     1.5MB/s   00:00    
> 313da10740267fba780e5a531b42cb60                                                         100%  547     1.0MB/s   00:00    
> 992d7d20984d42448bf2e4e3d26fd5c6                                                         100%  969     2.0MB/s   00:00    
> e2c3a44d4400c9f5a7380286ed42f71c                                                         100%  618     1.1MB/s   00:00    
> 094d4d3b57e387e90363d5c356536d37                                                         100% 1029     1.2MB/s   00:00    
> c4ac2f461e1f1714820f526d820ba7fb                                                         100%  506   750.5KB/s   00:00    
> f4e183fa15527d62741806bff66f8e9f                                                         100%  437   787.9KB/s   00:00    
> abfcf3ed7d46087fef19fd0520a11f0b                                                         100% 1024    12.0KB/s   00:00    
> 3916b816306ff52319abc0e8b0e85b0b                                                         100% 1041   995.5KB/s   00:00    
> 4bcae4ba114c0c5dc52d2b255075190a                                                         100%  541   558.0KB/s   00:00    
> 6e71eb2fbf96a2c8669689838c4e1b99                                                         100% 2815KB  20.1MB/s   00:00    
> a56ea9d7441a1446f0d9814eceb3dbce                                                         100%  372   375.9KB/s   00:00    
> servergroups.json                                                                        100% 4319     5.6MB/s   00:00    
> serverresources.json                                                                     100%  586   556.1KB/s   00:00    
> serverresources.json                                                                     100%  591   914.6KB/s   00:00    
> serverresources.json                                                                     100%  202   255.9KB/s   00:00    
> serverresources.json                                                                     100%  207   417.1KB/s   00:00    
> serverresources.json                                                                     100% 2196     3.0MB/s   00:00    
> serverresources.json                                                                     100%  302   533.1KB/s   00:00    
> serverresources.json                                                                     100%  644     1.1MB/s   00:00    
> serverresources.json                                                                     100%  308   524.9KB/s   00:00    
> serverresources.json                                                                     100% 5932     9.4MB/s   00:00    
> serverresources.json                                                                     100%  505     2.9KB/s   00:00    
> serverresources.json                                                                     100%  510   832.5KB/s   00:00    
> serverresources.json                                                                     100%  425   647.8KB/s   00:00    
> serverresources.json                                                                     100%  423   743.3KB/s   00:00    
> serverresources.json                                                                     100%  428   779.0KB/s   00:00    
> serverresources.json                                                                     100%  325   586.8KB/s   00:00    
> serverresources.json                                                                     100%  330   597.3KB/s   00:00    
> serverresources.json                                                                     100%  289   528.0KB/s   00:00    
> serverresources.json                                                                     100%  294   512.6KB/s   00:00    
> serverresources.json                                                                     100%  591     1.0MB/s   00:00    
> serverresources.json                                                                     100%  596   998.4KB/s   00:00    
> serverresources.json                                                                     100%  504   909.8KB/s   00:00    
> serverresources.json                                                                     100%  509   961.2KB/s   00:00    
> serverresources.json                                                                     100%  509     1.0MB/s   00:00    
> serverresources.json                                                                     100%  307   576.8KB/s   00:00    
> serverresources.json                                                                     100%  704   618.7KB/s   00:00    
> serverresources.json                                                                     100%  438   400.1KB/s   00:00    
> serverresources.json                                                                     100%  397   411.7KB/s   00:00    
> serverresources.json                                                                     100%  860     1.5MB/s   00:00    
> serverresources.json                                                                     100%  932     1.6MB/s   00:00    
> serverresources.json                                                                     100%  920     1.7MB/s   00:00    
> serverresources.json                                                                     100%  915     1.6MB/s   00:00    
> serverresources.json                                                                     100%  328   633.2KB/s   00:00    
> serverresources.json                                                                     100%  432   828.2KB/s   00:00    
> 
> # 在 node1 、node2 节点上执行
> [root@node1 ~]# kubectl get nodes
> NAME     STATUS   ROLES    AGE   VERSION
> master   Ready    master   26h   v1.17.4
> node1    Ready    <none>   26h   v1.17.4
> node2    Ready    <none>   26h   v1.17.4
> [root@node2 ~]# kubectl get nodes
> NAME     STATUS   ROLES    AGE   VERSION
> master   Ready    master   26h   v1.17.4
> node1    Ready    <none>   26h   v1.17.4
> node2    Ready    <none>   26h   v1.17.4
> 
> ```
>
> > 使用推荐：三种方式应该怎么用？
> >
> > 创建/更新资源 使用声明式对象配置 `kubectl apply -f XXX.yaml`
> >
> > 删除资源 使用命令式对象配置 `kubectl delete -f XXX.yaml`
> >
> > 查询资源 使用命令式对象管理 `kubectl get(describe) `资源名称

## NameSpace

`NameSpace` 是`kubernetes` 系统中一种非常重要资源，主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**，比如 常用的 dev 环境，prod 环境，test 环境。

默认情况下，`kubernetes`集群中所有的`Pod`都是可以互相访问的，但是在实际中，如果不想让两个`Pod`之间互相访问，那此时就可以将两个`Pod`划分到不同的`namespace`下，`kubernetes`通过将集群内部的资源分配到不同的`NameSpace`中，可以形成逻辑上**组**，以方便不同的组的资源进行隔离使用和管理。

通过`kubernetes`的授权机制，将不同的`namespace`交给不同的租户进行管理，这样就实现了多租户的资源隔离。此时还能结合`kubernetes`的资源配额机制，限定不同租户能占用的资源，例如 CPU 使用量，内存使用量等等，来实现租户可用资源的管理。

<img src="../../Library/Application%20Support/typora-user-images/image-20240229100651208.png" alt="image-20240229100651208" style="zoom:80%;" />

> `kubernetes`在集群启动之后，会默认创建几个`namespace`
>
> ```shell
> [root@master Download]# kubectl get ns
> NAME              STATUS   AGE
> default           Active   42h # 所有未指定的NameSpace的对象都会被分配在 default 命名空间
> kube-flannel      Active   42h 
> kube-node-lease   Active   42h # 集群节点之间的心跳维护，从 V1.13 开始引入
> kube-public       Active   42h # 此命名空间下的资源可以被所有人访问（包括未认证用户）
> kube-system       Active   42h # 所有由 kubernetes 系统创建的资源都处于这个命名空间
> ```

```shell
# 查看所有的ns 命令 kubectl get ns
[root@master Download]# kubectl get ns
NAME              STATUS   AGE
default           Active   42h
kube-flannel      Active   42h
kube-node-lease   Active   42h
kube-public       Active   42h
kube-system       Active   42h

# 查看 系统pod
[root@master Download]# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-9d85f5447-dlvzc          1/1     Running   0          42h
coredns-9d85f5447-mf72b          1/1     Running   0          42h
etcd-master                      1/1     Running   0          42h
kube-apiserver-master            1/1     Running   0          42h
kube-controller-manager-master   1/1     Running   0          42h
kube-proxy-j2p4h                 1/1     Running   0          42h
kube-proxy-kkwzq                 1/1     Running   0          42h
kube-proxy-lr55p                 1/1     Running   0          42h
kube-scheduler-master            1/1     Running   0          42h

# 查看指定的ns   命令：kubectl get ns ns名称
[root@master Download]# kubectl get ns default
NAME      STATUS   AGE
default   Active   43h

# 指定输出格式  命令：kubectl get ns ns名称  -o 格式参数
# kubernetes支持的格式有很多，比较常见的是wide、json、yaml
[root@master Download]# kubectl get ns default -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2024-02-27T06:54:25Z"
  name: default
  resourceVersion: "145"
  selfLink: /api/v1/namespaces/default
  uid: 9f2f7a12-1302-426c-8e38-7766fa3fac37
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
  
# 查看ns详情  命令：kubectl describe ns ns名称
# Active 命名空间正在使用中  Terminating 正在删除命名空间
# ResourceQuota 针对 namespace 做的资源限制
# LimitRange 针对 namespace 中的每个组件做的资源限制
[root@master Download]# kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.

# 创建namespace
[root@master Download]# kubectl create ns dev
namespace/dev created

# 删除 namespace
[root@master Download]# kubectl delete ns dev
namespace "dev" deleted

# 创建 yaml文件：ns-dev.yaml
[root@master Download]# vim ns-dev.yaml
[root@master Download]# cat ns-dev.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  
# 创建 
[root@master Download]# kubectl create -f ns-dev.yaml 
namespace/dev created

# 删除
[root@master Download]# kubectl delete -f ns-dev.yaml 
namespace "dev" deleted
```

## Pod

`Pod`是 `kubernetes` 集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于`Pod`中。

`Pod`可以认为是容器的封装，一个`Pod`中可以存在一个 或 多个容器。

<img src="../../Library/Application%20Support/typora-user-images/image-20240229102116959.png" alt="image-20240229102116959" style="zoom:80%;" />

> `kubernetes`在集群启动之后，集群中的各个组件也都是以Pod方式运行的。可以通过下面命令查看：
>
> ```shell
> [root@master Download]# kubectl get pod -n kube-system
> NAME                             READY   STATUS    RESTARTS   AGE
> coredns-9d85f5447-dlvzc          1/1     Running   0          43h
> coredns-9d85f5447-mf72b          1/1     Running   0          43h
> etcd-master                      1/1     Running   0          43h
> kube-apiserver-master            1/1     Running   0          43h
> kube-controller-manager-master   1/1     Running   0          43h
> kube-proxy-j2p4h                 1/1     Running   0          43h
> kube-proxy-kkwzq                 1/1     Running   0          43h
> kube-proxy-lr55p                 1/1     Running   0          43h
> kube-scheduler-master            1/1     Running   0          43h
> ```
>
> `kubernetes`没有提供单独运行Pod的命令，都是通过Pod控制器来实现的
>
> ```shell
> # 创建 ns
> [root@master Download]# kubectl create ns dev
> namespace/dev created
> 
> # 命令格式： kubectl run (pod控制器名称) [参数] 
> # --image  指定Pod的镜像
> # --port   指定端口
> # --namespace  指定namespace
> [root@master Download]# kubectl run nginx --image=nginx:latest --port=80 --namespace dev 
> kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
> deployment.apps/nginx created
> ```
>
> 查看`Pod`基本信息
>
> ```shell
> # 查看Pod基本信息
> [root@master Download]# kubectl get pods -n dev
> NAME                    READY   STATUS              RESTARTS   AGE
> nginx-dd6b5d745-d5hbn   0/1     ContainerCreating   0          28s
> 
> # 查看Pod的详细信息
> [root@master Download]# kubectl describe pod nginx -n dev
> Name:         nginx-dd6b5d745-d5hbn
> Namespace:    dev
> Priority:     0
> Node:         node2/192.168.2.108
> Start Time:   Wed, 28 Feb 2024 18:27:48 -0800
> Labels:       pod-template-hash=dd6b5d745
>            run=nginx
> Annotations:  <none>
> Status:       Running
> IP:           10.244.2.5
> IPs:
> IP:           10.244.2.5
> Controlled By:  ReplicaSet/nginx-dd6b5d745
> Containers:
> nginx:
>  Container ID:   docker://3f5251fe0b5b5e047f43016bb5b2f6a2b21d424c464c376f097171a4cbf7453b
>  Image:          nginx:latest
>  Image ID:       docker-pullable://nginx@sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
>  Port:           80/TCP
>  Host Port:      0/TCP
>  State:          Running
>    Started:      Wed, 28 Feb 2024 18:28:25 -0800
>  Ready:          True
>  Restart Count:  0
>  Environment:    <none>
>  Mounts:
>    /var/run/secrets/kubernetes.io/serviceaccount from default-token-xnkdm (ro)
> Conditions:
> Type              Status
> Initialized       True 
> Ready             True 
> ContainersReady   True 
> PodScheduled      True 
> Volumes:
> default-token-xnkdm:
>  Type:        Secret (a volume populated by a Secret)
>  SecretName:  default-token-xnkdm
>  Optional:    false
> QoS Class:       BestEffort
> Node-Selectors:  <none>
> Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
>               node.kubernetes.io/unreachable:NoExecute for 300s
> Events:
> Type    Reason     Age   From               Message
> ----    ------     ----  ----               -------
> Normal  Scheduled  49s   default-scheduler  Successfully assigned dev/nginx-dd6b5d745-d5hbn to node2
> Normal  Pulling    48s   kubelet, node2     Pulling image "nginx:latest"
> Normal  Pulled     12s   kubelet, node2     Successfully pulled image "nginx:latest"
> Normal  Created    12s   kubelet, node2     Created container nginx
> Normal  Started    12s   kubelet, node2     Started container nginx
> ```
>
> 访问`Pod`
>
> ```shell
> [root@master Download]# kubectl get pods -n dev -o wide
> NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE    NOMINATED NODE   READINESS GATES
> nginx-dd6b5d745-d5hbn   1/1     Running   0          4m17s   10.244.2.5   node2   <none>           <none>
> 
> # 访问 Pod
> [root@master Download]# curl 10.244.2.5:80
> <!DOCTYPE html>
> <html>
> <head>
> <title>Welcome to nginx!</title>
> <style>
> html { color-scheme: light dark; }
> body { width: 35em; margin: 0 auto;
> font-family: Tahoma, Verdana, Arial, sans-serif; }
> </style>
> </head>
> <body>
> <h1>Welcome to nginx!</h1>
> <p>If you see this page, the nginx web server is successfully installed and
> working. Further configuration is required.</p>
> 
> <p>For online documentation and support please refer to
> <a href="http://nginx.org/">nginx.org</a>.<br/>
> Commercial support is available at
> <a href="http://nginx.com/">nginx.com</a>.</p>
> 
> <p><em>Thank you for using nginx.</em></p>
> </body>
> </html>
> ```
>
> 删除指定 `Pod`
>
> ```shell
> # 查看 namespace 下的 pods
> [root@master Download]# kubectl get pods -n dev -o wide
> NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE    NOMINATED NODE   READINESS GATES
> nginx-dd6b5d745-d5hbn   1/1     Running   0          7m2s   10.244.2.5   node2   <none>           <none>
> 
> # 删除指定Pod
> [root@master Download]# kubectl delete pod nginx-dd6b5d745-d5hbn -n dev
> pod "nginx-dd6b5d745-d5hbn" deleted
> 
> # 此时，显示删除Pod成功，但是再查询，发现又新产生了一个 
> # 这是因为当前Pod是由Pod控制器创建的，控制器会监控Pod状况，一旦发现Pod死亡，会立即重建
> # 此时要想删除Pod，必须删除Pod控制器
> [root@master Download]# kubectl get pods -n dev
> NAME                    READY   STATUS    RESTARTS   AGE
> nginx-dd6b5d745-2xp6d   1/1     Running   0          25s
> 
> # 查询当前 namespace 下的 Pod 控制器
> [root@master Download]# kubectl get deploy -n dev
> NAME    READY   UP-TO-DATE   AVAILABLE   AGE
> nginx   1/1     1            1           8m29s
> 
> # 删除 Pod 控制器的同时，也会删除下面所有的Pod
> [root@master Download]# kubectl delete deploy nginx -n dev
> deployment.apps "nginx" deleted
> 
> # 再次查询 Pod，Pod 已被删除
> [root@master Download]# kubectl get pods -n dev
> No resources found in dev namespace.
> ```
>
> 配置操作：
>
> ```shell
> # 创建 pod-nginx.yaml文件
> [root@master Download]# vim pod-nginx.yaml
> 
> # 查看 pod-nginx.yaml文件
> [root@master Download]# more pod-nginx.yaml 
> apiVersion: v1
> kind: Pod
> metadata:
>   name: nginx
>   namespace: dev
> spec:
>   containers:
>   - image: nginx:latest
>     name: pod
>     ports:
>     - name: nginx-port
>       containerPort: 80
>       protocol: TCP
>       
> # 创建
> [root@master Download]# kubectl create -f pod-nginx.yaml 
> pod/nginx created
> 
> # 删除
> [root@master Download]# kubectl delete -f pod-nginx.yaml 
> pod "nginx" deleted
> ```

> >
> >
> >**PS**：**为什么 No resources found in dev namespace ？？？？**

<img src="../../Library/Application%20Support/typora-user-images/image-20240229105150133.png" alt="image-20240229105150133" style="zoom:50%;" />

> > 创建 `pod`的同时 ，不会同时创建 `deploy` ，但是创建 `deploy` 一定会创建 `Pod` ，比如命令行创建

## Label

​	`Label`是`kubernetes`系统中的一个重要概念。它的作用就是在资源上添加标识，用来对它们进行区分和选择。

> `Label`的特点：
>
> > - 一个`Label`会以`key/value`键值对的形式附加到各种对象上，如`Node`、`Pod`、`Service`等等；
> > - 一个资源对象可以定义任意数量的`Label` ，同一个`Label`也可以被添加到任意数量的资源对象上去；
> > - `Label`通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除；

​	通过`Label`实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作。

> 一些常用 `Label`示例如下：
>
> > - 版本标签："version":"release", "version":"stable"......
> > - 环境标签："environment":"dev"，"environment":"test"，"environment":"pro"
> > - 架构标签："tier":"frontend"，"tier":"backend"

​	标签定义完毕之后，还要考虑到标签的选择，这就要使用到`Label Selector`，即：

​	`Label`用于给某个资源对象定义标识

​	`Label Selector`用于查询和筛选拥有某些标签的资源对象

​	当前有两种`Label Selector`：

- 基于等式的`Label Selector`

  `name = slave`: 选择所有包含`Label`中`key="name"`且`value="slave"`的对象

  `env != production`: 选择所有包括`Label`中的`key="env"`且`value`不等于`"production"`的对象

- 基于集合的`Label Selector`

  `name in (master, slave)`: 选择所有包含`Label`中的`key="name"`且`value="master"或"slave"`的对象

  `name not in (frontend)`: 选择所有包含`Label`中的`key="name"`且`value不等于"frontend"`的对象

  >  标签的选择条件可以使用多个，此时将多个`Label Selector`进行组合，使用逗号","进行分隔即可。例如：
  >
  > name = slave，env != production
  >
  > name not in (frontend)，env != production

  ### 命令方式

  ```shell
  # 创建 namespace
  [root@master Download]# kubectl create ns dev
  namespace/dev created
  
  # 创建 pod
  [root@master Download]# kubectl create -f pod-nginx.yaml 
  pod/nginx created
  
  [root@master Download]# more pod-nginx.yaml 
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    namespace: dev
  spec:
    containers:
    - image: nginx:latest
      name: pod
      ports:
      - name: nginx-port
        containerPort: 80
        protocol: TCP
  
  [root@master Download]# kubectl get service
  NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
  kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        46h
  nginx        NodePort    10.109.11.2   <none>        80:30211/TCP   46h
  
  # 查看 namespace dev中的 pods
  [root@master Download]# kubectl get pods -n dev
  NAME    READY   STATUS    RESTARTS   AGE
  nginx   1/1     Running   0          26s
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pod -n dev --show-labels
  NAME    READY   STATUS    RESTARTS   AGE   LABELS
  nginx   1/1     Running   0          49s   <none>
  
  # 为 namespace dev 中的 pod 资源打标签
  [root@master Download]# kubectl label pod nginx -n dev version=1.0
  pod/nginx labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pod -n dev --show-labels
  NAME    READY   STATUS    RESTARTS   AGE   LABELS
  nginx   1/1     Running   0          31m   version=1.0
  
  # 为 namespace dev 中的 pod 资源打标签
  [root@master Download]# kubectl label pod nginx -n dev tier=back
  pod/nginx labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pod -n dev --show-labels
  NAME    READY   STATUS    RESTARTS   AGE   LABELS
  nginx   1/1     Running   0          33m   tier=back,version=1.0
  
  # 为 pod 资源更新标签
  [root@master Download]# kubectl label pod nginx -n dev version=2.0 --overwrite
  pod/nginx labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pod -n dev --show-labels
  NAME    READY   STATUS    RESTARTS   AGE   LABELS
  nginx   1/1     Running   0          38m   tier=back,version=2.0
  
  # 新建两个 pod 资源
  # 修改 pod-nginx.yaml 中的 name: nginx2 name: nginx1
  [root@master Download]# vim pod-nginx.yaml 
  [root@master Download]# kubectl create -f pod-nginx.yaml 
  pod/nginx2 created
  
  [root@master Download]# more pod-nginx.yaml 
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx2
    namespace: dev
  spec:
    containers:
    - image: nginx:latest
      name: pod
      ports:
      - name: nginx-port
        containerPort: 80
        protocol: TCP
  
  # 查看 namespace dev中的 pods 
  [root@master Download]# kubectl get pods -n dev
  NAME     READY   STATUS              RESTARTS   AGE
  nginx    1/1     Running             0          50m
  nginx1   1/1     Running             0          10m
  nginx2   0/1     ContainerCreating   0          11s
  
  # 为 namespace dev 中的 pod 资源打标签
  [root@master Download]# kubectl label pod nginx2 version=2.1 -n dev
  pod/nginx2 labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pods -n dev --show-labels
  NAME     READY   STATUS    RESTARTS   AGE   LABELS
  nginx    1/1     Running   0          51m   version=2.0
  nginx1   1/1     Running   0          11m   version=1.0
  nginx2   1/1     Running   0          81s   version=2.1
  
  # 筛选标签
  [root@master Download]# kubectl get pod -n dev -l version=2.0 --show-labels
  NAME    READY   STATUS    RESTARTS   AGE   LABELS
  nginx   1/1     Running   0          52m   version=2.0
  
  # 筛选标签
  [root@master Download]# kubectl get pod -n dev -l version!=2.0 --show-labels
  NAME     READY   STATUS    RESTARTS   AGE     LABELS
  nginx1   1/1     Running   0          12m     version=1.0
  nginx2   1/1     Running   0          2m31s   version=2.1
  
  
  # 为 namespace dev 中的 pod 资源打标签
  [root@master Download]# kubectl label pod nginx tier=back -n dev
  pod/nginx labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pods -n dev --show-labels
  NAME     READY   STATUS    RESTARTS   AGE     LABELS
  nginx    1/1     Running   0          56m     tier=back,version=2.0
  nginx1   1/1     Running   0          16m     version=1.0
  nginx2   1/1     Running   0          5m54s   version=2.1
  
  # 删除标签
  [root@master Download]# kubectl label pod nginx tier- -n dev
  pod/nginx labeled
  
  # 查看 namespace dev 中的 pods的标签
  [root@master Download]# kubectl get pods -n dev --show-labels
  NAME     READY   STATUS    RESTARTS   AGE     LABELS
  nginx    1/1     Running   0          57m     version=2.0
  nginx1   1/1     Running   0          16m     version=1.0
  nginx2   1/1     Running   0          6m47s   version=2.1
  ```
  
  ### 配置方式
  
  ```shell
  # 执行命令 kubectl apply -f pod-nginx.yaml
  
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    namespace: dev
    # 标签
    labels:
      version: "3.0" 
      env: "test"
  spec:
    containers:
    - image: nginx:latest
      name: pod
      ports:
      - name: nginx-port
        containerPort: 80
        protocol: TCP
  ```
  

## Deployment

在`kubernetes`中，`Pod`是最小的控制单元，但是`kubernetes`很少直接控制`Pod`，一般都是通过`Pod`控制器来完成的。`Pod`控制器用于`Pod`的管理，确保`pod`资源符合预期的状态，当`pod`的资源出现故障时，会尝试进行重启或重建`pod`。

在`kubernetes`中`Pod`控制器的种类有很多，本章节介绍一种：`Deployment`

<img src="../../Library/Application%20Support/typora-user-images/image-20240229143235933.png" alt="image-20240229143235933" style="zoom:80%;" />

 ### 命令方式

```shell
[root@master Download]# kubectl delete ns dev
namespace "dev" deleted
[root@master Download]# kubectl get ns
NAME              STATUS   AGE
default           Active   47h
kube-flannel      Active   47h
kube-node-lease   Active   47h
kube-public       Active   47h
kube-system       Active   47h
[root@master Download]# kubectl create ns dev
namespace/dev created
[root@master Download]# kubectl get deployment,pods -n dev
No resources found in dev namespace.

# 命令格式: kubectl create deployment 名称  [参数] 
# --image  指定pod的镜像
# --port   指定端口
# --replicas  指定创建pod数量
# --namespace  指定namespace
# 这里用的是 run 如果是创建用的是 create
[root@master Download]# kubectl run nginx --image=nginx:1.17.1 --port=80 --replicas=3 --namespace=dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created

# 查看deployment的信息
[root@master Download]#  kubectl get deploy -n dev
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           10s

# UP-TO-DATE：成功升级的副本数量
# AVAILABLE：可用副本的数量
[root@master Download]# kubectl get deploy -n dev -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx   3/3     3            3           15s   nginx        nginx:1.17.1   run=nginx

# 查看 deployment，pods 的信息
[root@master Download]# kubectl get deployment,pods -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           39s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-64777cd554-clppv   1/1     Running   0          39s
pod/nginx-64777cd554-hf4mf   1/1     Running   0          39s
pod/nginx-64777cd554-ngsv5   1/1     Running   0          39s

# 查看 deploy 的详细信息
[root@master Download]# kubectl describe deploy nginx -n dev
Name:                   nginx
Namespace:              dev
CreationTimestamp:      Wed, 28 Feb 2024 22:45:43 -0800
Labels:                 run=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx
  Containers:
   nginx:
    Image:        nginx:1.17.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-64777cd554 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  109s  deployment-controller  Scaled up replica set nginx-64777cd554 to 3
  
# 查看所有 Pods 标签
[root@master Download]# kubectl get pods -n dev --show-labels
NAME                     READY   STATUS    RESTARTS   AGE    LABELS
nginx-64777cd554-clppv   1/1     Running   0          4m7s   pod-template-hash=64777cd554,run=nginx
nginx-64777cd554-hf4mf   1/1     Running   0          4m7s   pod-template-hash=64777cd554,run=nginx
nginx-64777cd554-ngsv5   1/1     Running   0          4m7s   pod-template-hash=64777cd554,run=nginx


[root@master Download]# kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
nginx-64777cd554-clppv   1/1     Running   0          5m32s
nginx-64777cd554-hf4mf   1/1     Running   0          5m32s
nginx-64777cd554-ngsv5   1/1     Running   0          5m32s

# 删除 
[root@master Download]# kubectl delete deploy nginx -n dev
deployment.apps "nginx" deleted

# 注意这里已经不是 Running 状态了，删除需要一定的时间
[root@master Download]# kubectl get pods -n dev
NAME                     READY   STATUS        RESTARTS   AGE
nginx-64777cd554-clppv   0/1     Terminating   0          5m55s
nginx-64777cd554-hf4mf   1/1     Terminating   0          5m55s
nginx-64777cd554-ngsv5   1/1     Terminating   0          5m55s
[root@master Download]# kubectl get pods -n dev
No resources found in dev namespace.

```

### 配置操作

```shell
# 创建deploy-nginx.yaml 文件
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
	# 创建副本数量
  replicas: 3
  # 选择器
  selector:
    matchLabels:
      run: nginx
  # Pod 模板
  template:
    metadata:
    	# 标签
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP

[root@master Download]# vim deploy-nginx.yaml
[root@master Download]# more deploy-nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP

[root@master Download]# kubectl create ns dev
namespace/dev created
[root@master Download]# kubectl create -f deploy-nginx.yaml 
deployment.apps/nginx created
[root@master Download]# kubectl get deploy,pods -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/3     3            2           19s

NAME                        READY   STATUS              RESTARTS   AGE
pod/nginx-dd6b5d745-5qr97   1/1     Running             0          19s
pod/nginx-dd6b5d745-ggbqf   1/1     Running             0          19s
pod/nginx-dd6b5d745-plrgg   0/1     ContainerCreating   0          19s
[root@master Download]# kubectl delete -f deploy-nginx.yaml 
deployment.apps "nginx" deleted
[root@master Download]# kubectl get deploy,pods -n dev
NAME                        READY   STATUS        RESTARTS   AGE
pod/nginx-dd6b5d745-5qr97   0/1     Terminating   0          51s
pod/nginx-dd6b5d745-ggbqf   0/1     Terminating   0          51s
pod/nginx-dd6b5d745-plrgg   0/1     Terminating   0          51s
[root@master Download]# kubectl get deploy,pods -n dev
No resources found in dev namespace.

```

## Service

​	通过上节课的学习，已经能够利用 `Deployment` 来创建一组 `Pod` 来提供具有高可用的服务。

​	虽然每个 `Pod` 都会分配一个 单独的 `Pod IP`，然后仍然存在两个问题：

- `Pod IP` 会随着`Pod`的重建产生变化；
- `Pod IP` 仅仅是进群内可见的虚拟`IP`，外部无法访问；

这样对于访问这个服务带来了难度。因此，`kubernetes`设计了`Service`来解决这个问题。

`Service`可以看作是一组同类`Pod`**对外的访问接口**。借助`Service`，应用可以方便地实现服务发现和负载均衡。

<img src="../../Library/Application%20Support/typora-user-images/image-20240229152212680.png" alt="image-20240229152212680" style="zoom:80%;" />

### 命令方式

**问题：**

```shell
[root@master Download]# kubectl get ns
NAME              STATUS   AGE
default           Active   2d
dev               Active   42m
kube-flannel      Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
[root@master Download]# more  deploy-nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP

[root@master Download]# kubectl create -f deploy-nginx.yaml 
deployment.apps/nginx created
[root@master Download]# kubectl get pods -n dev
NAME                    READY   STATUS              RESTARTS   AGE
nginx-dd6b5d745-4hhrc   1/1     Running             0          28s
nginx-dd6b5d745-dr9s2   0/1     ContainerCreating   0          28s
nginx-dd6b5d745-kzmds   1/1     Running             0          28s
[root@master Download]# kubectl get pods,deploy -n dev
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-dd6b5d745-4hhrc   1/1     Running   0          38s
pod/nginx-dd6b5d745-dr9s2   1/1     Running   0          38s
pod/nginx-dd6b5d745-kzmds   1/1     Running   0          38s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           38s
[root@master Download]# kubectl get pods -n dev -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
nginx-dd6b5d745-4hhrc   1/1     Running   0          86s   10.244.1.15   node1   <none>           <none>
nginx-dd6b5d745-dr9s2   1/1     Running   0          86s   10.244.2.16   node2   <none>           <none>
nginx-dd6b5d745-kzmds   1/1     Running   0          86s   10.244.2.15   node2   <none>           <none>

# 注意 10.244.1.15 是可以访问
[root@master Download]# curl 10.244.1.15
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 删掉其中一个 pod
[root@master Download]# kubectl delete pod nginx-dd6b5d745-4hhrc -n dev
pod "nginx-dd6b5d745-4hhrc" deleted

# 再次查看 IP
[root@master Download]# kubectl get pods -n dev -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
nginx-dd6b5d745-6svsc   1/1     Running   0          31s     10.244.1.16   node1   <none>           <none>
nginx-dd6b5d745-dr9s2   1/1     Running   0          2m59s   10.244.2.16   node2   <none>           <none>
nginx-dd6b5d745-kzmds   1/1     Running   0          2m59s   10.244.2.15   node2   <none>           <none>

# 无法访问，且容器IP 已变更
[root@master Download]# curl 10.244.1.15
curl: (7) Failed connect to 10.244.1.15:80; 没有到主机的路由

```

> **操作一：**创建集群内部可访问的 `Service`
>
> ```shell
> # 暴露 Service
> [root@master Download]# kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev
> service/svc-nginx1 exposed
> 
> # 查看service
> [root@master Download]# kubectl get service -n dev
> NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
> svc-nginx1   ClusterIP   10.104.58.211   <none>        80/TCP    28s
> 
> # 查看service，这里可以简写为 svc
> [root@master Download]# kubectl get svc -n dev
> NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
> svc-nginx1   ClusterIP   10.104.58.211   <none>        80/TCP    44s
> 
> # 这里产生了一个 ClusterIP，这就是 service 的 IP ，在 Service 的生命周期中，这个地址是不会变动的
> # 可以通过这个 IP 访问当前 service 对应的 POD（相当于负载均衡）
> [root@master Download]# curl 10.104.58.211:80
> <!DOCTYPE html>
> <html>
> <head>
> <title>Welcome to nginx!</title>
> <style>
> html { color-scheme: light dark; }
> body { width: 35em; margin: 0 auto;
> font-family: Tahoma, Verdana, Arial, sans-serif; }
> </style>
> </head>
> <body>
> <h1>Welcome to nginx!</h1>
> <p>If you see this page, the nginx web server is successfully installed and
> working. Further configuration is required.</p>
> 
> <p>For online documentation and support please refer to
> <a href="http://nginx.org/">nginx.org</a>.<br/>
> Commercial support is available at
> <a href="http://nginx.com/">nginx.com</a>.</p>
> 
> <p><em>Thank you for using nginx.</em></p>
> </body>
> </html>
> 
> ```
>
> **操作二：**创建集群外部也可访问的 Service
>
> ```shell
> # 上面创建的 Service 的 type 类型为 ClusterIP ，这个 ip 地址只用集群内部可访问
> # 如果需要创建外部也可以访问的 Service ，需要修改 type 为 NodePort
> [root@master Download]# kubectl expose deploy nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev
> service/svc-nginx2 exposed
> # 此时查看，会发现出现了NodePort类型的Service，而且有一对Port（80:32599/TCP）
> [root@master Download]# kubectl get svc svc-nginx2 -n dev -o wide
> NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
> svc-nginx2   NodePort   10.106.222.111   <none>        80:32599/TCP   39s   run=nginx
> 
> # 接下来就可以通过集群外的主机访问 节点IP:32599 访问服务了
> # 例如在的电脑主机上通过浏览器访问下面的地址
> http://192.168.2.87:32599
> ```
>
> **删除 Service**
>
> ```shell
> [root@master Download]# kubectl delete svc svc-nginx2 -n dev
> service "svc-nginx2" deleted
> 
> ```
>
> 

### 配置方式

```shell
# 创建 svc-nginx.yaml 文件
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  namespace: dev
spec:
  clusterIP: 10.109.179.231 #固定svc的内网ip
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP
```

```shell
# 执行 创建文件
kubectl create -f svc-nginx.yaml

# 执行 删除文件
kubectl delete -f svc-nginx.yaml
```

w
