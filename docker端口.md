## docker swarm mode
[1](https://docs.docker.com/engine/swarm/networking/)  
Port 7946 TCP/UDP for container network discovery.  
Port 4789 UDP for the container overlay network.  
[2](https://github.com/docker/docker/blob/master/docs/swarm/swarm-tutorial/index.md)  
TCP port 2377 for cluster management communications  
TCP and UDP port 7946 for communication among nodes  
TCP and UDP port 4789 for overlay network traffic  
[3](https://github.com/docker/docker/blob/master/docs/swarm/ingress.md)  
Port 7946 TCP/UDP for container network discovery.  
Port 4789 UDP for the container ingress network.