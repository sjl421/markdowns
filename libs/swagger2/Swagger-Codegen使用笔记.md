# Swagger-Codegen使用笔记

首先得到swagger.json

在swagger2运行起来之后通过一下的rest api地址可以获得当前项目的api 文档的json格式数据;

```
http://localhost:8084/service_api/v2/api-docs
```

```json
{
    "swagger": "2.0",
    "info": {
        "description": "Api Documentation",
        "version": "1.0",
        "title": "Api Documentation",
        "termsOfService": "urn:tos",
        "contact": {},
        "license": {
            "name": "Apache 2.0",
            "url": "http://www.apache.org/licenses/LICENSE-2.0"
        }
    },
    "host": "localhost:8084",
    "basePath": "/service_api",
    "tags": [
        {
            "name": "debug-controller",
            "description": "Debug Controller"
        },
        {
            "name": "tenant-controller",
            "description": "Tenant Controller"
        },
        {
            "name": "user-controller",
            "description": "User Controller"
        },
        {
            "name": "login-controller",
            "description": "Login Controller"
        }
    ],
    "paths": {
        "/v1/debug": {
            "get": {
                "tags": [
                    "debug-controller"
                ],
                "summary": "defaultDebug",
                "operationId": "defaultDebugUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/debug/dms": {
            "get": {
                "tags": [
                    "debug-controller"
                ],
                "summary": "debugDmsDefault",
                "operationId": "debugDmsDefaultUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/debug/ops": {
            "get": {
                "tags": [
                    "debug-controller"
                ],
                "summary": "debugOpsDefault",
                "operationId": "debugOpsDefaultUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/debug/swagger": {
            "get": {
                "tags": [
                    "debug-controller"
                ],
                "summary": "debugSwagger",
                "operationId": "debugSwaggerUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/debug/swagger/test2": {
            "get": {
                "tags": [
                    "debug-controller"
                ],
                "summary": "swagger debug",
                "description": "notes xxx",
                "operationId": "debugSwaggerTestUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "id",
                        "in": "query",
                        "description": "id",
                        "required": true,
                        "type": "integer",
                        "format": "int32"
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/dms/login": {
            "post": {
                "tags": [
                    "login-controller"
                ],
                "summary": "dmsLogin",
                "operationId": "dmsLoginUsingPOST",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "in": "body",
                        "name": "reqBoby",
                        "description": "reqBoby",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/JsonObject"
                        }
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "201": {
                        "description": "Created"
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/tenants/{tenant}": {
            "post": {
                "tags": [
                    "tenant-controller"
                ],
                "summary": "addTenant",
                "operationId": "addTenantUsingPOST",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "loginName",
                        "in": "query",
                        "description": "loginName",
                        "required": false,
                        "type": "string"
                    },
                    {
                        "name": "loginId",
                        "in": "query",
                        "description": "loginId",
                        "required": false,
                        "type": "integer",
                        "format": "int32"
                    },
                    {
                        "in": "body",
                        "name": "jsonObject",
                        "description": "jsonObject",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/JsonObject"
                        }
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "201": {
                        "description": "Created"
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            },
            "delete": {
                "tags": [
                    "tenant-controller"
                ],
                "summary": "deleteTenant",
                "operationId": "deleteTenantUsingDELETE",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "loginName",
                        "in": "query",
                        "description": "loginName",
                        "required": false,
                        "type": "string"
                    },
                    {
                        "name": "loginId",
                        "in": "query",
                        "description": "loginId",
                        "required": false,
                        "type": "integer",
                        "format": "int32"
                    }
                ],
                "responses": {
                    "204": {
                        "description": "No Content"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/users/users/{tenant}/{username}/passwd": {
            "put": {
                "tags": [
                    "user-controller"
                ],
                "summary": "resetPasswd",
                "operationId": "resetPasswdUsingPUT",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "username",
                        "in": "path",
                        "description": "username",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "loginName",
                        "in": "query",
                        "description": "loginName",
                        "required": false,
                        "type": "string"
                    },
                    {
                        "name": "loginId",
                        "in": "query",
                        "description": "loginId",
                        "required": false,
                        "type": "integer",
                        "format": "int32"
                    },
                    {
                        "in": "body",
                        "name": "para",
                        "description": "para",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/JsonObject"
                        }
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "201": {
                        "description": "Created"
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/users/{tenant}": {
            "get": {
                "tags": [
                    "user-controller"
                ],
                "summary": "getUsersList",
                "operationId": "getUsersListUsingGET",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        },
        "/v1/users/{tenant}/{username}": {
            "post": {
                "tags": [
                    "user-controller"
                ],
                "summary": "createTenant",
                "operationId": "createTenantUsingPOST",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "username",
                        "in": "path",
                        "description": "username",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "loginName",
                        "in": "query",
                        "description": "loginName",
                        "required": false,
                        "type": "string"
                    },
                    {
                        "name": "loginId",
                        "in": "query",
                        "description": "loginId",
                        "required": false,
                        "type": "integer",
                        "format": "int32"
                    },
                    {
                        "in": "body",
                        "name": "para",
                        "description": "para",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/JsonObject"
                        }
                    }
                ],
                "responses": {
                    "404": {
                        "description": "Not Found"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "201": {
                        "description": "Created"
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            },
            "delete": {
                "tags": [
                    "user-controller"
                ],
                "summary": "deleteTenant",
                "operationId": "deleteTenantUsingDELETE_1",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "*/*"
                ],
                "parameters": [
                    {
                        "name": "tenant",
                        "in": "path",
                        "description": "tenant",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "username",
                        "in": "path",
                        "description": "username",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "loginName",
                        "in": "query",
                        "description": "loginName",
                        "required": false,
                        "type": "string"
                    },
                    {
                        "name": "loginId",
                        "in": "query",
                        "description": "loginId",
                        "required": false,
                        "type": "integer",
                        "format": "int32"
                    }
                ],
                "responses": {
                    "204": {
                        "description": "No Content"
                    },
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "object"
                        }
                    },
                    "401": {
                        "description": "Unauthorized"
                    },
                    "403": {
                        "description": "Forbidden"
                    }
                }
            }
        }
    },
    "definitions": {
        "JsonObject": {
            "type": "object"
        },
        "ErrorResponse": {
            "type": "object",
            "properties": {
                "error": {
                    "type": "integer",
                    "format": "int32",
                    "description": "error code"
                },
                "message": {
                    "type": "string",
                    "description": "Message"
                }
            }
        }
    }
}
```

## 将swagger.json转换成为java代码

在网址:http://editor.swagger.io/中粘贴进上上面的json格式的数据之后选择转换成`client`就可以下载到转换成的java代码了;



```
git clone https://github.com/swagger-api/swagger-codegen
cd swagger-codegen
mvn clean package
java -jar modules/swagger-codegen-cli/target/swagger-codegen-cli.jar generate \
   -i http://petstore.swagger.io/v2/swagger.json \
   -l php \
   -o /var/tmp/php_api_client
```

in windows

```
java -jar modules\swagger-codegen-cli\target\swagger-codegen-cli.jar generate -i http://petstore.swagger.io/v2/swagger.json -l php -o c:\temp\php_api_client
```

swagger生成的官网:

```
https://github.com/swagger-api/swagger-codegen
```

nazha:项目

```
http://maven.dtdream.com/content/groups/public/com/dtdream/emr/nezha-gen/1.2.2-SNAPSHOT/
```

```
[
    {
        "id": 1,
        "name": "tenant1",
        "stage": "RUNNING",
        "segmentType": "STANDARD",
        "segmentCount": 4,
        "createTime": "2017-08-28 10:12:06.0",
        "connectionUrl": "192.168.143.141:5432",
        "masterName": "tenant1-master",
        "standbyName": "tenant1-standby"
    }
]
```

```
{
    "id": 1,
    "name": "tenant1",
    "createTime": "2017-08-28 10:12:06.0",
    "connectionUrl": "192.168.143.141:5432",
    "segmentType": "STANDARD",
    "segmentCount": 4,
    "masterName": "tenant1-master",
    "standbyName": "tenant1-standby",
    "master": {
        "adbId": 1,
        "segId": 0,
        "segmentRole": "MASTER",
        "segmentType": "DOCKER",
        "hostname": "m1.adb.g1.com",
        "hostIp": "192.168.143.141",
        "dir": "/data0/tenant1",
        "diskSpace": 64,
        "cpuCores": "0,1,2,3,4,5,6,7",
        "memory": 16
    },
    "standby": {
        "adbId": 1,
        "segId": 0,
        "segmentRole": "STANDBY",
        "segmentType": "DOCKER",
        "hostname": "m2.adb.g1.com",
        "hostIp": "192.168.143.141",
        "dir": "/data0/tenant1",
        "diskSpace": 64,
        "cpuCores": "0,1,2,3,4,5,6,7",
        "memory": 16
    },
    "segments": [
        {
            "adbId": 1,
            "segId": 0,
            "segmentRole": "SEGMENT",
            "segmentType": "STANDARD",
            "hostname": "s1.adb.g1.com",
            "hostIp": "192.168.143.202",
            "dir": "/data0/tenant1",
            "diskSpace": 64,
            "cpuCores": "0,1",
            "memory": 16
        },
        {
            "adbId": 1,
            "segId": 0,
            "segmentRole": "SEGMENT",
            "segmentType": "STANDARD",
            "hostname": "s1.adb.g1.com",
            "hostIp": "192.168.143.202",
            "dir": "/data0/tenant1",
            "diskSpace": 64,
            "cpuCores": "0,1",
            "memory": 16
        },
        {
            "adbId": 1,
            "segId": 0,
            "segmentRole": "SEGMENT",
            "segmentType": "STANDARD",
            "hostname": "s2.adb.g1.com",
            "hostIp": "192.168.143.203",
            "dir": "/data0/tenant1",
            "diskSpace": 64,
            "cpuCores": "0,1",
            "memory": 16
        },
        {
            "adbId": 1,
            "segId": 0,
            "segmentRole": "SEGMENT",
            "segmentType": "STANDARD",
            "hostname": "s2.adb.g1.com",
            "hostIp": "192.168.143.203",
            "dir": "/data0/tenant1",
            "diskSpace": 64,
            "cpuCores": "0,1",
            "memory": 16
        }
    ]
}
```

```
{
	"services":[
      {
        "type":"tenant"
        "name":"tenant1",
        "action":"start"	//允许的类型有:start/stop/restart/delete
        "host":"192.168.143.78"
      },
      {
        "type":"tenant"
        "name":"tenant2",
        "action":"delete"	//允许的类型有:start/stop/restart/delete
        "host":"192.168.143.78"
      },
	]
}
```

```
{
	"services":[
      {
        "type":"tenant"
        "name":"tenant1",
        "action":"start"	//允许的类型有:start/stop/restart/delete
        "host":"192.168.143.78",
        "result":"ok"		// 成功的为ok, 失败的为fail
      },
      {
        "type":"tenant"
        "name":"tenant2",
        "action":"delete"	//允许的类型有:start/stop/restart/delete
        "host":"192.168.143.78",
        "result":"fail"
      },
	]
}
```

```
{
  "hosts":[
    	"192.168.143.100",
 		"192.168.143.101"  
	]
}
```

