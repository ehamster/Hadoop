kubernets集群
===============

kubernets构建出一个计算机集群，让app能在容器化之后在集群运行，与单个机器脱离！
组成：
-----------------
1.由两部分组成：master 和 节点
  master:管理调度，维护程序状态，扩展和升级。
  节点：可以是虚拟机或者实体机。包含一个keberlet和master通信，和一个容器工具(Docker)
2.交互
  用户通过API和master交互，控制app在节点上部署
  
搭建集群(使用kebeadm，实战可能不用这个)
--------------
1.一般至少有一个master个3个worker节点
kubeadm是快速部署工具，不能用于生产，但是体验够了
2.下载好所有镜像
kubeadm init --apiserver-advertise-address $(hostname -i) --kubernetes-version=v1.8.13  ---初始化集群
  之后出现You can now join....就是加入工作节点

3.使用kubectl命令与集群交互
  1）先看有没有安装kubectl工具   kubectl version
 
    kubectl cluster-info---查看集群信息
    kubectl get nodes--查看节点
  2）允许APP运行在master节点
    因为这里我们只用一个master节点做实验，所以需要接触设置，使得APP能运行在master节点
    kubectl apply -f http://aaa.yaml   ----使用预先定义好的yaml文件，激活模型。这样容器化的运行实例就部署好了。

部署APP
==================
步骤：
  1.创建deployment配置，之后master会将应用程序实例分发到各个节点
  2.Kubenetes Deployment Controller会监视实例，节点故障的话，控制器会替换它。


实例：将nginx（web服务器）部署到kubernets集群上

  1.下载好镜像
  -------------------------
  2.启动deployment(并为app设置好端口)
  ----------------------
    kubectl run deploymentName --image=nginx --image-pull-policy=IfNotPresent(在内网，所以不能从internet获取) --port=80
    
  这里包括3个行为：
  1.搜索合适节点
  2.安排应用程序运行在该节点
  3.配置集群以再需要时重新安排节点
  
获取所有deployments：   kubectl get deployments
  3.如何访问服务？
  -------------------------
    1.我们的app运行在容器里，容器在pod里，pod在私有网络里，外部无法访问
    2.启动好之后，需要激活kubectl的代理功能，将通信转发到集群范围内
    3.将代理后台运行 kubectl proxy &
    打开代理之后，可以用这个查看所有API信息
    4.访问APP之前，需要知道运行APP的容器是在哪个pod里，并保存在环境变量中，使用这个语句
       export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
       echo Name of the Pod: $POD_NAME
    5.获得Pod名字以后，就可以访问了
    curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
    


3.POD NODES概念
===========
pod：多个APP容器和资源，共享ip和端口号，同一个节点
nodes：可以是虚拟机或实体机。
  1）.运行了kubelet
  2）.可能有好多pod
  
 4.kubectl命令
 =================
 
kubectl get pods --查看正在运行的pods
kubectl describe pods  --查看所有pod使用的镜像和ip等等

暴露你的APP
=================

1.同一个node上的pod ip不一样。节点死亡，pod死亡。ReplicationController可以创建新的pod来将APP维持住
2.Service:定义一组pods和访问策略，可以使用yaml或json定义service
  1)ClusterIP（默认）只能集群内部访问
  2)NodePort 外部可访问
  3)LoadBalancer
  4)ExternalName
service指定了pod的ip是否可以被看见

3.Service使用label和selectors匹配一组pods,也就是使用label可以找到对应的pods

4.新建service并让外部可以访问app：
  1）kubectl expose deployment/nginx --type="NodePort" --port 80 ----新建service
  2）再查看服务就能发现有新的service了
  kubectl get service
  集群启动时会自动生成一个服务：Kubernetes，通过上面的操作产生新的service nginx
  3）查看外部端口
  export NODE_PORT=$(kubectl get services/nginx -o go-template='{{(index .spec.ports 0).nodePort}}')
  echo $NODE_PORT
  
  4)访问
  curl localhost:$NODE_PORT
  
5.如何使用label
 deployment自动为pod创建了label：使用这个查看
 kubectl describe deployment 获取label值 run=nginx
 
 然后查看该label
 kubectl get pods -l run=nginx
 也可以给pod添加label
 获取pod名字之后赋值给变量：
  kubectl label pod $POD_NAME app=v1
  
 6.删除服务
  kubectl delete service -l run=nginx
  服务删除后，仍然可以通过 pod ip访问容器里面的app
  
 7.清除工作环境
  kubectl delete deployment nginx
  
扩展缩放APP
==============

运动多个APP实例在多个Pods，需要使用service管理
1.新建deployment
kubectl run deploymentname --image=imageName --port=8080 --image-pull-policy=IfNotPresent (内网)
2.创建service以便以后验证
kubectl expose deployment/deploymentName --type='NodePort' --port 8080 暴露IP让外部可以访问
export NODE_PORT = $.....
echo NODE_....

3.扩充为4个实例
  kubectl scale deployments/kubernetes-bootcamp --replicas=4
  
4. 查看
curl localhost:$NODE_PORT  可以发现名字pod名字不一样

5.减小为2
kubectl scale deployments/kubernetes-bootcamp --relicas=2

滚动更新App
===================

保持APP可用的同时更新版本,所以需要缩放pods!!!而且可以还原为原始版本
1.下载镜像

2.kubectl set image deployments/deploymentName deploymentName=xxxxx:v2

3.暴露ip查看是否更新同上
或者查看当前pod版本
kubectl describe pods

4.升级失败如何回滚？

 kubectl rollout undo deployments/deployment_name

