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

# 服务的定制网络

### 创建网络




测试版安装脚本：https://experimental.docker.com/
测试过程中涉及多个节点，用Vbox克隆出多个虚机，改/etc/network/interfaces来改ip，改/etc/hostname来改主机名（ubuntu）。