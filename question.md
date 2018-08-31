[TOC]
# 1.istio和k8s
* istio 如何和k8s结合
* K8s如何将和策略流量引入istio
* istio-system namespace, gateway在k8s中起什么作用
   
在k8s集群中使用istio，首先在k8s集群中创建istio的相关服务，命令如下：
```
  在k8s中安装istio，
  $kubectl apply -f install/kubernetes/istio-demo.yaml
```
# 2.规则
istio流量控制是通过规则的转换，转发等实现的。规则主要有以下因素：
产生，内容，转换，下发,作用对象
规则管理的对象为servive,serverice在配置文件中在host选项中配置。
canonical representationi<br>
istio中定义了服务的标准表示，为了支持多平台，istio内部有相关平台的适配器。
k8s中的适配器，实现了检测API SERVER中系统资源变更的控制器，
把相关的变更转换为标准表示。envoy中的规则也是从标准规则中转换而来。
### 1.产生
1.用户指定
用户将配置写在yaml文件中，通过istioctrl create -f 命令 配置生效
istioctl 作为客户端和pilict通信，pilict下发配置到envoy。

### 2.内容
#### 配置元素
* viutual service<br>
  定义请求路由规则， 根据不同的服务或者服务的版本号进行路由。
* destinstionrule<br>
  路由请求之后的处理规则,负载均衡，tls设置等
* serviceentry<br>
  数据流出规则
* gateway<br>
  提供负载均衡,提供流量进入应用
  istio仅仅支持l4-l6的负载均衡。
  使用:<br>
  1.定义gateway规则
  2.在virutalservice中指定gateway规则，才能生效 

#### 实现功能
<pre>
路由规则：根据不同的路径和版本，
负载均衡：相同的请求，分发到不同的服务
错误恢复：针对不同的错误，定义恢复措施
注入错误：注入不同的错误，用于测试
精确匹配：定义匹配的规则，规则之间是且的关系
多条匹配：
优先级： 指定匹配的优先级
</pre>
### 3.下发
pilot实现了服务发现功能，同步的更新到负载均衡和路由模块。


# 3.istio 如何将策略下发给envoy
# 4.envoy如何控制流量
# 5.通信过程中的流量如何转发


# 6.服务发现和负载均衡
istio的服务发现建立在底层平台之上，向下收集底层平台的服务注册信息，向上提供一个独立的服务发现
接口，envoy获取接口的信息，动态更新负载均衡池。

## 负载均衡池
<pre>
服务通过dns名称访问，所有的流量经过envoy的路由，envoy根据负载均衡池进行流量转发。  
负载均衡池中定义了多种算法：轮询，随机，权重等。  
envoy或定期检查池中实例的健康度，评判的标准可api调用的失败率。
当失败率过高，就从池中调出。当失败率低就添加到池中。  
说明：当失败率过高，说明主机有多任务需要处理，就不应该给他再次导入流量。
      当失败率过高，就会出现503的情况，这个时候就是自动的从池中移除。
      当池中所有的实例都失败的情况下，enovy就会返回503.
</pre>


# 错误注入
延迟注入
404注入

# 容器注入
## 1.手动注入
istioctl 命令
kube-inject    Inject Envoy sidecar into Kubernetes pod resources
把k8s的配置信息，转换并添加iostio容器信息。
生成的信息会在development配置中添加：
```
代理
image: docker.io/istio/proxyv2:1.0.1
imagePullPolicy: IfNotPresent
name: istio-proxy
初始化代理
image: docker.io/istio/proxy_init:1.0.1
imagePullPolicy: IfNotPresent
name: istio-init
```
## 2.自动注入
设置nameapce的标签
```
$ kubectl label namespace default istio-injection=enabled
$ kubectl get namespace -L istio-injection
```
禁止自动注入
```
$ kubectl label namespace default istio-injection-
$ kubectl delete pod sleep-776b7bcdcd-bhn9m
```
## 3.注入的原理
注入的对象为deployment
 







#相关实践
## 注入实践
```
1. 手动注入
istioctl kube-inject -f samples/sleep/sleep.yaml | kubectl apply -f -

2.查看容器
在depoyment中查看容器
lindeMacBook-Pro:istio-1.0.1 lin$ kubectl get deployment sleep -o wide
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS          IMAGES                                          SELECTOR
sleep     1         1         1            1           1d        sleep,istio-proxy   tutum/curl,gcr.io/istio-release/proxyv2:1.0.1   app=sleep
查看pod
lindeMacBook-Pro:istio-1.0.1 lin$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
sleep-6bc4c89dcc-m4zgf            2/2       Running   0          11m

3.进入容器中查看进程
lindeMacBook-Pro:istio-1.0.1 lin$ kubectl exec -it sleep-6bc4c89dcc-m4zgf -c istio-proxy /bin/sh
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
istio-p+     1  0.1  0.0  29784     0 ?        Ssl  02:27   0:01 /usr/local/bin/pilot-agent proxy sidecar --configPath /etc/istio/proxy --binaryPath /usr/local/bin/envoy --serviceCluster sleep
istio-p+    14  0.2  0.0  79972  1968 ?        Sl   02:27   0:01 /usr/local/bin/envoy -c /etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --parent-shutdown-time-s 60 --servi
istio-p+    31  0.0  0.0   4500   712 pts/0    Ss   02:39   0:00 /bin/sh
istio-p+    35  0.0  0.1  34420  2676 pts/0    R+   02:40   0:00 ps aux
```

##配置规则示例
### 1.路由规则--指定版本
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
```
host：指定访问的主机
http：指定http服务规则
> route：指定路由规则
>> destinstion：指定目的地址
>>> host: 指定主机<br>
>>> subset:指定版本<br>

subset可以指定多个标签来指定服务的版本实例。<br>
上述规则定义了reviews服务的流量只转发到v1版本的路由规则。

### 2.目的规则--负载均衡
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```
上述目的规则，定义了在review服务中，在v1和v2版本中做负载均衡。
service 提供路由方向，destination定义具体的路由策略。

# 问题解决
## extern-ip pending
如果您的集群在不支持外部负载均衡器的环境中运行（例如 minikube），
istio-ingressgateway的 EXTERNAL-IP 将会显示为 <pending> 状态。
您将需要使用服务的 NodePort 来访问，或者使用 port-forwarding。<br>
参考网址：[istio网址](https://istio.io/zh/docs/setup/kubernetes/quick-start/)<br>
用的是minnikube模拟的k8s,造成extern-ip为pending，使用nodeport解决
配置：
```
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  type: NodePort
  ports:
  - name: http
    port: 8000
    nodePort: 31000
  selector:
```

