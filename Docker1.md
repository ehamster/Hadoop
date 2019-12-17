Docker常用命令
=======================
基本
-----------------------
```bash
旧版本容器使用gpu:
docker run --runtime=nvidia ....
新版本使用gpu:
docker --gpu all
docker ps -a #查看当前容器
docker run -it -v /home:/home -p 10000:10000 --name aaa --rm -e LANG=C.UTF-8 镜像名字:v1 bash
```
-v就是把本地路径所有文件copy到容器一样的
-p就是把某个端口开放！
在容器里也可以改编码
export LANG=C.UTF-8
```bash
Ctrl + P + Q 退出容器保持后台运行（按住ctrl和P 再按Q），再用 docker ps 进行查看：  exit可以直接退出容器
docker exec -it ContainerName /bin/bash # 返回container,推出后会保留在后台
docker rm -v mySpark3  删除container和volume
```
如果说docker daemon 没有running，使用这个:
```bash
service docker start

docke stop xxx//停止容器
```
关于镜像:
----------------------
```bash
Docker创建新镜像     docker commit –m “test” –a “zmc” d8990fec2141 testimage  
Docker保存镜像到本地  docker save -o /home/wy/a.tar 镜像名字
压缩镜像    tar -czf a.tar.gz a.tar
载入镜像    docker load --input a.tar

m是注释
a作者
后面是容器id
新镜像名字
```
挂在后台
----------------------

tmux new -s wy_nlp
挂在后台

ctrl+B D  退出来

tmux a -t wy_nlp再进去


Docker课程学习笔记
===================
```bash
1.虚拟机是硬件抽象，从主机获取cpuram
2.容器是应用程序抽象

docker image pull xxxxx   默认从dockerstore下载镜像

docker container run imagename ls -l  创建xxx的容器然后运行 ls -l 然后退出

docker container run -it imagename /bin/bash   就是进入容器了
docker container ls  = docker ps
docker container ls -a  能看到已经退出的容器
docker container start container ID  ---------启动已经退出的container


docker container diff contain ID   --------查看在容器中添加或者更改
docker container commit contain ID -----创建镜像
docker image tag image ID myname ------添加名字



```

使用DockerFile创建镜像
------------------

```bash
DockerFile是docker创建镜像说明，所有此镜像容器都会执行docker file而里面的命令
在一个叫Dockerfile文件里写：
FROM alpine  ----基础镜像
RUN apk update && apk add nodejs ---运行命令
COPY ./app  ------从当前目录复制到工作目录
WORKDIR /app   ----指定工作路径，容器启动时使用这个
CMD ["node","index.js"]

然后创建镜像：
docker image build -t hello:v1
```

层
-----------

```bash
docker image history image ID
可以观察创建镜像的时候用了哪些层，镜像其实是很多层组成的，上面的底层就是alpine
在index.js添加一行之后，再构建一个镜像，会发现docker构建前面相同层的时候直接使用的缓存

docker image inspect image id 可以查看镜像内部层
卷：docker容器层，允许数据持久化

```

swarm集群模式
--------------------

```bash
实际使用需要3个以上manager和多个worker，这里只用一个演示，manager也可以是worker
docker swarm init --advertise-addr $(hostname -i)

之后根据提示添加worker
docker node ls ------------查看所有节点
多个manager中有一个是leader节点

部署 STACK：  stack包含多个服务
  写好配置文件docker-stack.yml,定义整个stack。services里面有很多服务组件，
  replicas表示该服务的实例数量
  
  执行docker stackdeploy --compose-file=docker-stack.yml stackname
  docker stack ls -------查看当前stack
  dockers stack services stackname -----查看每个服务
  docker service ps service_name  ---查看服务的任务
  

```
