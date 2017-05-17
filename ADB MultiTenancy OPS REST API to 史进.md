# ADB MultiTenancy OPS REST API to 史进

* 显示ADB实例详情

显示指定指定ADB的实例详情

```json
#以下是示例：
{
  "code":0,
  "message":"OK",
  "data":{		
      "name":"ADB1",				
      "segmentType":"STANDARD",		
      "segmentCount":2,				
      "resource":{		//总资源
          "cpu":4,
          "memory":1024，
          "disk":512
      },
      "available":{			//剩余资源，按百分比
        "cpu":"20%",		//百分比
        "memory":"30%",		//百分比
        "dis":"50%"			//百分比
      }
  }
}
```

* 显示集群中资源视图

用来显示当前集群中所有租户及资源详情

```json
#显示集群中所有的host上的：
1. 每个租户的资源使用情况；
2. 当前host所有资源；
3. 当前host的剩余资源；
#实例：假如集群中共有有两台host：c1,c2,共有两个租户adb1,adb2，显示额
{  
  "code":0,
  "message":"ok",
  "data":[
    {
      "hostname":"c1.adb.com",		//显示当前host上所有ADB实例的资源使用情况
      "tenents_resource":[
        {
  			"instance":"adb1"		//adb1 的资源使用
         	"cpu":2,
			"memory":30
  			"disk":100
      	},
		{
         	"instance":"adb2",
             "cpu":2,
			 "memory":30
  			 "disk":100
		}
      ],
  	  "resource":{					//显示当前host整体资源
        	"cpu":32,
        	"memory":2048,
        	"disk":2048
  	  },
      "available":{					//显示当前host剩余可用资源
        	"cpu":2,
        	"memory":90,
        	"disk":512
      }
    },
    {
      "hostname":"c2.adb.com",
      "tenents_resource":[
        {
  			"instance":"adb1"
         	"cpu":2,
			"memory":30
  			"disk":100
      	},
		{
         	"instance":"adb2",
             "cpu":2,
			 "memory":30
  			 "disk":100
		}
      ],
  	  "resource":{
        	"cpu":32,
        	"memory":90,
        	"disk":512
  	  },
      "available":{
        	"cpu":2,
        	"memory":90,
        	"disk":512
      }
    }
  ]
}
```

* 显示指定ADB实例的资源详情

ops平台的最终需求：显示指定的adb实例的资源详情，一次请求的adb实例数目是不定的；

返回所要查询的adb实例的所在host上的所有用户的资源使用详情，其中包括：

* 该host 上每一个用户的资源详情，
* 该host上的总资源
* 该host上的剩余资源

假如说要查询adb1，adb2的资源使用详情：

当前系统中有3个adb实例：

adb1：c1.adb.com, c2.adb.com

adb2：c1.adb.com c3.adb.com

adb3： c2.adb.com, c3.adb.com

```json
#显示集群中所有的host上的：
1. 每个租户的资源使用情况；
2. 当前host所有资源；
3. 当前host的剩余资源；
#实例：假如集群中共有有两台host：c1,c2,共有两个租户adb1,adb2，显示额
{  
  "code":0,
  "message":"ok",
  "data":[
    {
      "hostname":"c1.adb.com",		//显示当前host上所有ADB实例的资源使用情况
      "tenents_resource":[
        {
  			"instance":"adb1",		//adb实例名
          	"id":1,					//adb id
         	"cpu":2,
			"memory":30
  			"disk":100
      	},
		{
         	"instance":"adb2",
          	"id":2
             "cpu":2,
			 "memory":30
  			 "disk":100
		}
      ],
  	  "resource":{					//显示当前host整体资源
        	"cpu":32,
        	"memory":2048,
        	"disk":2048
  	  },
      "available":{					//显示当前host剩余可用资源
        	"cpu":2,
        	"memory":90,
        	"disk":512
      }
    },
    {
      "hostname":"c2.adb.com",
      "tenents_resource":[
        {
  			"instance":"adb1"
          	"id":1,
         	"cpu":2,
			"memory":30
  			"disk":100
      	},
		{
         	"instance":"adb3",
          	 "id":3,
             "cpu":2,
			 "memory":30
  			 "disk":100
		}
      ],
  	  "resource":{
        	"cpu":32,
        	"memory":90,
        	"disk":512
  	  },
      "available":{
        	"cpu":2,
        	"memory":90,
        	"disk":512
      },
      {
      "hostname":"c3.adb.com",
      "tenents_resource":[
        {
  			"instance":"adb2"
      		"id":2,
         	"cpu":2,
			"memory":30
  			"disk":100
      	},
		{
         	"instance":"adb3",
          	"id":3,
             "cpu":2,
			 "memory":30
  			 "disk":100
		}
      ],
  	  "resource":{
        	"cpu":32,
        	"memory":90,
        	"disk":512
  	  },
      "available":{
        	"cpu":2,
        	"memory":90,
        	"disk":512
      }
    }
  ]
}
```

如果有什么问题我们随时沟通；