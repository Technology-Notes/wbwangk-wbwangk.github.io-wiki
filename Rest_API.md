各种流行开源软件的REST API节选：

#### openstack keystone 用户组
[1](https://developer.openstack.org/api-ref/identity/v3/index.html#groups)  
`GET /v3/groups`
List groups

 `POST /v3/groups`
Create group

 `GET /v3/groups/{group_id}`
Show group s

`PATCH /v3/groups/{group_id}`
Update group


 `DELETE /v3/groups/{group_id}`
Delete group

 `GET /v3/groups/{group_id}/users`
List users in group

 `PUT /v3/groups/{group_id}/users/{user_id}`
Add user to group


`HEAD /v3/groups/{group_id}/users/{user_id}`
Check whether user belongs to group

 `DELETE /v3/groups/{group_id}/users/{user_id}`
Remove user from group

#### openstack swift 对象
[1](https://developer.openstack.org/api-ref/object-store/)  
`GET /v1/{account}/{container}/{object}`  
Get object content and metadata


 `PUT /v1/{account}/{container}/{object}`  
Create or replace object


`COPY /v1/{account}/{container}/{object}`  
Copy object


 `DELETE /v1/{account}/{container}/{object}`
Delete object

`HEAD /v1/{account}/{container}/{object}`
Show object metadata

 `POST /v1/{account}/{container}/{object}`
Create or update object metadata