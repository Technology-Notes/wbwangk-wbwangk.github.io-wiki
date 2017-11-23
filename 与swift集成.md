openstack swift是一个分布式对象存储服务。对swift的访问有两种主要方式：  
- 后端访问，如java程序向swift存取文件 
- 前端访问，如通过js向swift存储文件  

在之前的swift使用实践中，都是采用了“后端访问”的方式。本文主要分析如何通过“前端访问”方式存取swift服务。  
“前端访问”方式的优势是架构简单，性能好，开发简单；劣势是对认证和授权的要求高。  

openstack的认证服务是keystone，应用需要先从keystone取得令牌，然后带着令牌才能访问swift的REST API。  

现在，应用自带了认证系统，必须让keystone信任应用的认证系统。为此，制定了以下约定：  
1. 自动在keystone中创建用户，用户id与应用中的用户id相同，密码都是1  
2. keystone只能被应用所在的IP访问，从而防止仿冒用户来获取令牌  


https://github.com/imaidev/imaidev.github.io/wiki/openstack-swift%E6%B5%8B%E8%AF%95#%E5%BA%94%E7%94%A8%E4%B8%8Eswift%E9%9B%86%E6%88%90