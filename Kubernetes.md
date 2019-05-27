kubernets集群
===============

搭建集群
----------------

至少有一个master个3个worker节点
kubeadm是快速部署工具，不能用于生产，但是体验够了

kubeadn ubut --apiserver-advetise-address $(hostname -i) --kubernetes-version=v1.8.13  ---初始化集群
之后出现You can now join....就是加入工作节点

因为是使用kubectl和集群交互
kubectl cluster-info---查看信息
kubectl get nodes--查看节点
kubectl apply -f http://aaa.yaml   ----使用预先定义好的yaml文件

部署APP
==================


1.创建deployment配置，之后master会将应用程序实例分发到各个节点
2.Kubenetes Deployment Controller会监视实例，节点故障的话，控制器会替换它。


实例：运行nginx（web服务器）
1.启动deployment
kubectl run deploymentName --image=nginx --image-pull-policy=IfNotPresent(再内网，所以不能从internet获取) --port=80
这里包括3个行为：
  1.搜索合适节点
  2.安排应用程序运行在该节点
  3.配置集群以再需要时重新安排节点
  
获取所有deployments：   kubectl get deployments
2.启动好之后，需要激活kubectl的代理功能，将通信转发到集群范围内
kubectl proxy &


3.POD NODES
===========
pod：一组多个应用程序容器，共享ip和端口号，同一个节点
nodes：可以是虚拟机或实体机，运行了kubelet，可能有好多pod
kubectl get pods
kubectl describe pods


同一个node上的podip不一样
节点死亡，pod死亡

service指定了pod的ip是否可以被看见
kubectl expose deployment/nginx --type="NodePort" --port 80
再查看服务就能发现有新的暴露了
kubectl get service


集群启动时会自动生成一个服务：Kubernetes，通过上面的操作产生新的service nginx

