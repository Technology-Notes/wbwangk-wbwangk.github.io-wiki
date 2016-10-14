一个有价值的参考文档：[Docker1.12服务发现，负载均衡和Routing Mesh](http://wwwbuild.net/dockerone/414200.html)
### 创建管理节点
```
docker swarm init --advertise-addr 10.10.56.1

Swarm initialized: current node (bzodyyh5o6jgj0k7xcjm5gmv3) is now a manager.
To add a worker to this swarm, run the following command:
    docker swarm join \
    --token SWMTKN-1-645auf8n7ww8an7m4e03m942w34d49dszg705u3d95xzjdpaf9-cvjgnka7av3tfz7i3cdfv2skp \
    10.10.56.1:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
### 查看管理节点token
```
$ docker swarm join-token worker
```
显示的信息和创建管理节点一样。

### 增加节点
在worker1(ip 10.10.56.2)上执行：
```
    docker swarm join \
    --token SWMTKN-1-645auf8n7ww8an7m4e03m942w34d49dszg705u3d95xzjdpaf9-cvjgnka7av3tfz7i3cdfv2skp \
    10.10.56.1:2377
```
就将worker1这个节点加入了swarm集群。
### 创建服务
```
docker service create --replicas 2 --name helloworld alpine ping qq.com
docker service create --replicas 2 --name nginx --publish 8084:80 nginx
```
--publish参数使nginx服务对swarm之外暴露了8084端口。可以通过swarm集群任何节点的8084端口都可以访问nginx服务，即使这个节点没有分配到nginx容器。
swarm集群对外暴露端口使用的ingress网络。而swarm内部通信使用overlay网络。
上述命令只能在管理节点执行。
### 查看服务
```
docker service ls
docker service ps helloworld
```
会显示容器和节点清单。到具体节点，也可用docker ps命令查看节点内的容器。
### 服务的扩容（水平扩展）
```
docker service scale nginx=3
```
### 删除服务
```
docker service rm nginx
```
# swarm服务的ingress网络模式
### ingress网络
创建服务时使用--publish参数来定义网络模式。在这一网络模式下，所有swarm节点均监听指定的外部网络端口，然后把请求反向代理到容器，即使这个节点上没有运行服务的容器。如下图（暴露的8080端口）：
![ingress网络](https://github.com/docker/docker/raw/master/docs/swarm/images/ingress-routing-mesh.png)

正在运行的服务，可以用update命令来暴露外部端口：
```
$ docker service update \
  --publish-add <PUBLISHED-PORT>:<TARGET-PORT>  <SERVICE>
```
查看服务暴露的端口：
```
$ docker service inspect --format="{{json .Endpoint.Spec.Ports}}" nginx
[{"Protocol":"tcp","TargetPort":80,"PublishedPort":8084}]
```
使用外部负载均衡器下的ingress网络示意图(暴露的8080端口)：
![](https://github.com/docker/docker/blob/master/docs/swarm/images/ingress-lb.png)
ingress网络模式下，外部负载均衡器将所有的节点都当成上游服务器。这样的好处是，即使swarm重新调度了运行服务任务的节点，负载均衡设置不用修改。

# swarm服务的定制网络

### 创建网络
```
docker network create --driver overlay nginx-network
78lax1rznqaim8uq31u68378b
```
### 查看网络
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
...
78lax1rznqai        nginx-network       overlay             swarm
```
### 创建带定制网络的服务
```
$ docker service create --replicas 1 --name nginx \
  --network nginx-network nginx
ds3nfquip9g1waylznyuycgke
```
查看nginx服务所在的节点：
```
$ docker service ps nginx
ID                         NAME     IMAGE  NODE   DESIRED STATE  CURRENT STATE           ERROR
79v9kauluouq2ftkxxqti72j2  nginx.1  nginx  node3  Running        Running 48 seconds ago
```
可以看到任务运行在node3，可以到node3去执行：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
0edc5d68066c        nginx:latest        "nginx"             3 minutes ago       Up 3 minutes        80/tcp, 443/tcp     nginx.1.79v9kauluouq2ftkxxqti72j2
```
然后执行```curl 10.10.56.3:8084```就找不到页面了。而未使用专用网络时，整个swarm集群的所有节点都可以在8084端口显示nginx首页。

### 负载均衡
swarm的内部负载均衡器自动将请求分发给服务的某个活动容器。会话粘性怎么实现？

### [测试swarm网络](https://github.com/docker/docker/blob/master/docs/swarm/networking.md)
1.在已有网络上创建服务：
```
docker service create --name my-busybox \
  --network nginx-network busybox sleep 3000
1tvrcg69rwi7eu9zwb6p58ruu
```
2.查看运行服务的节点
```
docker service ps my-busybox
ID                         NAME          IMAGE    NODE   DESIRED STATE  CURRENT STATE               ERROR
b9zrubs9ka6h1819wflr815xe  my-busybox.1  busybox  node1  Running        Running about a minute ago
```
在node1上执行命令：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
721e3822964a        busybox:latest      "sleep 3000"        2 minutes ago       Up 2 minutes                            my-busybox.1.b9zrubs9ka6h1819wflr815xe
```
(可以把b9zrubs9ka6h1819wflr815xe看作任务id？)
3. 在节点2上通过下列命令打开一个容器内部命令行
```
docker exec -it my-busybox.1.b9zrubs9ka6h1819wflr815xe /bin/sh
```
容器id构成算法是<TASK-NAME>+<ID>，em8ucnvb9db9qh59xzf4y0gyo是task id。在这个节点2上用docker ps命令可以看到这个容器id。
4. 在内部命令行中查找服务nginx的VIP:
```
$ nslookup nginx

Server:    127.0.0.11
Address 1: 127.0.0.11

Name:      nginx
Address 1: 10.0.0.2
```
再执行nslookup my-busybox，可以看到服务my-busybox的VIP是10.0.0.4。
5. 查看服务nginx的所有容器的虚拟IP地址。
```
/ # nslookup tasks.nginx
Server:    127.0.0.11
Address 1: 127.0.0.11
Name:      tasks.nginx
Address 1: 10.0.0.3 nginx.1.79v9kauluouq2ftkxxqti72j2.nginx-network

/ # nslookup tasks.my-busybox
Server:    127.0.0.11
Address 1: 127.0.0.11
Name:      tasks.my-busybox
Address 1: 10.0.0.5 721e3822964a
```

6. 在buybox服务的内部命令行中访问nginx服务：
```
$ wget -O- nginx

Connecting to nginx (10.0.0.2:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on Debian!</title>
...snip...
```
虽然nginx服务的容器有3个，但实测一直都是转发到了10.0.0.2，应存在IP粘性。而下面的dns轮询参数则去掉了粘性。

### DNS轮询服务
创建服务时增加```--endpoint-mode dnsrr```参数，可以创建DNS轮询服务。
```
 docker service create --replicas 3 --name nginx \
   --network nginx-network --endpoint-mode dnsrr nginx
```
这种方式创建的服务，请求会被随机发送到服务的各个容器。用上面的busybox测试确实是这样的。

### 绑定卷的服务
可以创建绑定卷的服务：
```
docker service create --mount src=<VOLUME-NAME>,dst=<CONTAINER-PATH> \
  --name myservice <IMAGE>
```
前提是所有节点要都存在卷指定的目录。swarm的调度器随时可能重新调度任务到其他的节点，所以本地卷要小心使用。

测试版安装脚本：https://experimental.docker.com/
测试过程中涉及多个节点，用Vbox克隆出多个虚机，改/etc/network/interfaces来改ip，改/etc/hostname来改主机名（ubuntu）。

# stack
stack是docker1.12.1正式版中没有，是experimental版的功能。
stack是多个service的集合，是docker-compose的替代品。docker-compose靠yml文件，stack的定义文件扩展名是dab(distributed application bundle)，更早的版本扩展名是dsb，是json格式文本文件。
定义一个hello-world.dab文件：
```
{
    "Version": "0.1",
    "Services": {
        "redis": {
            "Image": "redis@sha256:4b24131101fa0117bcaa18ac37055fffd9176aa1a240392bb8ea85e0be50f2ce",
            "Networks": ["default"]
        },
        "web": {
            "Image": "dockercloud/hello-world@sha256:fe79a2cfbd17eefc344fb8419420808df95a1e22d93b7f621a7399fd1e9dca1d",
            "Networks": ["default"],
            "User": "web"
        }
    }
}
```
部署这个stack的命令：
```
docker stack deploy hello-world --file hello-world.dab
```
查看部署的成果：
```
docker service ls
4rw6vetmmxxn  hello-world_redis  1/1       redis@sha256:4b24131101fa0117bcaa18ac37055fffd9176aa1a240392bb8ea85e0be50f2ce
ekx7q2z7iu3b  hello-world_web    0/1       dockercloud/hello-world@sha256:fe79a2cfbd17eefc344fb8419420808df95a1e22d93b
```
docker镜像id使用了full content hash格式的标记。