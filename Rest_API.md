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
