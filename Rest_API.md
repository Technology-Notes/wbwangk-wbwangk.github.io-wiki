各种流行开源软件的REST API节选：

### openstack keystone 用户组
[参考](https://developer.openstack.org/api-ref/identity/v3/index.html#groups)  
List groups:
```
GET /v3/groups
```

Create group:
```
POST /v3/groups
```
Show group:  
```
GET /v3/groups/{group_id}
```  
Update group:
```
PATCH /v3/groups/{group_id}
```  
Delete group:
```
DELETE /v3/groups/{group_id}
```  
List users in group:
```
GET /v3/groups/{group_id}/users
```  
Add user to group:
```
PUT /v3/groups/{group_id}/users/{user_id}
```  
Check whether user belongs to group:
```
HEAD /v3/groups/{group_id}/users/{user_id}
```  
Remove user from group:
```
DELETE /v3/groups/{group_id}/users/{user_id}
```  

### openstack swift 对象
[参考](https://developer.openstack.org/api-ref/object-store/)  
Get object content and metadata:
```
GET /v1/{account}/{container}/{object}
```  

Create or replace object:
```
PUT /v1/{account}/{container}/{object}
```  

Copy object:
```
COPY /v1/{account}/{container}/{object}
```  

Delete object:
```
DELETE /v1/{account}/{container}/{object}
```  
Show object metadata:
```
HEAD /v1/{account}/{container}/{object}
```
Create or update object metadata:
```
POST /v1/{account}/{container}/{object}
```  

### apache ambari 服务
[参考](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/service-resources.md)  
Returns a collection of the services for the cluster identified by ":name".  
```
GET /clusters/:name/services
```   

Gets the service information for the service identified by ":serviceName" for cluster identified by ":clusterName".  
```
GET /clusters/:clusterName/services/:serviceName
```  

Create the service identified by ":serviceName" in the cluster identified by ":clusterName".  
```
POST /clusters/:clusterName/services/:serviceName
```  

Update all services of the cluster identified by ":clusterName".  
```
PUT /clusters/:clusterName/services
```  

Update the service identified by ":serviceName" of the cluster identified by ":clusterName".  
```
PUT /clusters/:clusterName/services/:serviceName
```  
### Okta Groups API
[参考](https://developer.okta.com/docs/api/resources/groups.html)  
Add Group
```
POST /api/v1/groups
```
Get Group
```
GET /api/v1/groups/:id
```
List Groups
```
GET /api/v1/groups
```
Update Group
```
PUT /api/v1/groups/:id
```
Remove Group
```
DELETE /api/v1/groups/:id
```
List Group Members
```
GET /api/v1/groups/:id/users
```
Add User to Group
```
PUT /api/v1/groups/:gid/users/:uid
```
Remove User from Group
```
DELETE /api/v1/groups/:gid/users/:uid
```
List Assigned Applications
```
GET /api/v1/groups/:id/apps
```

### github文件API
[参考](https://github.com/imaidev/imaidev.github.io/wiki/github%E6%96%87%E4%BB%B6API%E6%B5%8B%E8%AF%95)  
取开源库的内容不需要认证。  
```
GET /repos/:owner/:repo/contents/:path
```
创建文件就需要认证了，需要在curl命令上增加-u参数。
```
PUT /repos/:owner/:repo/contents/:path
```
基本上与创建文件的API相同，只是输入的json串中多了一个"sha"对象，为要更新的文件哈希值。这个sha可以通过“取内容”的API返回。
```
PUT /repos/:owner/:repo/contents/:path
```
删除文件：
```
DELETE /repos/:owner/:repo/contents/:path
```