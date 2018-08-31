[TOC]

k8s命令实践
=======
#minikube 管理命令
```
minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.10.0 \
>     --extra-config=controller-manager.cluster-signing-cert-file="/var/lib/localkube/certs/ca.crt" \
>     --extra-config=controller-manager.cluster-signing-key-file="/var/lib/localkube/certs/ca.key" \
>     --vm-driver=`virtualbox`
kubectl cluster-info 去连一下k8s api server.

创建虚拟机
minikube start --registry-mirror=https://registry.docker-cn.com
可以用--extra-config=component.key=value的形式给 Kubernetes 指定参数


打开控制台
minikube dashboard
停止cluster
minikube stop

删除cluster
minikube delete

访问服务
minikube service [-n NAMESPACE] [--url] NAME

获取ip
minikube ip

提供外部服务
Minikube中，LoadBanlancer类型让服务可以通过minikube service来访问
minikube service hello-node
这个命令会自动打开浏览器窗口访问服务，使用本地ip

```
[minnikube提供外部服务](http://www.cnblogs.com/cocowool/p/minikube_setup_and_first_sample.html)


# k8s内部命令

```
执行命令
kubectl exec my-nginx-3800858182-e9ihh -- printenv | grep SERVICE
访问服务：
export INGRESS_HOST=$(minikube ip)
http://192.168.99.100:31004/productpage

应用配置
kubectl apply -f
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

删除配置
kubectl delete -f

查看配置规则
kubectl get destinationrules -o yaml

获取自动注入的属性
kubectl get namespace -L istio-injection

在命令空间中获得服务
kubectl get svc -n istio-system

在命令空间中获得pod
kubectl get pods -n istio-system

自动注入配置
kubectl label namespace <namespace> istio-injection=enabled
kubectl create -n <namespace> -f <your-app-spec>.yaml
eg:kubectl label namespace default istio-injection=enabled

进入pod中的容器
kubectl exec -it [httpbin] -c istio-proxy sh

更改服务配置
kubectl edit virtualservice -n istio-system

查看外部服务端口
kubectl get svc istio-ingressgateway -n istio-system
1、查看指定pod的日志
kubectl logs <pod_name>
kubectl logs -f <pod_name> #类似tail -f的方式查看(tail -f 实时查看日志文件 tail -f 日志文件log)

2、查看指定pod中指定容器的日志
kubectl logs <pod_name> -c <container_name>
PS：查看Docker容器日志 
docker logs <container_id>
```




