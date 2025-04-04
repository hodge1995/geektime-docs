你好，我是Chrono。

经过两个星期的学习，到今天我们的“初级篇”也快要结束了。

和之前的“入门篇”一样，在这次课里，我也会对前面学过的知识做一个比较全面的回顾，毕竟Kubernetes领域里有很多新名词、新术语、新架构，知识点多且杂，这样的总结复习就更有必要。

接下来我还是先简要列举一下“初级篇”里讲到的Kubernetes要点，然后再综合运用这些知识，演示一个实战项目——还是搭建WordPress网站，不过这次不是在Docker里，而是在Kubernetes集群里。

## Kubernetes技术要点回顾

容器技术开启了云原生的大潮，但成熟的容器技术，到生产环境的应用部署的时候，却显得“步履维艰”。因为容器只是针对单个进程的隔离和封装，而实际的应用场景却是要求许多的应用进程互相协同工作，其中的各种关系和需求非常复杂，在容器这个技术层次很难掌控。

为了解决这个问题，**容器编排**（Container Orchestration）就出现了，它可以说是以前的运维工作在云原生世界的落地实践，本质上还是在集群里调度管理应用程序，只不过管理的主体由人变成了计算机，管理的目标由原生进程变成了容器和镜像。

而现在，容器编排领域的王者就是——Kubernetes。

Kubernetes源自Borg系统，它凝聚了Google的内部经验和CNCF的社区智慧，所以战胜了竞争对手Apache Mesos和Docker Swarm，成为了容器编排领域的事实标准，也成为了云原生时代的基础操作系统，学习云原生就必须要掌握Kubernetes。

（[10讲](https://time.geekbang.org/column/article/529800)）Kubernetes的**Master/Node架构**是它具有自动化运维能力的关键，也对我们的学习至关重要，这里我再用另一张参考架构图来简略说明一下它的运行机制（[图片来源](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked)）：

![图片](https://static001.geekbang.org/resource/image/f4/05/f429ca7114eebf140632409f3fbcbb05.png?wh=1475x852)

Kubernetes把集群里的计算资源定义为节点（Node），其中又划分成控制面和数据面两类。

- 控制面是Master节点，负责管理集群和运维监控应用，里面的核心组件是**apiserver、etcd、scheduler、controller-manager**。
- 数据面是Worker节点，受Master节点的管控，里面的核心组件是**kubelet、kube-proxy、container-runtime**。

此外，Kubernetes还支持插件机制，能够灵活扩展各项功能，常用的插件有DNS和Dashboard。

为了更好地管理集群和业务应用，Kubernetes从现实世界中抽象出了许多概念，称为“**API对象**”，描述这些对象就需要使用**YAML**语言。

YAML是JSON的超集，但语法更简洁，表现能力更强，更重要的是它以“**声明式**”来表述对象的状态，不涉及具体的操作细节，这样Kubernetes就能够依靠存储在etcd里集群的状态信息，不断地“调控”对象，直至实际状态与期望状态相同，这个过程就是Kubernetes的自动化运维管理（[11讲](https://time.geekbang.org/column/article/529813)）。

Kubernetes里有很多的API对象，其中最核心的对象是“**Pod**”，它捆绑了一组存在密切协作关系的容器，容器之间共享网络和存储，在集群里必须一起调度一起运行。通过Pod这个概念，Kubernetes就简化了对容器的管理工作，其他的所有任务都是通过对Pod这个最小单位的再包装来实现的（[12讲](https://time.geekbang.org/column/article/531551)）。

除了核心的Pod对象，基于“单一职责”和“对象组合”这两个基本原则，我们又学习了4个比较简单的API对象，分别是**Job/CronJob**和**ConfigMap**/**Secret**。

- Job/CronJob对应的是离线作业，它们逐层包装了Pod，添加了作业控制和定时规则（[13讲](https://time.geekbang.org/column/article/531566)）。
- ConfigMap/Secret对应的是配置信息，需要以环境变量或者存储卷的形式注入进Pod，然后进程才能在运行时使用（[14讲](https://time.geekbang.org/column/article/533395)）。

和Docker类似，Kubernetes也提供一个客户端工具，名字叫“**kubectl**”，它直接与Master节点的apiserver通信，把YAML文件发送给RESTful接口，从而触发Kubernetes的对象管理工作流程。

kubectl的命令很多，查看自带文档可以用 `api-resources`、`explain` ，查看对象状态可以用 `get`、`describe`、`logs` ，操作对象可以用 `run`、`apply`、`exec`、`delete` 等等（[09讲](https://time.geekbang.org/column/article/529780)）。

使用YAML描述API对象也有固定的格式，必须写的“头字段”是“**apiVersion**”“**kind**”“**metadata**”，它们表示对象的版本、种类和名字等元信息。实体对象如Pod、Job、CronJob会再有“**spec**”字段描述对象的期望状态，最基本的就是容器信息，非实体对象如ConfigMap、Secret使用的是“**data**”字段，记录一些静态的字符串信息。

好了，“初级篇”里的Kubernetes知识要点我们就基本总结完了，如果你发现哪部分不太清楚，可以课后再多复习一下前面的课程加以巩固。

## WordPress网站基本架构

下面我们就在Kubernetes集群里再搭建出一个WordPress网站，用的镜像还是“入门篇”里的那三个应用：WordPress、MariaDB、Nginx，不过当时我们是直接以容器的形式来使用它们，现在要改成Pod的形式，让它们运行在Kubernetes里。

我还是画了一张简单的架构图，来说明这个系统的内部逻辑关系：

![图片](https://static001.geekbang.org/resource/image/3d/cc/3d9d09078f1200a84c63a7cea2f40bcc.jpg?wh=1920x865)

从这张图中你可以看到，网站的大体架构是没有变化的，毕竟应用还是那三个，它们的调用依赖关系也必然没有变化。

那么Kubernetes系统和Docker系统的区别又在哪里呢？

关键就在**对应用的封装**和**网络环境**这两点上。

现在WordPress、MariaDB这两个应用被封装成了Pod（由于它们都是在线业务，所以Job/CronJob在这里派不上用场），运行所需的环境变量也都被改写成ConfigMap，统一用“声明式”来管理，比起Shell脚本更容易阅读和版本化管理。

另外，Kubernetes集群在内部维护了一个自己的专用网络，这个网络和外界隔离，要用特殊的“端口转发”方式来传递数据，还需要在集群之外用Nginx反向代理这个地址，这样才能实现内外沟通，对比Docker的直接端口映射，这里略微麻烦了一些。

## WordPress网站搭建步骤

了解基本架构之后，接下来我们就逐步搭建这个网站系统，总共需要4步。

**第一步**当然是要编排MariaDB对象，它的具体运行需求可以参考“入门篇”的实战演练课，这里我就不再重复了。

MariaDB需要4个环境变量，比如数据库名、用户名、密码等，在Docker里我们是在命令行里使用参数 `--env`，而在Kubernetes里我们就应该使用ConfigMap，为此需要定义一个 `maria-cm` 对象：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: maria-cm

data:
  DATABASE: 'db'
  USER: 'wp'
  PASSWORD: '123'
  ROOT_PASSWORD: '123'
```

然后我们定义Pod对象 `maria-pod`，把配置信息注入Pod，让MariaDB运行时从环境变量读取这些信息：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: maria-pod
  labels:
    app: wordpress
    role: database

spec:
  containers:
  - image: mariadb:10
    name: maria
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 3306

    envFrom:
    - prefix: 'MARIADB_'
      configMapRef:
        name: maria-cm
```

注意这里我们使用了一个新的字段“**envFrom**”，这是因为ConfigMap里的信息比较多，如果用 `env.valueFrom` 一个个地写会非常麻烦，容易出错，而 `envFrom` 可以一次性地把ConfigMap里的字段全导入进Pod，并且能够指定变量名的前缀（即这里的 `MARIADB_`），非常方便。

使用 `kubectl apply` 创建这个对象之后，可以用 `kubectl get pod` 查看它的状态，如果想要获取IP地址需要加上参数 `-o wide` ：

```plain
kubectl apply -f mariadb-pod.yml
kubectl get pod -o wide
```

![图片](https://static001.geekbang.org/resource/image/3f/98/3fb0242f97c782f79ecf8ba845c81798.png?wh=1788x362)

现在数据库就成功地在Kubernetes集群里跑起来了，IP地址是“172.17.0.2”，注意这个地址和Docker的不同，是Kubernetes里的私有网段。

接着是**第二步**，编排WordPress对象，还是先用ConfigMap定义它的环境变量：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wp-cm

data:
  HOST: '172.17.0.2'
  USER: 'wp'
  PASSWORD: '123'
  NAME: 'db'
```

在这个ConfigMap里要注意的是“HOST”字段，它必须是MariaDB Pod的IP地址，如果不写正确WordPress会无法正常连接数据库。

然后我们再编写WordPress的YAML文件，为了简化环境变量的设置同样使用了 `envFrom`：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wp-pod
  labels:
    app: wordpress
    role: website

spec:
  containers:
  - image: wordpress:5
    name: wp-pod
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80

    envFrom:
    - prefix: 'WORDPRESS_DB_'
      configMapRef:
        name: wp-cm
```

接着还是用 `kubectl apply` 创建对象，`kubectl get pod` 查看它的状态：

```plain
kubectl apply -f wp-pod.yml
kubectl get pod -o wide
```

![图片](https://static001.geekbang.org/resource/image/d5/de/d5e8c09e70e90179d651bf3c28abc0de.png?wh=1562x426)

**第三步**是为WordPress Pod映射端口号，让它在集群外可见。

因为Pod都是运行在Kubernetes内部的私有网段里的，外界无法直接访问，想要对外暴露服务，需要使用一个专门的 `kubectl port-forward` 命令，它专门负责把本机的端口映射到在目标对象的端口号，有点类似Docker的参数 `-p`，经常用于Kubernetes的临时调试和测试。

下面我就把本地的“8080”映射到WordPress Pod的“80”，kubectl会把这个端口的所有数据都转发给集群内部的Pod：

```plain
kubectl port-forward wp-pod 8080:80 &
```

![图片](https://static001.geekbang.org/resource/image/d4/be/d445d205ae6f8c966200ffa9ba7f29be.png?wh=1366x240)

注意在命令的末尾我使用了一个 `&` 符号，让端口转发工作在后台进行，这样就不会阻碍我们后续的操作。

如果想关闭端口转发，需要敲命令 `fg` ，它会把后台的任务带回到前台，然后就可以简单地用“Ctrl + C”来停止转发了。

**第四步**是创建反向代理的Nginx，让我们的网站对外提供服务。

这是因为WordPress网站使用了URL重定向，直接使用“8080”会导致跳转故障，所以为了让网站正常工作，我们还应该在Kubernetes之外启动Nginx反向代理，保证外界看到的仍然是“80”端口号。（这里的细节和我们的课程关系不大，感兴趣的同学可以留言提问讨论）

Nginx的配置文件和[第7讲](https://time.geekbang.org/column/article/528740)基本一样，只是目标地址变成了“127.0.0.1:8080”，它就是我们在第三步里用 `kubectl port-forward` 命令创建的本地地址：

```plain
server {
  listen 80;
  default_type text/html;

  location / {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http://127.0.0.1:8080;
  }
}
```

然后我们用 `docker run -v` 命令加载这个配置文件，以容器的方式启动这个Nginx代理：

```plain
docker run -d --rm \
    --net=host \
    -v /tmp/proxy.conf:/etc/nginx/conf.d/default.conf \
    nginx:alpine
```

![图片](https://static001.geekbang.org/resource/image/9f/51/9f2b16fb58dbe0a358e26042565f9851.png?wh=1920x238)

有了Nginx的反向代理之后，我们就可以打开浏览器，输入本机的“127.0.0.1”或者是虚拟机的IP地址（我这里仍然是“[http://192.168.10.208](http://192.168.10.208)”），看到WordPress的界面：

![图片](https://static001.geekbang.org/resource/image/73/f4/735552be9cf6d45ac41a001252ayyef4.png?wh=1524x1858)

你也可以在Kubernetes里使用命令 `kubectl logs` 查看WordPress、MariaDB等Pod的运行日志，来验证它们是否已经正确地响应了请求：

![图片](https://static001.geekbang.org/resource/image/84/62/8498c598e6f3142d490218601acdbc62.png?wh=1920x809)

## 使用Dashboard管理Kubernetes

到这里WordPress网站就搭建成功了，我们的主要任务也算是完成了，不过我还想再带你看看Kubernetes的图形管理界面，也就是Dashboard，看看不用命令行该怎么管理Kubernetes。

启动Dashboard的命令你还记得吗，在第10节课里讲插件的时候曾经说过，需要用minikube，命令是：

```plain
minikube dashboard
```

它会自动打开浏览器界面，显示出当前Kubernetes集群里的工作负载：

![图片](https://static001.geekbang.org/resource/image/53/59/536eeb176a7737c9ed815c10af0fcf59.png?wh=1920x1022)

点击任意一个Pod的名字，就会进入管理界面，可以看到Pod的详细信息，而右上角有4个很重要的功能，分别可以查看日志、进入Pod内部、编辑Pod和删除Pod，相当于执行 `logs`、`exec`、`edit`、`delete` 命令，但要比命令行要直观友好的多：

![图片](https://static001.geekbang.org/resource/image/d5/28/d5e5131bfb1d6aae2f026177bf283628.png?wh=1920x781)

比如说，我点击了第二个按钮，就会在浏览器里开启一个Shell窗口，直接就是Pod的内部Linux环境，在里面可以输入任意的命令，无论是查看状态还是调试都很方便：

![图片](https://static001.geekbang.org/resource/image/46/4c/466c67a48616c946505242d0796ed74c.png?wh=1820x1240)

ConfigMap/Secret等对象也可以在这里任意查看或编辑：

![图片](https://static001.geekbang.org/resource/image/de/22/defyybc05ed793b7966e1f6b68018022.png?wh=1312x976)

Dashboard里的可操作的地方还有很多，这里我只是一个非常简单的介绍。虽然你也许已经习惯了使用键盘和命令行，但偶尔换一换口味，改用鼠标和图形界面来管理Kubernetes也是件挺有意思的事情，有机会不妨尝试一下。

## 小结

好了，作为“初级篇”的最后一节课，今天我们回顾了一下Kubernetes的知识要点，我还是画一份详细的思维导图，帮助你课后随时复习总结。

![图片](https://static001.geekbang.org/resource/image/87/1f/87a1d338340c8ca771a97d0fyy4b611f.jpg?wh=1920x1877)

这节课里我们使用Kubernetes搭建了WordPress网站，和第7讲里的Docker比较起来，我们应用了容器编排技术，以“声明式”的YAML来描述应用的状态和它们之间的关系，而不会列出详细的操作步骤，这就降低了我们的心智负担——调度、创建、监控等杂事都交给Kubernetes处理，我们只需“坐享其成”。

虽然我们朝着云原生的方向迈出了一大步，不过现在我们的容器编排还不够完善，Pod的IP地址还必须手工查找填写，缺少自动的服务发现机制，另外对外暴露服务的方式还很原始，必须要依赖集群外部力量的帮助。

所以，我们的学习之旅还将继续，在接下来的“中级篇”里，会开始研究更多的API对象，来解决这里遇到的问题。

## 课下作业

最后是课下作业时间，给你留两个动手题：

1. MariaDB、WordPress现在用的是ConfigMap，能否改用Secret来实现呢？
2. 你能否把Nginx代理转换成Pod的形式，让它在Kubernetes里运行呢？

期待能看到你动手体验后的想法，如果觉得有帮助，欢迎分享给自己身边的朋友一起学习。

下节课就是视频演示的操作课了，我们下节课再见。

![图片](https://static001.geekbang.org/resource/image/3c/ea/3c3036bc56bb9ec14598342e56c11bea.jpg?wh=1920x1544)
<div><strong>精选留言（15）</strong></div><ul>
<li><span>YueShi</span> 👍（18） 💬（7）<p>Mac上或者Window上，可能出现访问不到问题，这里写了一篇排查问题的思路，期望能帮到各位老板

https:&#47;&#47;juejin.cn&#47;post&#47;7127679053242302477</p>2022-08-04</li><br/><li><span>pyhhou</span> 👍（18） 💬（3）<p>思考题 2，

试着写出了 Pod 的 YAML 文件

apiVersion: v1
kind: ConfigMap
metadata:
  name: ngx-cm

data:
  default.conf: |-
    server {
      listen 80;
      default_type text&#47;html;

      location &#47; {
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_pass http:&#47;&#47;127.0.0.1:8080;
      }
    }

---

apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    app: nginx-alpine
    role: website

spec:
  volumes:
    - name: conf
      configMap:
        name: ngx-cm

  containers:
  - volumeMounts:
    - mountPath: &#47;etc&#47;nginx&#47;conf.d&#47;
      name: conf

    image: nginx:alpine
    name: ngx-pod
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80

可以成功创建 Pod，nginx 的配置文件也被挂载到指定的位置，但是浏览器还是无法访问 wordpress，不知道是不是我的 YAML 文件中少了什么🤔，还望老师指点</p>2022-07-25</li><br/><li><span>朱雯</span> 👍（11） 💬（1）<p>学了这一节，没感觉到容器编排的好用，反而遇到了一个问题，在配置中，我把wordpress链接数据库的host给小写了，结果一直告诉我链接数据库失败，我也没有排查手段，一直失败，我想的是，这个k8s的部署步骤实在是太复杂了，复杂到配置需要一个yaml，容器需要一个yaml，port-forward也需要一个yaml，我想如果是docker直接部署，其实就需要两条命令，所有参数都放到env中，或者放到配置文件中，也比检查yaml文件强，在入门篇中没感受到k8s的好用，只感觉到琐碎，是因为服务太小，还是因为没用好的原因呢
</p>2022-07-28</li><br/><li><span>马以</span> 👍（9） 💬（7）<p>写一下第2题吧解题步骤：
首先如果我们把docker 形式改成pod形式这里存在两个问题（1）网络问题，（2）nginx配置文件加载问题，
第一个问题k8s的插件会处理节点内以及节点间网络通信问题，这个我们这里暂时不需要考虑，
那么需要我们处理的就是配置文件问题：这里我们可以用多个方法处理
   a: pod挂载宿主机目录，把配置文件放到宿主机目录下，容器启动挂载
   b: 使用我们学过的configMap
   c：使用pv、pvc 这个后面老师应该会讲到
这里我门就使用第二种方法，也是评论里使用最多的方法：
yml文件如下：
    ngx-cm.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: ngx-cm
data:
  nginx.conf: |
    server {
      listen 8080;
      #default_type text&#47;html;

      location &#47; {
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_pass http:&#47;&#47;172.17.0.9:80;
      }
    }

nginx.yml:

apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    env: demo
    owner: ant

spec:
  volumes:
    - name: nginx-conf
      configMap:
        name: ngx-cm
        items:
        - key: nginx.conf
          path: default.conf
  containers:
  - image: nginx:alpine
    name: ngx
    volumeMounts:
      - name: nginx-conf
        mountPath: &#47;etc&#47;nginx&#47;conf.d
    ports:
    - containerPort: 8080

然后使用 kubectl apply -f nginx.yml 创建pod，但是这个时候由于k8s节点还是一个隔离环境，所以我们还是无法访问，
所以我们要在网页观察到具体的网页内容，还是要再用docker创建一层nginx代理，server 内容如下：
server {
  listen 80;
  default_type text&#47;html;

  location &#47; {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http:&#47;&#47;127.0.0.1:8888;
  }
}

使用 docker run -d --rm --net=host -v &#47;home&#47;ant&#47;kubernetes&#47;pod&#47;nginx-conf&#47;nginx.conf:&#47;etc&#47;nginx&#47;conf.d&#47;default.conf nginx:alpine
启动，
然后使用 kubectl port-forward ngx-pod 8888:8080 &amp; 做转发

访问 http:&#47;&#47;192.168.56.208&#47; 正常访问</p>2022-08-12</li><br/><li><span>椰子</span> 👍（7） 💬（1）<p>老师，目前Pod配置中使用了固定IP。如果pod被重启了，ip 是否会变化？如果有变化，这样配置是否有点不合理？</p>2022-07-25</li><br/><li><span>wcy</span> 👍（4） 💬（1）<p>作业1：改用secret可以，这个简单，把data里对应的值用base64编码就行，对应pod的yml改下
作业2：nginx改成pod 运行是成功的， 但是port-forward出来，linux虚机上用firefox打开页面是ok的。但在shell终端本地浏览器访问不了，看了下应该是port-forward的问题，映射的端口绑定在Linux虚机127.0.0.1上，所以virtualbox转发不出来。
nginx cm:
apiVersion: v1
kind: ConfigMap
metadata:
  name: ngx-cm
data:
  nginx.conf: |
    server {
    #  listen 80;
       listen 8080;
    #   default_type text&#47;html;

       location &#47; {
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_pass http:&#47;&#47;172.17.0.10:80;
       }
     }

nginx pod配置：
apiVersion: v1
kind: Pod
metadata:
  name: wp-ngx-pod
  labels:
    env: test

spec:
  containers:
  - image: nginx:alpine
    name: wp-ngx
    ports:
    - containerPort: 8080
    volumeMounts:
    - mountPath: &#47;etc&#47;nginx&#47;conf.d&#47;
      name: myngxconf

  volumes:
  - name: myngxconf
    configMap:
      name: ngx-cm
      items:
      - key: nginx.conf
        path: default.conf
</p>2022-08-04</li><br/><li><span>edward</span> 👍（3） 💬（2）<p>kubectl port-forward 老师请教下 这个命令设置端口转发后，怎么查看和取消？</p>2022-11-08</li><br/><li><span>peter</span> 👍（3） 💬（2）<p>请教老师几个问题：
Q1：MariaDB和WP可以跑在一个POD里面吗？

Q2：文中的prefix是什么意思？
文中定义mariadb-pod.yml的时候，有一个prefix: &#39;MARIADB_&#39;， 这是什么意思？
是把configMap中的变量名字前面加上这个前缀作为pod中的变量名称吗？
即：pod中的变量名称 = MARIADB_&#39; + configMap中的变量名称。
Q3：关于转发的疑问
文中有一句：“因为 WordPress 网站使用了 URL 重定向，直接使用“8080”会导致跳转故障”。 如果没有nginx，宿主机上用浏览器直接访问WP的8080端口，则会因为WP内部实现机制而产生错误； 如果加上nginx，nginx去访问WP的8080端口，就没事。 是这样吗？</p>2022-07-25</li><br/><li><span>pyhhou</span> 👍（3） 💬（3）<p>有在操作过程中遇到几个问题，还烦请老师指点

1、前面 2 个步骤中，在 kubectl apply -f *-pod.yml 之前是不是也得 apply 一下之前定义的 configMap 对象？

2、对于反向代理，不是特别的理解，老师能简单解释一下反向代理，并附带说明这里 Nginx 这里作为反向代理的目的是什么吗？

3、`minikube dashboard` 无法在命令行显示界面，我将 URL 复制粘贴到 Ubuntu 虚拟机图形界面上的浏览器中，但回复的 JSON 中显示 404？不太清楚有没有更好的方法
Opening http:&#47;&#47;127.0.0.1:38539&#47;api&#47;v1&#47;namespaces&#47;kubernetes-dashboard&#47;services&#47;http:kubernetes-dashboard:&#47;proxy&#47; in your default browser...
Error: no DISPLAY environment variable specified

4、对于思考题的 2，我创建一个了 nginx Pod，然后将其配置文件通过 kubectl cp 拷贝到 Pod 容器中的对应位置，但是发现并不起作用，本来想着重启 Pod 让配置生效，但是并没有找到对应的指令。这里是不是需要其他的 K8S API 对象的帮助？</p>2022-07-25</li><br/><li><span>未来已来</span> 👍（2） 💬（1）<p>第四步 Nginx 部分使用 docker 启动命令把 -v 后面的 &#47;tmp&#47;proxy.conf 改为 `pwd`&#47;proxy.conf 即可，跟 dokcer 实战部分保持一致</p>2023-05-14</li><br/><li><span>Geek_f2f06e</span> 👍（2） 💬（1）<p>老师我有一个疑问想请教您：
这里把每个应用都封装成pod，为什么不将MariaDB和WordPress封装到一个pod里面，pod的意义不就是将多个密切的容器封装在一起</p>2023-04-02</li><br/><li><span>doos</span> 👍（2） 💬（1）<p>这个入门写的太好了，我当时学是先学习docker，研究了3个月docker，后来才慢慢去接触k8s学了半年，感觉内容太多了，而且知识比较分散</p>2023-02-23</li><br/><li><span>Geek_39bdd5</span> 👍（2） 💬（4）<p>kubectl get pod -o wide查看的时候ip显示是none是怎么回事
</p>2022-08-10</li><br/><li><span>RRR</span> 👍（2） 💬（2）<p>两个问题

1. port-forward 实际在生产用的多么？应该都是 ingress 吧
2. 老师后面能讲讲 helm 和 Istio 吗，他们和 k8s 关系是什么，我看不明白</p>2022-07-27</li><br/><li><span>牙小木</span> 👍（1） 💬（1）<p>注意wp-pod.yaml中的HOST值，需要按照自己机器上的来。
tb@tb-laptop:~$ k get pod maria-pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
maria-pod   1&#47;1     Running   0          26m   10.244.0.76   minikube   &lt;none&gt;           &lt;none&gt;

</p>2023-07-26</li><br/>
</ul>