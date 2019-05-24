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
kubectl apply -f http://aaa.yml

部署APP
==================

1.创建deployment配置，之后master会将应用程序实例分发到各个节点
2.Kubenetes Deployment Controller会监视实例，节点故障的话，控制器会替换它。

