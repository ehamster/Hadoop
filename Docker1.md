Docker常用命令
=======================
基本
-----------------------
```bash
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


Docker创建新镜像     docker commit –m “test” –a “zmc” d8990fec2141 testimage  
Docker保存镜像到本地  docker save -o /home/wy/a.tar 镜像名字

压缩镜像    tar -czf a.tar.gz a.tar
载入镜像    docker load --input a.tar

m是注释
a作者
后面是容器id
新镜像名字

挂在后台
----------------------

tmux new -s wy_nlp
挂在后台

ctrl+B D  退出来

tmux a -t wy_nlp再进去
