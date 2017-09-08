Ambari视图可运行在独立(standalone)Ambari服务器下。当Views运行在独立Ambari服务器下时，hadoop集群将是远程集群。可以将多个独立Ambari组服务用反向代理组成独立Ambari集群，需要反向代理支持用户会话粘性。  
与独立Ambari服务器相对称的是运维(operational)Ambari服务器，运维Ambari服务器管理着一个hadoop集群。  
创建Ambari视图的选项：
- 本地，视图要连接的集群时Ambari的本地集群
- 远程，视图要连接的集群时Ambari管理下远程集群
- 定制，视图要连接的集群是非Ambari管理的远程集群