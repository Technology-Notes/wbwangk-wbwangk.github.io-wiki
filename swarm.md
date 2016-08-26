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
```
docker network create --driver overlay nginx-network
```
### 查看网络
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5b0f3ab27531        bridge              bridge              local
2cf1a0ff25c1        docker_gwbridge     bridge              local
be31a37caa10        host                host                local
8db8mexh3ble        ingress             overlay             swarm
cwpdlwhn13zd        nginx-network       overlay             swarm
```
### 创建带定制网络的服务
```
docker service create --replicas 3 --name nginx \
  --network nginx-network nginx
```
实测时，nginx服务的3个实例分别运行在了node1和node2。然后执行```curl 10.10.56.3:8084```就找不到页面了。而未使用专用网络时，整个swarm集群的所有节点都可以在8084端口显示nginx首页。

### 负载均衡
swarm的内部负载均衡器自动将请求分发给服务的某个活动容器。会话粘性怎么实现？

### [测试swarm网络](https://github.com/docker/docker/blob/master/docs/swarm/networking.md)
1.在已有网络上创建服务：
```
docker service create --name my-busybox \
  --network nginx-network busybox sleep 3000
```
2.查看运行服务的节点
```
docker service ps my-busybox
ID                         NAME          IMAGE    NODE   DESIRED STATE  CURRENT STATE             ERROR
em8ucnvb9db9qh59xzf4y0gyo  my-busybox.1      busybox  node2  Running        Running 2 minutes ago
```
3. 在节点2上通过下列命令打开一个容器内部命令行
```
docker exec -it my-busybox.1.em8ucnvb9db9qh59xzf4y0gyo /bin/sh
```
容器id构成算法是<TASK-NAME>+<ID>。在这个节点2上用docker ps命令可以看到这个容器id。
4. 在内部命令行中查找服务nginx的VIP:
```
$ nslookup nginx

Server:    127.0.0.11
Address 1: 127.0.0.11

Name:      nginx
Address 1: 10.0.0.2
```
5. 查看服务nginx的所有容器的虚拟IP地址。
```
/ # nslookup tasks.nginx
Server:    127.0.0.11
Address 1: 127.0.0.11

Name:      tasks.nginx
Address 1: 10.0.0.5 nginx.2.11p5ygyzlva9mlip6bp3npa9j.nginx-network
Address 2: 10.0.0.3 nginx.3.d5bzw1qz9w1h6xyuqmfuzdr3y.nginx-network
Address 3: 10.0.0.4 nginx.1.14e5wpx0h6s442rrl628idj0u.nginx-network
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

测试版安装脚本：https://experimental.docker.com/
测试过程中涉及多个节点，用Vbox克隆出多个虚机，改/etc/network/interfaces来改ip，改/etc/hostname来改主机名（ubuntu）。