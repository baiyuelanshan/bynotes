 [toc]
 # Druid认证和授权管理api
 ## 创建tsdb_admin_role和tsdb_operator_role角色
 请求地址：http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/[role]  
 请求方式：POST
 
 示例：  
 tsdb_admin:  
 curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_admin_role++ -u druid_system:changeme
 
 tsdb_operator:  
 curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_operator_role++ -u druid_system:changeme
 
 ## 修改角色tsdb_admin和tsdb_operator的权限
 请求地址：http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/[role]/permissions/  
 请求方式：POST  
 
 示例：
 
curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_admin_role++/permissions/ -u druid_system:changeme -H 'Content-Type:application/json' -d @grant.json
 
curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_operator_role++/permissions/ -u druid_system:changeme -H 'Content-Type:application/json' -d @grant.json


tsdb_admin_role的权限:
```
[
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "WRITE"
    },
    {
        "resource": {
            "name": ".*",
            "type": "CONFIG"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "CONFIG"
        },
        "action": "WRITE"
    },
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "WRITE"
    }
]
```

tsdb_operator_role的权限：
```
[
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "WRITE"
    },
    {
        "resource": {
            "name": ".*",
            "type": "CONFIG"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "WRITE"
    }
]
```


## 权限说明
### 数据读
```
[
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "READ"
    }
]
```
### 数据写
```
[
    {
        "resource": {
            "name": ".*",
            "type": "DATASOURCE"
        },
        "action": "WRITE"
    }
]
```

### 数据配置
```
[
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "STATE"
        },
        "action": "WRITE"
    }
]
```

### 系统管理
```
[
    {
        "resource": {
            "name": ".*",
            "type": "CONFIG"
        },
        "action": "READ"
    },
    {
        "resource": {
            "name": ".*",
            "type": "CONFIG"
        },
        "action": "WRITE"
    }
]
```

## 获取角色的权限
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/[role]  
请求方式：GET
 
示例：
curl -X GET http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_test_role++ -u druid_system:changeme

返回值：
```
{"name":"tsdb_test_role","permissions":[{"resourceAction":{"resource":{"name":".*","type":"DATASOURCE"},"action":"READ"},"resourceNamePattern":".*"}]}
```
## 删除角色
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/[role]  
请求方式：DELETE  
示例：  
curl -X DELETE http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles/++tsdb_demo_role++




  

## 创建新用户
 创建新用户时，需要两个步骤，认证和授权：
 
认证请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/[user]  
授权请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/[user]  
请求方式：POST

示例：  
curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/++operator++ -u druid_system:changeme

curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/++operator++ -u druid_system:changeme

## 获取用户列表
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users  
请求方式：GET  

示例：  
curl -X GET http://10.92.208.219:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users -u druid_system:changeme  

返回值：
["druid_system","powertsdbadmin","newuser","operator"]


## 设置用户的角色
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/[user]/roles/[role]  
请求方式：POST  
示例：
curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/++operator++/roles/++tsdb_operator_role++ -u druid_system:changeme

## 删除用户的角色
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/[user]/roles/[role]  
请求方式：DELETE  
示例：curl -X DELETE http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/++admin++/roles/++tsdb_test_role++ -u druid_system:changeme


## 查看用户的角色
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/[user]  
请求方式：GET  
示例：  
curl -X GET http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/++operator++ -u druid_system:changeme


## 删除用户
删除一个用户时，需要同时删除它的认证和授权用户：  
删除认证用户地址：  
curl -X DELETE http://druid_coordinator_host:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/[user] -u druid_system:changeme  

删除授权用户地址：  
curl -X DELETE http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/[user] -u druid_system:changeme

请求方式：DELETE

示例：
curl -X DELETE http://10.92.208.219:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/++operator++ -u druid_system:changeme   

curl -X DELETE http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/users/++operator++ -u druid_system:changeme


## 设置/修改用户密码
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/[user]/credentials   
请求方式：POST

示例：  
curl -X POST http://10.92.208.219:8081/druid-ext/basic-security/authentication/db/BasicAuthenticator/users/++operator++/credentials -u druid_system:changeme -H 'Content-Type:application/json' -d @password.json


password.json:
```
{
    "password": "newpassword"
}
```

## 获取角色列表
请求地址：  
http://druid_coordinator_host:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles
请求方式：GET

示例：
curl -X GET http://10.92.208.219:8081/druid-ext/basic-security/authorization/db/BasicAuthorizer/roles -u druid_system:changeme

返回值：  
```
["admin","druid_system","newrole","tsdb_admin","tsdb_operator","tsdb_admin_role","tsdb_operator_role"]
```

