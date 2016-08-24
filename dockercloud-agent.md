Registering in Docker Cloud via POST: https://cloud.docker.com/api/agent/v1/node/
2016/08/24 11:17:37 The token is empty. Please run 'dockercloud-agent set Token=xxx' first!

curl -Ls https://get.cloud.docker.com/ | sudo -H sh -s a810ca69aaae48c9ae09f480a8ca0260
dockercloud-agent set Token=a810ca69aaae48c9ae09f480a8ca0260

ubuntu配置文件：
/etc/dockercloud/agent/
ubuntu日志：
/var/log/dockercloud/
