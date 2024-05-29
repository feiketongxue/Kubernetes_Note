# `Pod`配置

> `Pod.spec.containers`属性，是`Pod`配置中最为关键的一项配置
>
> ```shell
> [root@master ~]# kubectl explain pod.spec.containers
> KIND:     Pod
> VERSION:  v1
> 
> # 数组 ，代表多个容器
> RESOURCE: containers <[]Object>
> # 描述
> DESCRIPTION:
> FIELDS:
> 	 # 容器的启动命令需要的参数列表
>    args <[]string>
>    # 容器启动命令命令列表，如不指定，使用打包时使用的启动命令
>    command      <[]string>
> 	 # 容器环境变量的配置
>    env  <[]Object>
>    envFrom      <[]Object>
>    # 容器需要的镜像地址
>    image        <string>
> 	 # 镜像拉去策略
>    imagePullPolicy      <string>
>    lifecycle    <Object>
>    livenessProbe        <Object>
> 	 # 容器名称
>    name <string> -required-
> 	 # 容器需要暴漏的端口号列表
>    ports        <[]Object>
> 	 # 
>    readinessProbe       <Object>
>    # 资源限制和资源请求设置
>    resources    <Object>
>    securityContext      <Object>
>    startupProbe <Object>
>    stdin        <boolean>
>    stdinOnce    <boolean>
>    terminationMessagePath       <string>
>    terminationMessagePolicy     <string>
> 
>    tty  <boolean>
>    volumeDevices        <[]Object>
>    volumeMounts <[]Object>
>    workingDir   <string>
> 
> ```

### 基本配置

```shell
# 创建 pod-base.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: heima
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox
    image: busybox:1.30
```

上面定义了一个比较简单的Pod 的配置，里面有两个容器：

- `nginx`：用`1.17.1`版本的 `nginx` 镜像创建，（`nginx` 是一个轻量级 `web` 容器）
- busybox：用`1.30` 版本的`busybox`镜像创建，（`busybox 是一个小巧的linux 命令集合`）

```shell
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
[root@master ~]# kubectl get ns 
NAME              STATUS   AGE
default           Active   6d23h
kube-flannel      Active   6d23h
kube-node-lease   Active   6d23h
kube-public       Active   6d23h
kube-system       Active   6d23h
[root@master ~]# kubectl create ns dev
namespace/dev created
[root@master ~]# vim pod-base.yaml
[root@master ~]# mv pod-base.yaml /root/Download/
[root@master ~]# cd Download/

# 创建Pod
[root@master Download]# kubectl create -f pod-base.yaml 
pod/pod-base created
[root@master Download]# kubectl get pods -n dev
NAME       READY   STATUS              RESTARTS   AGE
pod-base   0/2     ContainerCreating   0          10s
[root@master Download]# kubectl get pods -n dev
NAME       READY   STATUS             RESTARTS   AGE
pod-base   1/2     CrashLoopBackOff   1          30s

# 查看Pod状况
# READY 1/2 : 表示当前Pod中有2个容器，其中1个准备就绪，1个未就绪
# RESTARTS  : 重启次数，因为有1个容器故障了，Pod一直在重启试图恢复它
[root@master Download]# kubectl get pods -n dev
NAME       READY   STATUS    RESTARTS   AGE
pod-base   1/2     Running   2          44s

# 通过 describe 查看内部的详情
# 此时已经运行起来了一个基本的 Pod，虽然它暂时有问题
[root@master Download]# kubectl describe pods pod-base -n dev
Name:         pod-base
Namespace:    dev
Priority:     0
Node:         node2/192.168.2.108
Start Time:   Mon, 04 Mar 2024 22:28:38 -0800
Labels:       user=heima
Annotations:  <none>
Status:       Running
IP:           10.244.2.25
IPs:
  IP:  10.244.2.25
Containers:
  nginx:
    Container ID:   docker://29d7c500893ee9221cd683c3be55c6e821d208557b4f46cd5cb0b1dcf58df510
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 04 Mar 2024 22:28:39 -0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2k89b (ro)
  busybox:
    Container ID:   docker://bb5645cbdb2b09819261b35a64d81feb517e8dee52e0fc80d4012cc64bbc0e91
    Image:          busybox:1.30
    Image ID:       docker-pullable://busybox@sha256:4b6ad3a68d34da29bf7c8ccb5d355ba8b4babcad1f99798204e7abb43e54ee3d
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 04 Mar 2024 22:30:40 -0800
      Finished:     Mon, 04 Mar 2024 22:30:40 -0800
    Ready:          False
    Restart Count:  4
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2k89b (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-2k89b:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2k89b
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m45s                default-scheduler  Successfully assigned dev/pod-base to node2
  Normal   Pulled     2m44s                kubelet, node2     Container image "nginx:1.17.1" already present on machine
  Normal   Created    2m44s                kubelet, node2     Created container nginx
  Normal   Started    2m44s                kubelet, node2     Started container nginx
  Normal   Pulling    2m44s                kubelet, node2     Pulling image "busybox:1.30"
  Normal   Pulled     2m22s                kubelet, node2     Successfully pulled image "busybox:1.30"
  Normal   Created    98s (x4 over 2m22s)  kubelet, node2     Created container busybox
  Normal   Started    97s (x4 over 2m22s)  kubelet, node2     Started container busybox
  Warning  BackOff    59s (x8 over 2m20s)  kubelet, node2     Back-off restarting failed container
  Normal   Pulled     44s (x4 over 2m21s)  kubelet, node2     Container image "busybox:1.30" already present on machine

```

### 镜像拉取

 创建 pod-imagepullpollicy.yaml 文件
```java
apiVersion: v1
kind: Pod
metadata:
  name: pod-imagepullpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    # 用于设置镜像拉取策略
    imagePullPolicy: Never 
  - name: busybox
    image: busybox:1.30
```

> `imagePullPolicy`用于设置镜像拉去策略，`kubernetes`支持配置三种拉去策略：
>
> - `Always`：总是从远程仓库拉取镜像（一直远程下载）
> - `IfNotPresent`：本地有责使用本地镜像，本地没有责从远程仓库拉取镜像
> - `Never`：使用本地镜像，从不远程仓库拉取，一直使用本地
>
> > 默认值说明：
> >
> > ​	如果镜像 tag 为具体版本号，比如 image: nginx:1.17.1 这种，在不指定策略的情况下 ，默认策略是：`IfNotPresent`
> >
> > ​	如果镜像 tag 为：latest （最终版本），默认策略是`always`

### 启动命令

在前面的案例中，一只有一个问题没有解决，就是`busybox` 容器一只没有成功运行，那么到底是什么原因导致这个容器的故障呢？

`busybox`容器并不是一个程序，而是类似于工具类的集合，`kubernetes`集群启动管理后，回自动关闭，解决方法就是让其一直在运行，这就用到了 `command`配置

```shell
# 创建 pod-command.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
```

```shell
[root@master Download]# vim pod-command.yml
[root@master Download]# kubectl create -f pod-command.yml 
pod/pod-command created
[root@master Download]# kubectl get pods -n dev
NAME          READY   STATUS              RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff    22         91m
pod-command   0/2     ContainerCreating   0          10s
[root@master Download]# kubectl get pods -n dev
NAME          READY   STATUS             RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff   22         92m
pod-command   2/2     Running            0          79s

# 进入pod中的busybox容器，查看文件内容
# 补充一个命令: kubectl exec  pod名称 -n 命名空间 -it -c 容器名称 /bin/sh  在容器内部执行命令
# 使用这个命令就可以进入某个容器的内部，然后进行相关操作了
# 比如，可以查看txt文件的内容
[root@master Download]# kubectl exec pod-command -n dev -it -c busybox /bin/sh
/ # tail -f /tmp/hello.txt 
09:18:41
09:18:44
09:18:47
^C
/ # exit
command terminated with exit code 130

```

> `command` 用于在`pod`中的容器初始化完毕后运行一个命令
>
> 稍微解释一下上面的命令
>
> "/bin/sh","-c",    使用`sh` 执行命令
>
> touch /tmp/hello.txt;      创建 /tmp/hello.txt 文件
>
> while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;  每隔3秒向文件中写入当前时间

```shell
特别说明：
    通过上面发现command已经可以完成启动命令和传递参数的功能，为什么这里还要提供一个args选项，用于传递参数呢?这其实跟docker有点关系，kubernetes中的command、args两项其实是实现覆盖Dockerfile中ENTRYPOINT的功能。
 1 如果command和args均没有写，那么用Dockerfile的配置。
 2 如果command写了，但args没有写，那么Dockerfile默认的配置会被忽略，执行输入的command
 3 如果command没写，但args写了，那么Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前args的参数
 4 如果command和args都写了，那么Dockerfile的配置被忽略，执行command并追加上args参数
```

### 环境变量

```shell
# 创建 pod-env.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","while true;do /bin/echo $(date +%T);sleep 60; done;"]
    env: # 设置环境变量列表
    - name: "username"
      value: "admin"
    - name: "password"
      value: "123456"
```

```shell
# env，环境变量，用于在pod中的容器设置环境变量。
[root@master Download]# vim pod-env.yaml 
[root@master Download]# kubectl create -f pod-env.yaml
pod/pod-env created
[root@master Download]# kubectl get pods -n dev
NAME          READY   STATUS             RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff   38         174m
pod-command   2/2     Running            0          83m
pod-env       1/1     Running            0          16s

# 进入容器，输出环境变量
[root@master Download]# kubectl exec pod-env -n dev -c busybox -it /bin/sh
/ # echo $username
admin
/ # echo $password
123456

```

> 这种方式不是很推荐，推荐将这些配置单独存储在配置文件中，这种方式将在后面介绍。

### 端口设置

介绍容器的端口设置，也就是 `containers`的`ports`选项

```shell
# ports 支持的子选项：
[root@k8s-master01 ~]# kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1
RESOURCE: ports <[]Object>
FIELDS:
   name         <string>  # 端口名称，如果指定，必须保证name在pod中是唯一的		
   containerPort<integer> # 容器要监听的端口(0<x<65536)
   hostPort     <integer> # 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本(一般省略) 
   hostIP       <string>  # 要将外部端口绑定到的主机IP(一般省略)
   protocol     <string>  # 端口协议。必须是UDP、TCP或SCTP。默认为“TCP”。
```

```shell
[root@master ~]# kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1

RESOURCE: ports <[]Object>

DESCRIPTION:
     List of ports to expose from the container. Exposing a port here gives the
     system additional information about the network connections a container
     uses, but is primarily informational. Not specifying a port here DOES NOT
     prevent that port from being exposed. Any port which is listening on the
     default "0.0.0.0" address inside a container will be accessible from the
     network. Cannot be updated.

     ContainerPort represents a network port in a single container.

FIELDS:
   containerPort        <integer> -required-
     Number of port to expose on the pod's IP address. This must be a valid port
     number, 0 < x < 65536.

   hostIP       <string>
     What host IP to bind the external port to.

   hostPort     <integer>
     Number of port to expose on the host. If specified, this must be a valid
     port number, 0 < x < 65536. If HostNetwork is specified, this must match
     ContainerPort. Most containers do not need this.

   name <string>
     If specified, this must be an IANA_SVC_NAME and unique within the pod. Each
     named port in a pod must have a unique name. Name for the port that can be
     referred to by services.

   protocol     <string>
     Protocol for port. Must be UDP, TCP, or SCTP. Defaults to "TCP".
# 创建 pod-ports.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: # 设置容器暴露的端口列表
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

```shell
[root@master Download]# vim pod-ports.yaml
[root@master Download]# kubectl get pods -n dev
NAME          READY   STATUS             RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff   39         179m
pod-command   2/2     Running            0          88m
pod-env       1/1     Running            0          4m57s
[root@master Download]# kubectl create -f pod-ports.yaml 
pod/pod-ports created
[root@master Download]# kubectl get pods -n dev
NAME          READY   STATUS             RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff   40         3h2m
pod-command   2/2     Running            0          90m
pod-env       1/1     Running            0          7m32s
pod-ports     1/1     Running            0          4s
[root@master Download]# kubectl get pod pod-ports -n dev -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-03-05T09:30:47Z"
  name: pod-ports
  namespace: dev
  resourceVersion: "1470670"
  selfLink: /api/v1/namespaces/dev/pods/pod-ports
  uid: 520583ac-41b7-411d-b7b3-7938feb0fc21
spec:
  containers:
  - image: nginx:1.17.1
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    # 访问容器中的程序需要使用的是`Podip:containerPort`
    - containerPort: 80
      name: nginx-port
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-2k89b
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
  - name: default-token-2k89b
    secret:
      defaultMode: 420
      secretName: default-token-2k89b
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-03-05T09:30:47Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-03-05T09:30:49Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-03-05T09:30:49Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-03-05T09:30:47Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://312641836aa102a9cb4104cd01e5d0c2337bbf7228843c6a8e54170124a4cbba
    image: nginx:1.17.1
    imageID: docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-03-05T09:30:48Z"
  hostIP: 192.168.2.108
  phase: Running
  podIP: 10.244.2.27
  podIPs:
  - ip: 10.244.2.27
  qosClass: BestEffort
  startTime: "2024-03-05T09:30:47Z"

```

### 资源配额

​	容器中的程序要运行，肯定是要占用一定资源的，比如`cpu`和`内存`等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其它容器无法运行。针对这种情况，`kubernetes`提供了对`内存`和`cpu`的资源进行配额的机制，这种机制主要通过`resources`选项实现，可以通过以下两个选项设置资源的上下限：

- `limits`：用于限制运行时容器的最大占用资源，当容器占用资源超过`limits`时会被终止，并进行重启
- `requests` ：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

```shell
# 创建 pod-resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-resources
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    resources: 					# 资源配额
      limits: 				  # 限制资源（上限）
        cpu: "2" 				# CPU限制，单位是core数 
        memory: "10Gi"  # 内存限制
      requests: 				# 请求资源（下限）
        cpu: "1"  			# CPU限制，单位是core数
        memory: "10Mi"  # 内存限制
# 说明： 
# cpu：core数，可以为整数或小数
# memory： 内存大小，可以使用Gi、Mi、G、M等形式
```

```shell
# 创建 pod 文件
[root@master Download]# vim pod-resources.yaml
# 创建 pod 
[root@master Download]# kubectl create -f pod-resources.yaml 
pod/pod-resources created

# 查看发现pod运行正常
[root@master Download]# kubectl get pod pod-resources -n dev
NAME            READY   STATUS    RESTARTS   AGE
pod-resources   1/1     Running   0          14s
[root@master Download]# kubectl delete -f pod-resources.yaml 
pod "pod-resources" deleted

# 编辑 pod，修改 resources.requests.memory的值为 10Gi
[root@master Download]# vim pod-resources.yaml

# # 再次启动 pod
[root@master Download]# kubectl create -f pod-resources.yaml 
pod/pod-resources created

# # 查看 Pod 状态，发现 Pod 启动失败,status 为Pending
[root@master Download]# kubectl get pods pod-resources -n dev 
NAME            READY   STATUS    RESTARTS   AGE
pod-resources   0/1     Pending   0          21s
[root@master Download]# kubectl get pods pod-resources -n dev -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
pod-resources   0/1     Pending   0          33s   <none>   <none>   <none>           <none>

# 查看 pod 详情会发现，如下提示
[root@master Download]# kubectl describe pod pod-resources -n dev
Name:         pod-resources
Namespace:    dev
Priority:     0
Node:         <none>
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:           
IPs:          <none>
Containers:
  nginx:
    Image:      nginx:1.17.1
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     2
      memory:  10Gi
    Requests:
      cpu:        1
      memory:     10Gi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2k89b (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  default-token-2k89b:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2k89b
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  55s   default-scheduler  0/3 nodes are available: 3 Insufficient memory.
```

