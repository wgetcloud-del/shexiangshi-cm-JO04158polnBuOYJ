
副本集是什么？我们在前文中讲过什么是pod，简单来说pod就是k8s直接操作的基本单位。不了解的同学可以参考前文：


*k8s 实战 1 \-\-\-\- 初识 （https://github.com/jilodream/p/18245222）**k8s 实战 2 \-\-\-\- pod 基础 (https://github.com/jilodream/p/18284282\)**k8s 实战 3 \-\-\-\- 标签（https://github.com/jilodream/p/18293278 ）*


如果只是有pod，基本是可以满足我们普通开发学习的，但是这和我们直接使用docker容器区别就不是很大了，要知道k8s要做的事情就是提供一整套的管理部署系统，如果仅仅就是操作几个pod，就有点大材小用了。因此我们必须要学习一些更高阶的资源，才能利用到k8s提供的各种能力。比如副本集 ReplicaSet，部署Deployment等。今天我们来学习和使用下副本集 ReplicaSetreplica 美\[ˈreplɪkə] 复制品;仿制品;


 ReplcaSet字面意思就是复制品的集合，我们一般译作副本集。通常来说，为了高可用，我们会对每个微服务创建多个实例，为了管理方便，每个实例通常对应一个容器，(防盗连接：本文首发自https://github.com/jilodream/ )每个容器又常常对应一个pod，多个实例实际上对应的就是多个pod。在此基础上，k8s定义了一种更高级的资源称之为ReplicaSet，为的就是管理这个pod集合（副本集合）。比如我们要创建3个kuard服务的副本集，我们应该如何通过副本集创建呢？和直接定义pod一样，我们可以直接定义一个yaml模板，通过模板来声明我们要定义的副本集的各项特性如下：




```
 1 apiVersion: apps/v1
 2 kind: ReplicaSet
 3 metadata:
 4   name: kuardrs
 5 spec:
 6   replicas: 2
 7   selector:
 8     matchLabels:
 9       appRs: kuardrs
10   template:
11     metadata:
12       labels:
13         appRs: kuardrs
14         version: "2"
15     spec:
16       containers:
17         - ports:
18             - containerPort: 80
19           name: kuardpod1
20           image: "docker.io/library/kuard-amd64:blue"
```


这就是一个标准的ReplicaSet 模板。


和pod模板一样，我们依次来看每个键的含义：首先是apiVersion 表示当前选定的版本kind 表示类型，我们的类型是ReplicaSet，也就是副本集metadata 表示元数据，在这里我们定义新建的副本集名称叫：kuardrsspec 表示特性，在这里我们定义副本集的一些内部特性信息：spec.replicas 表示副本集中副本的个数spec.selector 表示副本集匹配的标签，也称之为标签选择器。这个需要详细说说。在前一节中，我们已经知道了什么是标签，它可以理解为定义到资源上边的一系列kv值，我们可以用标签进行匹配和筛选资源。而我们创建的副本集和pod，或者说未来的Deployment和副本集之间，都是松耦合的，并不是直接依赖的关系。换句话说创建好副本集之后，你可以随便增删pod，副本集本身是不会受影响的。但是副本集会通过标签，感知到当前所管理的pod是否符合自己的要求，当它发现多出来了匹配的pod，就会尝试删除pod。当它发现匹配到的pod，少于自己要求的pod时，就会尝试新增pod。因此直接说副本集是pod的一个集合其实并不贴切，更准确的说法应该是，副本集是用来定义和规范pod集合的一个更高维度的资源。而我们平常发现某一个pod运行的有一些问题时，又不想直接删除pod，又不想生产环境继续持有一个有问题的pod，我们就可以直接修改pod的标签，将pod从副本集管理范围中，剥离出来。然后再围绕有问题的pod进一步研究。而这些pod对于副本集来说，都是无状态的，替换谁都可以，只要符合副本集的标签匹配要求就可以。就像下边这个样子：


![](https://img2024.cnblogs.com/blog/704073/202412/704073-20241210150749967-573915719.jpg)


在本示例中，selector所匹配的资源就是符合标签：**appRs: kuardrs** 的pod。注意这里用matchLabels进行了匹配。spec.template是pod的模板定义，其中我们分别有pod元数据以及pod特性等节点。我们在pod的metadata节点中定义了pod要持有的两个标签(防盗连接：本文首发自https://github.com/jilodream/ )：appRs: kuardrsversion: "2"


在pod的spec特性节点中定义了pod中容器的镜像、名称、端口等。template中pod的定义和前文中声明pod基本类似，有兴趣的同学一定要看下前文，这里就不过多的赘述了。总而言之定义好的pod，包含容器、标签等信息，而我们定好的副本集又指定了要筛选的标签的信息，两者就通过标签很松散的连接到了一起。


定义好了副本集的yaml文件之后，我们来看看副本集的常规操作**创建副本集**


kubectl apply \-f xxx.yaml              xxx.yaml文件是副本集的声明文件执行效果如下：




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl apply -f kuard-rs.yaml 
replicaset.apps/kuardrs created
```


**查看副本集**


kubectl get replicaset ，注意replicaset也可以是replicasets/rs （以下不再重复），执行效果如下，这里查到的副本集就是刚刚创建的副本集：




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get replicaset
NAME      DESIRED   CURRENT   READY   AGE
kuardrs   2         2         2       57s
```


**查看副本集详情**


k describe replicasets  xxx    xxx 是副本集名称执行效果如下：




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl describe replicaset kuardrs
Name:         kuardrs
Namespace:    default
Selector:     appRs=kuardrs
Labels:       <none>
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  appRs=kuardrs
           version=2
  Containers:
   kuardpod1:
    Image:        docker.io/library/kuard-amd64:blue
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  3m30s  replicaset-controller  Created pod: kuardrs-q8ghw
  Normal  SuccessfulCreate  3m30s  replicaset-controller  Created pod: kuardrs-wd29m
```


我们可以从详情中看到副本集定义的各种属性和模板。以及关联出的pod的一些相关信息。


有时我们还需要根据**pod反向的找到它属于哪个副本集**，此时可以通过pod 中的yaml 文件查找到对应创建的副本集


在键ownerReferences下边可以看到，如下：




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
kuardrs-q8ghw   1/1     Running   0          4m23s
kuardrs-wd29m   1/1     Running   0          4m23s
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get pod kuardrs-q8ghw  -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 619b5de35bedd683352ce58bab8e6efd03e79fa4495a8ae89bae85f93ac7ed3f
    cni.projectcalico.org/podIP: 192.168.240.165/32
    cni.projectcalico.org/podIPs: 192.168.240.165/32
  creationTimestamp: "2024-12-10T07:13:15Z"
  generateName: kuardrs-
  labels:
    appRs: kuardrs
    version: "2"
  name: kuardrs-q8ghw
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: kuardrs
    uid: 8a91c42a-864a-4172-8653-373210a59eaa
  resourceVersion: "28299737"
  uid: 587be9bd-2d0a-402b-a3a4-72edbc3170db
spec:
  containers:
  - image: docker.io/library/kuard-amd64:blue
    imagePullPolicy: IfNotPresent
....省略....
```


如果我们**想调整副本的数量或者其他属性**


我们可以通过（1）k edit rs xxx  ，   xxx是副本集的名称（2）k apply \-f xxx.yaml ，xxx.yaml文件是副本集的声明文件（3）k scale rs xxx \-\-replicas\=新副本数量  ，xxx是副本集名称


我们直接展示最后一种(防盗连接：本文首发自https://github.com/jilodream/ )执行效果：




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get pod 
NAME            READY   STATUS    RESTARTS   AGE
kuardrs-q8ghw   1/1     Running   0          9m19s
kuardrs-wd29m   1/1     Running   0          9m19s
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl scale rs kuardrs --replicas=3
replicaset.apps/kuardrs scaled
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get pod 
NAME            READY   STATUS    RESTARTS   AGE
kuardrs-lhgtc   1/1     Running   0          8s
kuardrs-q8ghw   1/1     Running   0          10m
kuardrs-wd29m   1/1     Running   0          10m
```


**删除副本集**


k delete rs xxx  ，xxx是副本集的名称




```
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get rs
NAME      DESIRED   CURRENT   READY   AGE
kuardrs   3         3         3       29m
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl delete rs kuardrs
replicaset.apps "kuardrs" deleted
[root@iZ2ze3bpa0o5cw6gp42ry2Z learnRs]# kubectl get rs
No resources found in default namespace.
```


 本博客参考[豆荚加速器官网](https://baitenghuo.com)。转载请注明出处！
