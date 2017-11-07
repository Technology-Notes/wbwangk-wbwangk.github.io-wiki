DSP是Data Sharing Platform(数据共享平台)的简写。[Kong](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong)是一个基于Nginx的微服务网关软件。  
本文讲的是如何在DSP中利用Kong实现“服务”类型的数据资源共享问题。数据资源的共享有三种方式：数据库、文件、服务。  
Kong在DSP中的作用有：
- 反向代理(路由功能)，将外部系统请求路由到相应的服务器  
- 认证功能，对外部调用者的身份进行认证  
- 权限控制，对于“有条件共享”且为“接口”类型的数据资源进行访问控制  

## Kong的部署
Kong的安装参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong安装)。  
