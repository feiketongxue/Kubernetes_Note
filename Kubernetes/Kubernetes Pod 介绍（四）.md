# `Pod` 详解

## `Pod` 介绍

### `Pod` 结构

<img src="../Images/image-20240305112414222.png" alt="image-20240305112414222" style="zoom:80%;" />

>  每个`Pod` 中都可以包含一个或者多个容器，这些容器可以分为两类：
>
> - 用户程序所在的容器，数量可多可少
> - `Pause` 容器，每个`Pod`都会有一个根容器：作用有两个：
>   1. 以它为依据，评估整个`Pod`的**健康状态**
>   2. 在根容器上设置`IP`**地址**，其他容器都此`ip`（Pod IP）,以实现`Pod`内部的网络通信（这里是`Pod`内部的通讯，`Pod`之间的通讯采用哦虚拟二层网络技术来实现，当前环境用的`Flannel`）

### `Pod` 定义

> **资源清单**
>
> ```shell
> apiVersion: v1      #必选，版本号，例如v1
> kind: Pod       　  #必选，资源类型，例如 Pod
> metadata:       　  #必选，元数据
>   name: string      #必选，Pod名称
>   namespace: string #Pod所属的命名空间,默认为"default"
>   labels:       　　 #自定义标签列表
>     - name: string      　          
> spec:              #必选，Pod中容器的详细定义
>   containers:      #必选，Pod中容器列表
>   - name: string   #必选，容器名称
>     image: string  #必选，容器的镜像名称
>     imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 
>     command: [string]   #容器的启动命令列表，如不指定，使用打包时使用的启动命令
>     args: [string]      #容器的启动命令参数列表
>     workingDir: string  #容器的工作目录
>     volumeMounts:       #挂载到容器内部的存储卷配置
>     - name: string      #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
>       mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
>       readOnly: boolean #是否为只读模式
>     ports: 								#需要暴露的端口库号列表
>     - name: string        #端口的名称
>       containerPort: int  #容器需要监听的端口号
>       hostPort: int       #容器所在主机需要监听的端口号，默认与Container相同
>       protocol: string    #端口协议，支持TCP和UDP，默认TCP
>     env:   					#容器运行前需设置的环境变量列表
>     - name: string  #环境变量名称
>       value: string #环境变量的值
>     resources: 					#资源限制和请求的设置
>       limits:  					#资源限制的设置
>         cpu: string     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
>         memory: string  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
>       requests: 				#资源请求的设置
>         cpu: string     #Cpu请求，容器启动的初始可用数量
>         memory: string  #内存请求,容器启动的初始可用数量
>     lifecycle: 			#生命周期钩子
>         postStart: 	#容器启动后立即执行此钩子,如果执行失败,会根据重启策略进行重启
>         preStop: 		#容器终止前执行此钩子,无论结果如何,容器都会终止
>     livenessProbe:  #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器
>       exec:       　 #对Pod容器内检查方式设置为exec方式
>         command: [string]  #exec方式需要制定的命令或脚本
>       httpGet:      			 #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
>         path: string
>         port: number
>         host: string
>         scheme: string
>         HttpHeaders:
>         - name: string
>           value: string
>       tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
>          port: number
>        initialDelaySeconds: 0        #容器启动完成后首次探测的时间，单位为秒
>        timeoutSeconds: 0    　　     #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
>        periodSeconds: 0     　　     #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
>        successThreshold: 0
>        failureThreshold: 0
>        securityContext:
>          privileged: false
>   restartPolicy: [Always | Never | OnFailure]  #Pod的重启策略
>   nodeName: <string> #设置NodeName表示将该Pod调度到指定到名称的node节点上
>   nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上
>   imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
>   - name: string
>   hostNetwork: false   #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
>   volumes:   					 #在该pod上定义共享存储卷列表
>   - name: string    	 #共享存储卷名称 （volumes类型有很多种）
>     emptyDir: {}       #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
>     hostPath: string   #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
>       path: string     #Pod所在宿主机的目录，将被用于同期中mount的目录
>     secret:       　　　#类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
>       scretname: string  
>       items:     
>       - key: string
>         path: string
>     configMap:         #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
>       name: string
>       items:
>       - key: string
>         path: string
> ```
>
> 在`kubernetes` 中基本所有资源的一级属性都是一样的，主要包含 5 部分：
>
> - `apiverion` 版本，由 `kubernetes` 内部定义，可以用 `kubectl api-version` 查询
> - `kind` 版本，由`kubernetes`内部定义，可以用 `kubectl api-resouces`查询
> - `metadata`元数据，主要是资源标识和说明，常用的有 `name`、`namespace`、`labels`等
> - `spec`描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述
> - `status` 状态信息，里面的内容不需要定义，由 `kubernetes`自动生成
>
> 在上面的属性中，`spec` 是重点，常见的子属性：
>
> - `containers <[]Object>` 容器列表，用于定义容器的详细信息
> - `nodeName` 根据`nodeName` 的值将`Pod` 调度到指定的Node节点上
> - `nodeSelector <map[]>` 根据`NodeSelector`中定义的信息选择将该 `Pod` 调度到包含这些`label`的`Node`
> - `hostNetwork` 是否使用主机网络模式，默认为 `false`、如果设置为`true`，表示使用宿主机网络
> - `volumes <[]Object>` 存储卷，用于定义`Pod`上面挂载的存储信息
> - `restartPolicy`重启策略，表示`Pod`在遇到故障的时候处理策略

```shell
# 小提示：
# 在这里，可通过一个命令来查看每种资源的可配置项
# kubectl explain 资源类型		 查看某种资源可以配置的一级属性
# kubectl explain 资源类型.属性 查看属性的子属性

[root@master ~]#  kubectl explain pod
KIND:     Pod
VERSION:  v1
# 描述
DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
     
# 查看某种资源
[root@master ~]# kubectl explain pod.metadata
KIND:     Pod
VERSION:  v1

RESOURCE: metadata <Object>

DESCRIPTION:
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
 
     ObjectMeta is metadata that all persisted resources must have, which
     includes all objects users must create.

FIELDS:
   annotations  <map[string]string>
     Annotations is an unstructured key value map stored with a resource that
     may be set by external tools to store and retrieve arbitrary metadata. They
     are not queryable and should be preserved when modifying objects. More
     info: http://kubernetes.io/docs/user-guide/annotations

   clusterName  <string>
     The name of the cluster which the object belongs to. This is used to
     distinguish resources with same name and namespace in different clusters.
     This field is not set anywhere right now and apiserver is going to ignore
     it if set in create or update request.

   creationTimestamp    <string>
     CreationTimestamp is a timestamp representing the server time when this
     object was created. It is not guaranteed to be set in happens-before order
     across separate operations. Clients may not set this value. It is
     represented in RFC3339 form and is in UTC. Populated by the system.
     Read-only. Null for lists. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   deletionGracePeriodSeconds   <integer>
     Number of seconds allowed for this object to gracefully terminate before it
     will be removed from the system. Only set when deletionTimestamp is also
     set. May only be shortened. Read-only.

   deletionTimestamp    <string>
     DeletionTimestamp is RFC 3339 date and time at which this resource will be
     deleted. This field is set by the server when a graceful deletion is
     requested by the user, and is not directly settable by a client. The
     resource is expected to be deleted (no longer visible from resource lists,
     and not reachable by name) after the time in this field, once the
     finalizers list is empty. As long as the finalizers list contains items,
     deletion is blocked. Once the deletionTimestamp is set, this value may not
     be unset or be set further into the future, although it may be shortened or
     the resource may be deleted prior to this time. For example, a user may
     request that a pod is deleted in 30 seconds. The Kubelet will react by
     sending a graceful termination signal to the containers in the pod. After
     that 30 seconds, the Kubelet will send a hard termination signal (SIGKILL)
     to the container and after cleanup, remove the pod from the API. In the
     presence of network partitions, this object may still exist after this
     timestamp, until an administrator or automated process can determine the
     resource is fully terminated. If not set, graceful deletion of the object
     has not been requested. Populated by the system when a graceful deletion is
     requested. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   finalizers   <[]string>
     Must be empty before the object is deleted from the registry. Each entry is
     an identifier for the responsible component that will remove the entry from
     the list. If the deletionTimestamp of the object is non-nil, entries in
     this list can only be removed. Finalizers may be processed and removed in
     any order. Order is NOT enforced because it introduces significant risk of
     stuck finalizers. finalizers is a shared field, any actor with permission
     can reorder it. If the finalizer list is processed in order, then this can
     lead to a situation in which the component responsible for the first
     finalizer in the list is waiting for a signal (field value, external
     system, or other) produced by a component responsible for a finalizer later
     in the list, resulting in a deadlock. Without enforced ordering finalizers
     are free to order amongst themselves and are not vulnerable to ordering
     changes in the list.

   generateName <string>
     GenerateName is an optional prefix, used by the server, to generate a
     unique name ONLY IF the Name field has not been provided. If this field is
     used, the name returned to the client will be different than the name
     passed. This value will also be combined with a unique suffix. The provided
     value has the same validation rules as the Name field, and may be truncated
     by the length of the suffix required to make the value unique on the
     server. If this field is specified and the generated name exists, the
     server will NOT return a 409 - instead, it will either return 201 Created
     or 500 with Reason ServerTimeout indicating a unique name could not be
     found in the time allotted, and the client should retry (optionally after
     the time indicated in the Retry-After header). Applied only if Name is not
     specified. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#idempotency

   generation   <integer>
     A sequence number representing a specific generation of the desired state.
     Populated by the system. Read-only.

   labels       <map[string]string>
     Map of string keys and values that can be used to organize and categorize
     (scope and select) objects. May match selectors of replication controllers
     and services. More info: http://kubernetes.io/docs/user-guide/labels

   managedFields        <[]Object>
     ManagedFields maps workflow-id and version to the set of fields that are
     managed by that workflow. This is mostly for internal housekeeping, and
     users typically shouldn't need to set or understand this field. A workflow
     can be the user's name, a controller's name, or the name of a specific
     apply path like "ci-cd". The set of fields is always in the version that
     the workflow used when modifying the object.

   name <string>
     Name must be unique within a namespace. Is required when creating
     resources, although some resources may allow a client to request the
     generation of an appropriate name automatically. Name is primarily intended
     for creation idempotence and configuration definition. Cannot be updated.
     More info: http://kubernetes.io/docs/user-guide/identifiers#names

   namespace    <string>
     Namespace defines the space within each name must be unique. An empty
     namespace is equivalent to the "default" namespace, but "default" is the
     canonical representation. Not all objects are required to be scoped to a
     namespace - the value of this field for those objects will be empty. Must
     be a DNS_LABEL. Cannot be updated. More info:
     http://kubernetes.io/docs/user-guide/namespaces

   ownerReferences      <[]Object>
     List of objects depended by this object. If ALL objects in the list have
     been deleted, this object will be garbage collected. If this object is
     managed by a controller, then an entry in this list will point to this
     controller, with the controller field set to true. There cannot be more
     than one managing controller.

   resourceVersion      <string>
     An opaque value that represents the internal version of this object that
     can be used by clients to determine when objects have changed. May be used
     for optimistic concurrency, change detection, and the watch operation on a
     resource or set of resources. Clients must treat these values as opaque and
     passed unmodified back to the server. They may only be valid for a
     particular resource or set of resources. Populated by the system.
     Read-only. Value must be treated as opaque by clients and . More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#concurrency-control-and-consistency

   selfLink     <string>
     SelfLink is a URL representing this object. Populated by the system.
     Read-only. DEPRECATED Kubernetes will stop propagating this field in 1.20
     release and the field is planned to be removed in 1.21 release.

   uid  <string>
     UID is the unique in time and space value for this object. It is typically
     generated by the server on successful creation of a resource and is not
     allowed to change on PUT operations. Populated by the system. Read-only.
     More info: http://kubernetes.io/docs/user-guide/identifiers#uids
     
# 可以继续点击
[root@master ~]# kubectl explain pod.metadata.managedFields
KIND:     Pod
VERSION:  v1

RESOURCE: managedFields <[]Object>

DESCRIPTION:
     ManagedFields maps workflow-id and version to the set of fields that are
     managed by that workflow. This is mostly for internal housekeeping, and
     users typically shouldn't need to set or understand this field. A workflow
     can be the user's name, a controller's name, or the name of a specific
     apply path like "ci-cd". The set of fields is always in the version that
     the workflow used when modifying the object.

     ManagedFieldsEntry is a workflow-id, a FieldSet and the group version of
     the resource that the fieldset applies to.

FIELDS:
   apiVersion   <string>
     APIVersion defines the version of this resource that this field set applies
     to. The format is "group/version" just like the top-level APIVersion field.
     It is necessary to track the version of a field set because it cannot be
     automatically converted.

   fieldsType   <string>
     FieldsType is the discriminator for the different fields format and
     version. There is currently only one possible value: "FieldsV1"

   fieldsV1     <map[string]>
     FieldsV1 holds the first JSON version format as described in the "FieldsV1"
     type.

   manager      <string>
     Manager is an identifier of the workflow managing these fields.

   operation    <string>
     Operation is the type of operation which lead to this ManagedFieldsEntry
     being created. The only valid values for this field are 'Apply' and
     'Update'.

   time <string>
     Time is timestamp of when these fields were set. It should always be empty
     if Operation is 'Apply'
[root@master ~]# kubectl api-versions
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
discovery.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
[root@master ~]# kubectl api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
leases                                         coordination.k8s.io            true         Lease
endpointslices                                 discovery.k8s.io               true         EndpointSlice
events                            ev           events.k8s.io                  true         Event
ingresses                         ing          extensions                     true         Ingress
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
[root@master ~]# kubectl get pods -n dev 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-64777cd554-7snm4   1/1     Running   0          4d19h
nginx-64777cd554-sjpkm   1/1     Running   0          4d19h
nginx-64777cd554-vbjsd   1/1     Running   0          4d19h
[root@master ~]# kubectl get pods nginx-64777cd554-7snm4 -n dev -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-02-29T10:35:07Z"
  generateName: nginx-64777cd554-
  labels:
    pod-template-hash: 64777cd554
    run: nginx
  name: nginx-64777cd554-7snm4
  namespace: dev
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-64777cd554
    uid: 2eeca344-c9d5-4b5b-aa23-3a1f3de1d2b3
  resourceVersion: "442828"
  selfLink: /api/v1/namespaces/dev/pods/nginx-64777cd554-7snm4
  uid: ca2d66bc-a413-4544-be2e-574da2ef5faf
spec:
  containers:
  - image: nginx:1.17.1
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-bncww
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
  - name: default-token-bncww
    secret:
      defaultMode: 420
      secretName: default-token-bncww
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-02-29T10:35:07Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-02-29T10:35:08Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-02-29T10:35:08Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-02-29T10:35:07Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://a59191746d614341aab3a8a9dec085239d2efe373a4565a3f242e2e32ad307d6
    image: nginx:1.17.1
    imageID: docker-pullable://nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-02-29T10:35:08Z"
  hostIP: 192.168.2.108
  phase: Running
  podIP: 10.244.2.24
  podIPs:
  - ip: 10.244.2.24
  qosClass: BestEffort
  startTime: "2024-02-29T10:35:07Z"
  
# 查看描述信息
[root@master ~]# kubectl explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds        <integer>
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   affinity     <Object>
     If specified, the pod's scheduling constraints

   automountServiceAccountToken <boolean>
     AutomountServiceAccountToken indicates whether a service account token
     should be automatically mounted.

   containers   <[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

   dnsConfig    <Object>
     Specifies the DNS parameters of a pod. Parameters specified here will be
     merged to the generated DNS configuration based on DNSPolicy.

   dnsPolicy    <string>
     Set DNS policy for the pod. Defaults to "ClusterFirst". Valid values are
     'ClusterFirstWithHostNet', 'ClusterFirst', 'Default' or 'None'. DNS
     parameters given in DNSConfig will be merged with the policy selected with
     DNSPolicy. To have DNS options set along with hostNetwork, you have to
     specify DNS policy explicitly to 'ClusterFirstWithHostNet'.

   enableServiceLinks   <boolean>
     EnableServiceLinks indicates whether information about services should be
     injected into pod's environment variables, matching the syntax of Docker
     links. Optional: Defaults to true.

   ephemeralContainers  <[]Object>
     List of ephemeral containers run in this pod. Ephemeral containers may be
     run in an existing pod to perform user-initiated actions such as debugging.
     This list cannot be specified when creating a pod, and it cannot be
     modified by updating the pod spec. In order to add an ephemeral container
     to an existing pod, use the pod's ephemeralcontainers subresource. This
     field is alpha-level and is only honored by servers that enable the
     EphemeralContainers feature.

   hostAliases  <[]Object>
     HostAliases is an optional list of hosts and IPs that will be injected into
     the pod's hosts file if specified. This is only valid for non-hostNetwork
     pods.

   hostIPC      <boolean>
     Use the host's ipc namespace. Optional: Default to false.

   hostNetwork  <boolean>
     Host networking requested for this pod. Use the host's network namespace.
     If this option is set, the ports that will be used must be specified.
     Default to false.

   hostPID      <boolean>
     Use the host's pid namespace. Optional: Default to false.

   hostname     <string>
     Specifies the hostname of the Pod If not specified, the pod's hostname will
     be set to a system-defined value.

   imagePullSecrets     <[]Object>
     ImagePullSecrets is an optional list of references to secrets in the same
     namespace to use for pulling any of the images used by this PodSpec. If
     specified, these secrets will be passed to individual puller
     implementations for them to use. For example, in the case of docker, only
     DockerConfig type secrets are honored. More info:
     https://kubernetes.io/docs/concepts/containers/images#specifying-imagepullsecrets-on-a-pod

   initContainers       <[]Object>
     List of initialization containers belonging to the pod. Init containers are
     executed in order prior to containers being started. If any init container
     fails, the pod is considered to have failed and is handled according to its
     restartPolicy. The name for an init container or normal container must be
     unique among all containers. Init containers may not have Lifecycle
     actions, Readiness probes, Liveness probes, or Startup probes. The
     resourceRequirements of an init container are taken into account during
     scheduling by finding the highest request/limit for each resource type, and
     then using the max of of that value or the sum of the normal containers.
     Limits are applied to init containers in a similar fashion. Init containers
     cannot currently be added or removed. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

   nodeName     <string>
     NodeName is a request to schedule this pod onto a specific node. If it is
     non-empty, the scheduler simply schedules this pod onto that node, assuming
     that it fits resource requirements.

   nodeSelector <map[string]string>
     NodeSelector is a selector which must be true for the pod to fit on a node.
     Selector which must match a node's labels for the pod to be scheduled on
     that node. More info:
     https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

   overhead     <map[string]string>
     Overhead represents the resource overhead associated with running a pod for
     a given RuntimeClass. This field will be autopopulated at admission time by
     the RuntimeClass admission controller. If the RuntimeClass admission
     controller is enabled, overhead must not be set in Pod create requests. The
     RuntimeClass admission controller will reject Pod create requests which
     have the overhead already set. If RuntimeClass is configured and selected
     in the PodSpec, Overhead will be set to the value defined in the
     corresponding RuntimeClass, otherwise it will remain unset and treated as
     zero. More info:
     https://git.k8s.io/enhancements/keps/sig-node/20190226-pod-overhead.md This
     field is alpha-level as of Kubernetes v1.16, and is only honored by servers
     that enable the PodOverhead feature.

   preemptionPolicy     <string>
     PreemptionPolicy is the Policy for preempting pods with lower priority. One
     of Never, PreemptLowerPriority. Defaults to PreemptLowerPriority if unset.
     This field is alpha-level and is only honored by servers that enable the
     NonPreemptingPriority feature.

   priority     <integer>
     The priority value. Various system components use this field to find the
     priority of the pod. When Priority Admission Controller is enabled, it
     prevents users from setting this field. The admission controller populates
     this field from PriorityClassName. The higher the value, the higher the
     priority.

   priorityClassName    <string>
     If specified, indicates the pod's priority. "system-node-critical" and
     "system-cluster-critical" are two special keywords which indicate the
     highest priorities with the former being the highest priority. Any other
     name must be defined by creating a PriorityClass object with that name. If
     not specified, the pod priority will be default or zero if there is no
     default.

   readinessGates       <[]Object>
     If specified, all readiness gates will be evaluated for pod readiness. A
     pod is ready when all its containers are ready AND all conditions specified
     in the readiness gates have status equal to "True" More info:
     https://git.k8s.io/enhancements/keps/sig-network/0007-pod-ready%2B%2B.md

   restartPolicy        <string>
     Restart policy for all containers within the pod. One of Always, OnFailure,
     Never. Default to Always. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

   runtimeClassName     <string>
     RuntimeClassName refers to a RuntimeClass object in the node.k8s.io group,
     which should be used to run this pod. If no RuntimeClass resource matches
     the named class, the pod will not be run. If unset or empty, the "legacy"
     RuntimeClass will be used, which is an implicit class with an empty
     definition that uses the default runtime handler. More info:
     https://git.k8s.io/enhancements/keps/sig-node/runtime-class.md This is a
     beta feature as of Kubernetes v1.14.

   schedulerName        <string>
     If specified, the pod will be dispatched by specified scheduler. If not
     specified, the pod will be dispatched by default scheduler.

   securityContext      <Object>
     SecurityContext holds pod-level security attributes and common container
     settings. Optional: Defaults to empty. See type description for default
     values of each field.

   serviceAccount       <string>
     DeprecatedServiceAccount is a depreciated alias for ServiceAccountName.
     Deprecated: Use serviceAccountName instead.

   serviceAccountName   <string>
     ServiceAccountName is the name of the ServiceAccount to use to run this
     pod. More info:
     https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

   shareProcessNamespace        <boolean>
     Share a single process namespace between all of the containers in a pod.
     When this is set containers will be able to view and signal processes from
     other containers in the same pod, and the first process in each container
     will not be assigned PID 1. HostPID and ShareProcessNamespace cannot both
     be set. Optional: Default to false.

   subdomain    <string>
     If specified, the fully qualified Pod hostname will be
     "<hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>". If not
     specified, the pod will not have a domainname at all.

   terminationGracePeriodSeconds        <integer>
     Optional duration in seconds the pod needs to terminate gracefully. May be
     decreased in delete request. Value must be non-negative integer. The value
     zero indicates delete immediately. If this value is nil, the default grace
     period will be used instead. The grace period is the duration in seconds
     after the processes running in the pod are sent a termination signal and
     the time when the processes are forcibly halted with a kill signal. Set
     this value longer than the expected cleanup time for your process. Defaults
     to 30 seconds.

   tolerations  <[]Object>
     If specified, the pod's tolerations.

   topologySpreadConstraints    <[]Object>
     TopologySpreadConstraints describes how a group of pods ought to spread
     across topology domains. Scheduler will schedule pods in a way which abides
     by the constraints. This field is alpha-level and is only honored by
     clusters that enables the EvenPodsSpread feature. All
     topologySpreadConstraints are ANDed.

   volumes      <[]Object>
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes
```

