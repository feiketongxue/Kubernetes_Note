# `Pod`生命周期

一般将`Pod`对象从创建至终这段时间范围称为**Pod 生命周期**，主要包含以下过程：

- `Pod`创建过程
- 运行初始化容器（ init container ）过程
- 运行主容器 （ main container ）
  1. 容器启动后钩子 （ post start ）、容器终止前钩子（ pre stop ）
  2. 容器的存活性探测（liveness probe ）、就绪性探测（readiness probe ）

- `Pod`终止过程

<img src="../../Library/Application%20Support/typora-user-images/image-20240306142932633.png" alt="image-20240306142932633" style="zoom:80%;" />

在整个生命周期中，Pod会出现5种**状态**（**相位**），分别如下：

- 挂起（Pending）：`apiserver`已经创建了`pod`资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中
- 运行中（Running）：`pod`已经被调度至某节点，并且所有容器都已经被`kubelet`创建完成
- 成功（Succeeded）：`pod`中的所有容器都已经成功终止并且不会被重启
- 失败（Failed）：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了`非0值`的退出状态
- 未知（Unknown）：`apiserver`无法正常获取到`pod`对象的状态信息，通常由网络通信失败所导致

### 创建和终止

#### `Pod` 创建过程

<img src="../../Library/Application%20Support/typora-user-images/image-20240306152554541.png" alt="image-20240306152554541" style="zoom:80%;" />

1. 客户端通过`kubectl`或用户提交需要创建的`Pod`信息到`apiServer`
2. `apiServer`开始生成`Pod`对象的信息，并将信息存入`etcd`，返回确认信息至客户端
3. `apiServer`开始反映`etcd`中的`Pod`对象的变化，其他组件使用`Watch`机制来跟踪检查`apiServer`上的变动
4. `scheduler`发现有新的`Pod`对象要创建，开始为`Pod`分配主机并将结果信息更新至`apiServer`
5. `node`节点上的`kubectl`发现有`Pod`调度过来，尝试调用`docker`启动容器，并将结果返回至`apiServer`
6. `apiServer`将接收到的`Pod`状态信息存入`etch`中

#### `Pod`终止过程

1. 客户端向`apiServer`发送删除`Pod`对象的命令
2. `apiServer`中的`Pod`对象信息会随着时间的推移而更新，在宽限期内（ 默认 30s ）,`Pod`被视为`dead`
3. 将`Pod`标记为`terminating`状态
4. `kubectl`在监控到`Pod`对象转为`terminating`状态的同时启动`Pod`关闭过程
5. 端点控制器（`controller-manager`）监控到`Pod`兑现的关闭行为时，将其从所有匹配到此端点的`service`资源的端点列表中移除
6. 如果当前`Pod`对象定义了`preStop`钩子处理器，则在其标记为`terminating`后机会以同步的方式启动执行
7. `Pod`对象中的容器进程收到停止信号
8. 宽限期结束后，若`Pod`中还存在仍在运行的进程，那么`Pod`对象会疏导立即终止的信号
9. `kubectl`请求`apiServer`将此`Pod`资源宽限期设置为0，从而完成删除操作，此时，`Pod`对于用户已不可见

### 初始化容器

初始化容器时在`Pod`的主容器启动之前要运行的容器，主要是做一些主容器的前置工作，具有以下特征：

1. 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么`kubernetes`需要重启，直至成功完成
2. 初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面的一个才能运行

初始化容器有很多的应用场景，下面列出的事最常见的几个：

1. 提供主容器镜像中不具备的工具程序或自定义代码
2. 初始化容器要先于应用容器串行启动并运行完成，因此，可用于延后应用容器的启动直至以来的条件得到满足

举例：接下来做一个案例，模拟下面这个需求：

假设要以主容器来运行`nginx`,但是要求在运行`nginx`之前先能够连接上`mysql`和`redis`所在的服务器

为了简化测试，事先规定好`mysql`和`redis`服务器地址

```shell
# 创建 pod-initcontainer.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-initcontainer
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports: 
    - name: nginx-port
      containerPort: 80
  initContainers:
  - name: test-mysql
    image: busybox:1.30
    command: ['sh', '-c', 'until ping 192.168.2.14 -c 1 ; do echo waiting for mysql...; sleep 2; done;']
  - name: test-redis
    image: busybox:1.30
    command: ['sh', '-c', 'until ping 192.168.2.15 -c 1 ; do echo waiting for reids...; sleep 2; done;']
```

```sh
# 创建 pod
[root@master Download]# kubectl create ns dev
namespace/dev created
[root@master Download]# vim pod-initcontainer.yaml
[root@master Download]# kubectl create -f pod-initcontainer.yaml 
pod/pod-initcontainer created
[root@master Download]# kubectl get pods -n dev
NAME                READY   STATUS     RESTARTS   AGE
pod-initcontainer   0/1     Init:0/2   0          11s


[root@master Download]# ping 192.168.2.14
PING 192.168.2.14 (192.168.2.14) 56(84) bytes of data.
^C
--- 192.168.2.14 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms


[root@master Download]# ping 192.168.2.15
PING 192.168.2.15 (192.168.2.15) 56(84) bytes of data.
^C
--- 192.168.2.15 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2000ms

# 注意：再新开一个 shell，为当前服务器新增两个 ip，观察 pod的变化
[root@master ~]# ifconfig ens33:1 192.168.2.14 netmask 255.255.255.0 up
[root@master ~]# ifconfig ens33:1 192.168.2.15 netmask 255.255.255.0 up

# 查看 pod 状态
# 发现 pod 卡在启动第一个初始化容器过程中，后面的容器不会运行
[root@master Download]# kubectl get pod pod-initcontainer -n dev -w
NAME                READY   STATUS     RESTARTS   AGE
pod-initcontainer   0/1     Init:0/2   0          2m3s
pod-initcontainer   0/1     Init:1/2   0          2m54s
pod-initcontainer   0/1     Init:1/2   0          2m55s
pod-initcontainer   0/1     PodInitializing   0          3m7s
pod-initcontainer   1/1     Running           0          3m8s

```

### 钩子函数

钩子函数能够感知自身生命周期中的事件，并在相应的时刻到来时，运行用户指定的程序代码。

`kubernetes`在主容器的启动之后和停止之前提供了两个钩子函数：

- `post start`：容器创建之后执行，如果失败了会重启容器
- `pre stop`：容器终止之前执行，执行完成之后，容器将成功终止，在其完成之前会阻塞删除容器的操作

钩子处理器支持使用下面三种方式定义动作：

1. `Exec`命令：在容器内执行一次命令

```sh
……
  lifecycle:
    postStart: 
      exec:
        command:
        - cat
        - /tmp/healthy
……
```

2. `TCPSocket`：在当前容器尝试访问指定的`socket`

```sh
……      
  lifecycle:
    postStart:
      tcpSocket:
        port: 8080
……
```

3. `HTTPGet`：在当前容器中向某`url`发起`http`请求

```shell
……
  lifecycle:
    postStart:
      httpGet:
        path: / 						#URI地址
        port: 80 						#端口号
        host: 192.168.5.3 	#主机地址
        scheme: HTTP 				#支持的协议，http或者https
……
```

例子：接下来，以`Exec`方式为例，演示下钩子函数的使用

```shell
# 创建 pod-hook-exec.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-hook-exec
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    lifecycle:
      postStart: 
        exec: # 在容器启动的时候执行一个命令，修改掉nginx的默认首页内容
          command: ["/bin/sh", "-c", "echo postStart... > /usr/share/nginx/html/index.html"]
      preStop:
        exec: # 在容器停止之前停止nginx服务
          command: ["/usr/sbin/nginx","-s","quit"]
```

```sh
[root@master Download]# vim pod-hook-exec.yaml
[root@master Download]# kubectl create -f pod-hook-exec.yaml 
pod/pod-hook-exec created
[root@master Download]# kubectl get pods pod-hook-exec -n dev -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
pod-hook-exec   1/1     Running   0          25s   10.244.2.11   node2   <none>           <none>
[root@master Download]# curl 10.244.2.11:80
postStart...
```

### 容器探测

​	容器探测用于检测容器中应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么`kubernetes`就会吧该问题的实例“摘除”，不承担业务流量。`kubernetes`提供了两种探针来实现容器探测，分别是：

- `liveness probes`：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，K8s 会重启容器
- `readiness probes`：就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，K8s 不会转发流量

> `livenessProbe`决定是否重启容器，`readinessProbe`决定是否将请求转发给容器

以上两种探针，目前均支持三种探测方式：

- `Exec`命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则则不正常

  ```sh
  ……
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
  ……
  ```

- `TCPSoket`：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常

  ```shell
  ……      
    livenessProbe:
      tcpSocket:
        port: 8080
  ……
  ```

- `HTTPGet`：调用容器内`Web`应用的`URL`，如果返回的状态码在200-399之间，则认为程序正常，否则则不正常

  ```sh
  ……
    livenessProbe:
      httpGet:
        path: / 					# URI地址
        port: 80 					# 端口号
        host: 127.0.0.1 	# 主机地址
        scheme: HTTP 			# 支持的协议，http或者https
  ……
  ```

例如：下面以`liveness probes`为例：

##### 方式一：Exec

```shell
# 创建 pod-liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-exec
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: 
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      exec:
        command: ["/bin/cat","/tmp/hello.txt"] # 执行一个查看文件的命令
```

 

```shell
[root@master Download]# vim pod-liveness-exec.yaml
[root@master Download]# kubectl create -f pod-liveness-exec.yaml 
pod/pod-liveness-exec created

# 会发现一直发现 重启 restarts 
# 观察上面的信息就会发现nginx容器启动之后就进行了健康检查
# 检查失败之后，容器被kill掉，然后尝试进行重启（这是重启策略的作用，后面讲解）
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
[root@master Download]# kubectl get pod pod-liveness-exec -n dev
NAME                READY   STATUS    RESTARTS   AGE
pod-liveness-exec   1/1     Running   1          43s

# 查看描述
[root@master Download]# kubectl describe pod pod-liveness-exec -n dev
Name:         pod-liveness-exec
Namespace:    dev
Priority:     0
Node:         node1/192.168.2.107
Start Time:   Wed, 06 Mar 2024 02:19:01 -0800
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.8
IPs:
  IP:  10.244.1.8
Containers:
  nginx:
    Container ID:   docker://dd3dd70136f0b3f7a83653c4633f7090e796be06e2378957d0d56922a1f90252
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 06 Mar 2024 02:19:58 -0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 06 Mar 2024 02:19:28 -0800
      Finished:     Wed, 06 Mar 2024 02:19:57 -0800
    Ready:          True
    Restart Count:  2
    Liveness:       exec [/bin/cat /tmp/hello.txt] delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wvfmk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-wvfmk:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wvfmk
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  77s                default-scheduler  Successfully assigned dev/pod-liveness-exec to node1
  Normal   Pulled     21s (x3 over 77s)  kubelet, node1     Container image "nginx:1.17.1" already present on machine
  Normal   Killing    21s (x2 over 51s)  kubelet, node1     Container nginx failed liveness probe, will be restarted
  Normal   Created    20s (x3 over 77s)  kubelet, node1     Created container nginx
  Normal   Started    20s (x3 over 76s)  kubelet, node1     Started container nginx
  # 会发现是失败的
  Warning  Unhealthy  1s (x8 over 71s)   kubelet, node1     Liveness probe failed: /bin/cat: /tmp/hello.txt: No such file or directory

# 删除 pod
[root@master Download]# kubectl delete -f pod-liveness-exec.yaml 
pod "pod-liveness-exec" deleted

[root@master Download]# vim pod-liveness-exec.yaml 

# 修改文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-exec
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      exec:
      	# 修改
        command: ["/bin/ll","/tmp/"] 
        
[root@master Download]# kubectl create -f pod-liveness-exec.yaml 
pod/pod-liveness-exec created
[root@master Download]# kubectl get pods pod-liveness-exec -n dev
NAME                READY   STATUS    RESTARTS   AGE
pod-liveness-exec   1/1     Running   0          21s
```

##### 方式二：TCPSocket

```shell
# 创建 pod-liveness-tcpsocket.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-tcpsocket
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: 
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      tcpSocket:
      	 # 尝试访问8080端口
        port: 8080
```

```sh
[root@master Download]# vim pod-liveness-tcpsocket.yaml
[root@master Download]# kubectl create -f pod-liveness-tcpsocket.yaml
pod/pod-liveness-tcpsocket created

# 观察上面的信息，发现尝试访问8080端口,但是失败了
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
[root@master Download]# kubectl get pod pod-liveness-tcpsocket -n dev
NAME                     READY   STATUS    RESTARTS   AGE
pod-liveness-tcpsocket   1/1     Running   1          28s

# 查看Pod详情
[root@master Download]# kubectl describe pods pod-liveness-tcpsocket -n dev
Name:         pod-liveness-tcpsocket
Namespace:    dev
Priority:     0
Node:         node2/192.168.2.108
Start Time:   Wed, 06 Mar 2024 02:28:18 -0800
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.2.12
IPs:
  IP:  10.244.2.12
Containers:
  nginx:
    Container ID:   docker://5d45d8002420bfae8ffacf1235237ba57fd61016c0e7851cc4626b6a092ae409
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 06 Mar 2024 02:28:41 -0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 06 Mar 2024 02:28:19 -0800
      Finished:     Wed, 06 Mar 2024 02:28:40 -0800
    Ready:          True
    Restart Count:  1
    Liveness:       tcp-socket :8080 delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wvfmk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-wvfmk:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wvfmk
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  49s                default-scheduler  Successfully assigned dev/pod-liveness-tcpsocket to node2
  Normal   Killing    27s                kubelet, node2     Container nginx failed liveness probe, will be restarted
  Normal   Pulled     26s (x2 over 48s)  kubelet, node2     Container image "nginx:1.17.1" already present on machine
  Normal   Created    26s (x2 over 48s)  kubelet, node2     Created container nginx
  Normal   Started    26s (x2 over 48s)  kubelet, node2     Started container nginx
  Warning  Unhealthy  7s (x5 over 47s)   kubelet, node2     Liveness probe failed: dial tcp 10.244.2.12:8080: connect: connection refused

# 当然接下来，可以修改成一个可以访问的端口，比如80，再试，结果就正常了......
```

##### 方式三：HTTPGet

```shell
# 创建 pod-liveness-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:  					# 其实就是访问http://127.0.0.1:80/hello  
        scheme: HTTP 			# 支持的协议，http或者https
        port: 80 					# 端口号
        path: /hello 			# URI地址
```

```shell
[root@master Download]# vim pod-liveness-httpget.yaml
[root@master Download]# kubectl create -f pod-liveness-httpget.yaml
pod/pod-liveness-httpget created

# 观察上面信息，尝试访问路径，但是未找到,出现404错误
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
[root@master Download]# kubectl get pod pod-liveness-httpget -n dev
NAME                   READY   STATUS    RESTARTS   AGE
pod-liveness-httpget   1/1     Running   0          23s
[root@master Download]# kubectl get pod pod-liveness-httpget -n dev
NAME                   READY   STATUS    RESTARTS   AGE
pod-liveness-httpget   1/1     Running   1          33s

# 查看Pod详情
[root@master Download]# kubectl describe pod pod-liveness-httpget -n dev
Name:         pod-liveness-httpget
Namespace:    dev
Priority:     0
Node:         node2/192.168.2.108
Start Time:   Wed, 06 Mar 2024 02:31:58 -0800
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.2.13
IPs:
  IP:  10.244.2.13
Containers:
  nginx:
    Container ID:   docker://1ba5cb3d509b16840b0b8796b24a98ca449a64e7fc36bf3192df335a5eb8ad83
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 06 Mar 2024 02:32:53 -0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 06 Mar 2024 02:32:23 -0800
      Finished:     Wed, 06 Mar 2024 02:32:53 -0800
    Ready:          True
    Restart Count:  2
    Liveness:       http-get http://:80/hello delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wvfmk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-wvfmk:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wvfmk
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  77s                default-scheduler  Successfully assigned dev/pod-liveness-httpget to node2
  Normal   Pulled     22s (x3 over 77s)  kubelet, node2     Container image "nginx:1.17.1" already present on machine
  Normal   Created    22s (x3 over 77s)  kubelet, node2     Created container nginx
  Normal   Started    22s (x3 over 77s)  kubelet, node2     Started container nginx
  Normal   Killing    22s (x2 over 52s)  kubelet, node2     Container nginx failed liveness probe, will be restarted
  Warning  Unhealthy  2s (x8 over 72s)   kubelet, node2     Liveness probe failed: HTTP probe failed with statuscode: 404
  
# 当然接下来，可以修改成一个可以访问的路径path，比如/，再试，结果就正常了......
```

##### 至此，已经使用`liveness Probe`三种探针方式，但是查看`liveness Probe`的子属性，会发现除了这三种方式，还有一些其他的配置：

```shell
[root@master Download]# kubectl explain pod.spec.containers.livenessProbe
KIND:     Pod
VERSION:  v1

RESOURCE: livenessProbe <Object>

DESCRIPTION:
FIELDS:
   exec <Object>
   # 连续探测失败多少次才被认定为失败。默认是3。最小值是1
   failureThreshold     <integer>
   httpGet      <Object>
   # 容器启动后等待多少秒执行第一次探测
   initialDelaySeconds  <integer>
   # 执行探测的频率。默认是10秒，最小1秒
   periodSeconds        <integer>
   # 连续探测成功多少次才被认定为成功。默认是1
   successThreshold     <integer>
   tcpSocket    <Object>
   # 探测超时时间。默认1秒，最小1秒
   timeoutSeconds       <integer>
```

```shell
# 例子
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80 
        path: /
      initialDelaySeconds: 30 # 容器启动后30s开始探测
      timeoutSeconds: 5 # 探测超时时间为5s
```

##### 重启策略

在上面讲到，一旦容器探测出现了问题，`kubernetes`就会对容器所在的`Pod`进行重启，其实这是由重启策略决定的，`Pod`的重启策略有三种，分别如下：

- `Always`：容器失效时，自动重启该容器，也是默认值
- `OnFailure`：容器终止运行且退出码不为0时 重启
- `Never`：不管状态如何，都不重启该容器

> 重启策略适用于`Pod`对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由`kubectl`延迟一段时间后进行，且反复的重启操作的延迟时长依此为 10s、20s、40s、80s、160s、300s，300s 是最大延迟时长

```shell
# 创建 pod-restartpolicy.yaml 文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-restartpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80
        path: /hello
  restartPolicy: Never # 设置重启策略为Never
```



```shell
[root@master Download]# vim pod-restartpolicy.yaml
[root@master Download]# kubectl create -f pod-restartpolicy.yaml
pod/pod-restartpolicy created

# 多等一会，再观察 pod 的重启次数，发现一直是0，并未重启 
[root@master Download]# kubectl  get pods pod-restartpolicy -n dev
NAME                READY   STATUS    RESTARTS   AGE
pod-restartpolicy   1/1     Running   0          10s
[root@master Download]# kubectl  get pods pod-restartpolicy -n dev
NAME                READY   STATUS      RESTARTS   AGE
pod-restartpolicy   0/1     Completed   0          47s

# 查看 Pod 详情，发现 nginx 容器失败
[root@master Download]# kubectl  describe pods pod-restartpolicy  -n dev
Name:         pod-restartpolicy
Namespace:    dev
Priority:     0
Node:         node1/192.168.2.107
Start Time:   Wed, 06 Mar 2024 02:59:59 -0800
Labels:       <none>
Annotations:  <none>
Status:       Succeeded
IP:           10.244.1.10
IPs:
  IP:  10.244.1.10
Containers:
  nginx:
    Container ID:   docker://13ad65e44fb4e8a5549d64cb6c470ed9d8a529f90b7101f96ae7b25e2940374e
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 06 Mar 2024 03:00:00 -0800
      Finished:     Wed, 06 Mar 2024 03:00:26 -0800
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:80/hello delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wvfmk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-wvfmk:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wvfmk
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  54s                default-scheduler  Successfully assigned dev/pod-restartpolicy to node1
  Normal   Pulled     53s                kubelet, node1     Container image "nginx:1.17.1" already present on machine
  Normal   Created    53s                kubelet, node1     Created container nginx
  Normal   Started    53s                kubelet, node1     Started container nginx
  Warning  Unhealthy  27s (x3 over 47s)  kubelet, node1     Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    27s                kubelet, node1     Stopping container nginx

```

