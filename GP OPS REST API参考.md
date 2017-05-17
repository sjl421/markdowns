[TOC]

## 描述
baseurl加在所有url前面，baseUrl的为http://ip:port/ops/api/v1
修改过后的目录:
    1.运维大盘
    2.服务中心
       2.1服务管理 --> 服务监控
       2.2实时sql
       2.3业务自检
       2.4冒烟测试
    3.集群管理
        3.1主机列表 --> 硬件监控
        3.2新建扩容
    4告警中心
        4.1告警管理
        4.2告警邮件设置
    5日志中心
        5.1操作日志
        5.2系统日志
        5.3GP操作记录
        5.4 审计日志
    6系统管理
        6.1用户管理
        6.2系统管理

## 1.运维大盘
<font color='blue'>角色数没意义，想把角色数给干掉,加上一个磁盘使用率，从spark运维平台来看，他们的运维大盘restapi来看和原先ads的更接近,显示图表直接强制显示内存和磁盘使用率，也没看到早期留我们的操作日志那部分</font>

- 集群预览

| URL           | ACTION | DESCRIPTION        |
| ------------- | ------ | ------------------ |
| /cluster_view | GET    | GP集群概览,查看GP集群当前使用量 |
前台请求参数:

```

```
后台返回JSON:
```
{
  "code": 0,
  "message": "OK",
  "data": {
    "ctrlHosts":2,
    "diskTotal": "10.5G",
    "diskPercnet": "25.6%",
    "dbCount": 3,
    "calculate_lived":2,
	"calculateHosts":"2",
	"calculate_died":0
  }
}
```

- 总表数

| URL          | ACTION | DESCRIPTION |
| ------------ | ------ | ----------- |
| /table_count | GET    | 查看GP用户表总数   |
前台请求参数:

```

```
后台返回JSON:
```
{
  "code": 0,
  "message": "OK",
  "data": {
	"tableCount":4
  }
}
```

- 集群概况

| URL         | ACTION | DESCRIPTION           |
| ----------- | ------ | --------------------- |
| /hosts_view | GET    | GP集群概览，能查看集群各进程在哪些机器上 |
前台请求参数:

```

```
后台返回参数:

```
{
      "code": 0,
      "message": "OK",
      "data":  [
          {
            "components": [
             {
               "componentName": "Master",
               "serviceName": "Greenplum"
             }],
             "hostName": "mdw1.com",
             "ip": "192.168.181.230"
          },
          {
            "components": [
             {
               "componentName": "Standby Master",
               "serviceName": "Greenplum"
             }],
             "hostName": "mdw2.com",
             "ip": "192.168.181.231"
          },
          {
            "components": [
             {
               "componentName": "primary segment 1",
               "serviceName": "Greenplum"nd"
             },
             {
               "componentName": "mirror segment 2",
               "serviceName": "Greenplum"
             }],
             "hostName": "sdw1.com",
             "ip": "192.168.181.232"
          }
       ]
}
```

- 集群内存使用率，cpu使用率等暂待不写

*TODO*
- 操作日志

| URL       | ACTION | DESCRIPTION |
| --------- | ------ | ----------- |
| /ops/logs | GET    | 查看集群操作日志    |
请求参数：

```
start：开始时间（非必须），格式为时间戳，单位为秒
end：结束时间（非必须）
keyword：关键字（非必须）
```
后台返回参数：
```json
{
  "message": "OK",
  "data": {
    "total": 40,
    "logs": [
      {
        "msg": "进入主页",
        "startTime": "2016-09-27",
        "userName": ""
      },
      {
        "msg": "test",
        "startTime": "2016-10-08",
        "userName": "345"
      },
      {
        "msg": "test",
        "startTime": "2016-10-08",
        "userName": "default user"
      }]
      }
}
```

## 2.服务中心

*此处api遵循了spark组的，原型设计中的监控中心-概览和监控中心-服务详情不复存在。但后面还是追加了gp概览和gp详情*


### 2.1 GP

- 集群预览
  *使用1运维大盘中获取集群状态的那个api,返回参数与其相同*

| URL         | ACTION | DESCRIPTION |
| ----------- | ------ | ----------- |
| /hosts_view | GET    | gp集群概览      |

- gp状态详情

| URL            | ACTION | DESCRIPTION |
| -------------- | ------ | ----------- |
| /hosts/service | GET    | gp集群状态详情    |

前台请求参数：

```

```
后台返回参数:

```
{
  "message": "OK",
  "data": [
    {
      "dbid": 1,
      "content": -1,
      "mode": "synced",
      "status": "up",
      "port": 5432,
      "hostname": "master",
      "address": "192.168.128.45",
      "serviceName": "master",
      "serviceRole": "master primary"
    },
    {
      "dbid": 6,
      "content": 0,
      "mode": "synced",
      "status": "up",
      "port": 50000,
      "hostname": "seg2",
      "address": "192.168.128.48",
      "replicationPort": "51000",
      "serviceName": "segment0",
      "serviceRole": "segment mirror"
    },
    {
      "dbid": 2,
      "content": 0,
      "mode": "synced",
      "status": "up",
      "port": 40000,
      "hostname": "seg1",
      "address": "192.168.128.47",
      "replicationPort": "41000",
      "serviceName": "segment0",
      "serviceRole": "segment primary"
    },
    {
      "dbid": 7,
      "content": 1,
      "mode": "synced",
      "status": "up",
      "port": 50001,
      "hostname": "seg2",
      "address": "192.168.128.48",
      "replicationPort": "51001",
      "serviceName": "segment1",
      "serviceRole": "segment mirror"
    },
    {
      "dbid": 3,
      "content": 1,
      "mode": "synced",
      "status": "up",
      "port": 40001,
      "hostname": "seg1",
      "address": "192.168.128.47",
      "replicationPort": "41001",
      "serviceName": "segment1",
      "serviceRole": "segment primary"
    },
    {
      "dbid": 8,
      "content": 2,
      "mode": "synced",
      "status": "up",
      "port": 50000,
      "hostname": "seg1",
      "address": "192.168.128.47",
      "replicationPort": "51000",
      "serviceName": "segment2",
      "serviceRole": "segment mirror"
    },
    {
      "dbid": 4,
      "content": 2,
      "mode": "synced",
      "status": "up",
      "port": 40000,
      "hostname": "seg2",
      "address": "192.168.128.48",
      "replicationPort": "41000",
      "serviceName": "segment2",
      "serviceRole": "segment primary"
    },
    {
      "dbid": 9,
      "content": 3,
      "mode": "synced",
      "status": "up",
      "port": 50001,
      "hostname": "seg1",
      "address": "192.168.128.47",
      "replicationPort": "51001",
      "serviceName": "segment3",
      "serviceRole": "segment mirror"
    },
    {
      "dbid": 5,
      "content": 3,
      "mode": "synced",
      "status": "up",
      "port": 40001,
      "hostname": "seg2",
      "address": "192.168.128.48",
      "replicationPort": "41001",
      "serviceName": "segment3",
      "serviceRole": "segment primary"
    }
  ]
}}
```

### 2.2 实时sql
- 实时sql

| URL         | ACTION | DESCRIPTION |
| ----------- | ------ | ----------- |
| /statements | GET    | 当前存在的会话     |

请求参数:

```
orderby:以哪个字段排序
asc：是否降序排列
```
返回参数:

```
{
	"code":0,
    "message":"OK",
    "data":{
    	"session":[
        	{"procpid":481,
            "sessId":120,
            "datname":"i3eye",
            "usename":"gpadmin",
            "currentQuery":"select * from pg_stat_activity",
            "queryStart":"2016-02-12 11:22:33",
            "waiting":"f",
            "waitngReason":null}
        ]
    }
}
```

### 2.3业务自检

- 获取业务自检项

| URL        | ACTION | DESCRIPTION |
| ---------- | ------ | ----------- |
| /diagnoses | GET    | 业务自检列表      |

请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":["greenplum","ops"]
    }
```

- 开启业务自检

| URL        | ACTION | DESCRIPTION |
| ---------- | ------ | ----------- |
| /diagnoses | POST   | 开始业务自检      |

请求参数:

```
request body :
["hosts","adb_vip"]//自检项内容从业务自检列表的返回值中取，不输入参数表示执行所有业务自检项
```
返回参数:

**返回参数中的1为此次业务自检的id，可根据id查看该次业务自检的结果,id为0则查看最近一次的业务自检**
```
{
	"code":0,
    "message": "OK",
    "data":1
}
```


- 业务自检概览

**id为0的话，就拉取最近一次的执行记录**

| URL             | ACTION | DESCRIPTION |
| --------------- | ------ | ----------- |
| /diagnoses/{id} | GET    | 业务自检概览      |

请求参数:
```

```
返回参数:
```
{
  "code": 0,
  "message": "OK",
  "data":
                  {
                      "succTaskCount": 5,
                      "failedTaskCount": 0,
                      "endTime": "2016-04-21 16:22:33",
                      "id": 1,
                      "startTime": "2016-04-21 16:22:33",
                      "status": "sucess",
                      "taskCount": 8,
                      "waitingTaskCount": 0,
                 	  "diagnoses":[
                      	{"name":"GPSTATE","startTime"："2016-04-21 16:22:33","endTime":1471434870501,"status":"success","outLog":"","errLog":"","diagnoseId":1},
                        {"name":"OPS","startTime"："2016-04-21 16:22:33","endTime":1471434870501,"status":"success","outLog":"","errLog":"","diagnoseId":1},
                      ]
                    }
}
```


### 2.4 冒烟测试
- 执行冒烟测试

| URL     | ACTION | DESCRIPTION |
| ------- | ------ | ----------- |
| /smokes | POST   | 执行冒烟测试      |

请求参数:

```
["conn","restart"]//冒烟测试项，有conn,restart,gpstate,reboot_recover,sql五项可选,不输入参数则表示执行全部冒烟测试项
```
* 与业务自检相同返回一个冒烟测试的id，根据id可以查看冒烟测试情况
  返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":1
    }
```

- 查看冒烟测试
  **id为0时返回最近一次冒烟测试结果**
| URL          | ACTION | DESCRIPTION |
| ------------ | ------ | ----------- |
| /smokes/{id} | GET    | 查看冒烟测试      |
请求参数:

```

```
返回参数
```
   {
  "message": "OK",
  "data": {
    "id": 92,
    "succTaskCount": 0,
    "failedTaskCount": 5,
    "endTime": "2016-10-19 09:39:09",
    "startTime": "2016-10-19 09:39:03",
    "status": "FAILED",
    "taskCount": 5,
    "waitingTaskCount": 0,
    "smokes": [
      {
        "name": "conn",
        "status": "FAILED",
        "startTime": "2016-10-19 17:41:06.0",
        "endTime": "2016-10-19 09:39:08.0",
        "outLog": "",
        "errLog": "python: can't open file '/etc/adb_scripts/manual_entry.py': [Errno 2] No such file or directory",
        "smokeId": 92
      },
      {
        "name": "restart",
        "status": "FAILED",
        "startTime": "2016-10-19 17:41:06.0",
        "endTime": "2016-10-19 09:39:09.0",
        "outLog": "",
        "errLog": "python: can't open file '/etc/adb_scripts/manual_entry.py': [Errno 2] No such file or directory",
        "smokeId": 92
      },
      {
        "name": "gpstate",
        "status": "FAILED",
        "startTime": "2016-10-19 17:41:06.0",
        "endTime": "2016-10-19 09:39:09.0",
        "outLog": "",
        "errLog": "python: can't open file '/etc/adb_scripts/manual_entry.py': [Errno 2] No such file or directory",
        "smokeId": 92
      },
      {
        "name": "reboot_recover",
        "status": "FAILED",
        "startTime": "2016-10-19 17:41:06.0",
        "endTime": "2016-10-19 09:39:09.0",
        "outLog": "",
        "errLog": "python: can't open file '/etc/adb_scripts/manual_entry.py': [Errno 2] No such file or directory",
        "smokeId": 92
      },
      {
        "name": "sql",
        "status": "FAILED",
        "startTime": "2016-10-19 17:41:06.0",
        "endTime": "2016-10-19 09:39:09.0",
        "outLog": "",
        "errLog": "python: can't open file '/etc/adb_scripts/manual_entry.py': [Errno 2] No such file or directory",
        "smokeId": 92
      }
    ]
  }
}
```

## 3. 集群管理
### 3.1 主机监控

| URL    | ACTION | DESCRIPTION |
| ------ | ------ | ----------- |
| /hosts | GET    | 监控主机的信息     |

请求参数:

```

```
返回参数:

```
{
      "code": 0,
      "message": "OK",
      "data": [
        {
          "cpu_count": null,
          "mem_total": null,
          "disk_total": null,
          "host_name": "gpmaster.com",
          "host_ip": "192.168.181.225"
        },
        {
          "cpu_count": null,
          "mem_total": null,
          "disk_total": null,
          "host_name": "gpseg1.com",
          "host_ip": "192.168.181.228"
        },
        {
          "cpu_count": null,
          "mem_total": null,
          "disk_total": null,
          "host_name": "gpseg2.com",
          "host_ip": "192.168.181.229"
        }
      ]
    }
```

### 3.2 硬件信息监控
| URL                     | ACTION | DESCRIPTION |
| ----------------------- | ------ | ----------- |
| /hosts/{host_name}/info | GET    | 硬件信息监控      |
请求参数:

```

```
返回参数：
```
{
  "code": 200,
  "message": "OK",
  "data": {
    "cpu_count": 4,
    "disk_used": 14.82,
    "mem_total": 3882464,
    "disk_total": 63.26,
    "load_avg": 0.06,
    "host_name": "dn1.com",
    "version": null,
    "os_type": "centos7",
    "ip": "192.168.128.56"
  }
}
```



### 3.3 扩容
#### 3.3.1 segment扩容
- segment扩容

| URL                | ACTION | DESCRIPTION |
| ------------------ | ------ | ----------- |
| /capacity/segments | POST   | 开始扩容        |
请求参数:
```
	header:
        Content-Type:application/json
    requestbody:
          {
              "expandNum": 2,
			  "hosts": "sdw1,sdw2,sdw3"
          }
```
返回:

```
	{
    	"data":"1486519009890"
    }
```

- segment扩容进度查询

| URL                | ACTION | DESCRIPTION |
| ------------------ | ------ | ----------- |
| /capacity/segments | GET    | 查看          |

请求参数:
```
{
	expandId:1486519009890
}
```
返回参数:
```
    {
        "code": 0,
        "message": "OK",
        "data": {
                "result": "FAILED"
              }
        }
```

- segment扩容失败回滚

| URL                | ACTION | DESCRIPTION |
| ------------------ | ------ | ----------- |
| /capacity/segments | DELETE | 回滚最近一次扩容操作  |
请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

#### 3.3.2 物理机扩容

- 物理机扩容

| URL                | ACTION | DESCRIPTION |
| ------------------ | ------ | ----------- |
| /capacity/newhosts | POST   | 开始物理机扩容     |
请求参数:
```
	header:
        Content-Type:application/json
    requestbody:
          {
              "expandNum":0,
			  "host_ip":"seg4.com:192.168.181.45,seg5.com:192.168.181.46"
          }
```
返回:

```
	{
        "data":1474961422130
    }
```

- 物理机扩容进度查询

| URL                | ACTION | DESCRIPTION |
| ------------------ | ------ | ----------- |
| /capacity/newhosts | GET    | 查看物理机扩容进度   |

请求参数:
```
{
	expandId:1474961422130
}
```
返回参数:
```
    {
        "code": 0,
        "message": "OK",
        "data": {
                "result": "FAILED"
              }
        }
```

### 3.4 主备切换
| URL             | ACTION | DESCRIPTION |
| --------------- | ------ | ----------- |
| /hosts/switchms | POST   | 主备切换        |

请求参数
```

```

返回参数：

```
    {
        "code": 0,
        "message": "OK",
        "data": "SUCCESS"
        }
```

## 4. 告警中心
### 4.1 告警管理
- 查看告警信息(api返回的信息会在1.1.0中变更，此处暂不删除)

  | URL               | ACTION | DESCRIPTION |
  | ----------------- | ------ | ----------- |
  | /notifices/alerts | GET    | 查看告警信息列表    |

  ​

请求参数：

```

```
返回参数：

```json
{
  "code": 200,
  "message": "OK",
  "data": [
    {
      "cluster_name": "aaa",
      "component_name": "AMBARI_SERVER",
      "definition_id": 63,
      "definition_name": "ambari_server_stale_alerts",
      "host_name": null,
      "id": 7,
      "instance": null,
      "label": "Ambari Server Alerts",
      "latest_timestamp": 1477029983706,
      "maintenance_state": "OFF",
      "original_timestamp": 1477029983706,
      "scope": "SERVICE",
      "service_name": "AMBARI",
      "state": "CRITICAL",
      "text": "There are 1 stale alerts from 1 host(s):\nnn.com\n  [Host Disk Usage (2m)]"
    },
    {
      "cluster_name": "aaa",
      "component_name": "NODEMANAGER",
      "definition_id": 48,
      "definition_name": "yarn_nodemanager_webui",
      "host_name": "nn.com",
      "id": 8,
      "instance": null,
      "label": "NodeManager Web UI",
      "latest_timestamp": 1477030230685,
      "maintenance_state": "ON",
      "original_timestamp": 1474646286619,
      "scope": "HOST",
      "service_name": "YARN",
      "state": "CRITICAL",
      "text": "Connection failed to http://nn.com:8042 (<urlopen error [Errno 111] Connection refused>)"
    }
    ]
}
```

* 获取集群告警

| URL               | ACTION | DESCRIPTION |
| ----------------- | ------ | ----------- |
| /notifices/alerts | GET    | 查看告警信息列表    |

说明：用来显示租户管理员界面的告警信息,同时包含一些正常的信息等；

若登录用户是集群管理员，返回的告警信息包括：

- 集群硬件告警
- 当前租户实例的告警（待进一步明确）

若登录用户是租户管理员，返回的告警信息包括：

- 当前租户实例的告警（待进一步明确）
- 当前租户实例服务的告警

```json
无
```

返回参数：

```json
{
	"code":"0",
	"message":"ok",
	"data":{
		"cluster":[	#改字段的信息只有当前登录用户是集群管理员时才有
			{
				"cluster_name": "adb_g1",
				"component_name": "AMBARI_AGENT",
				"definition_id": 13,
				"definition_name": "Host Disk Usage",
				"host_name": "m1.adb.g1.com",
				"id": 24,
				"instance": null,
				"label": "ADB Host Disk Usage",
				"latest_timestamp": 1492647572285,
				"maintenance_state": "OFF",
				"original_timestamp": 1489405052286,
				"scope": "SERVICE",
				"service_name": "AMBARI",
				"state": "CRITICAL",
				"text": "Capacity Used: [55.47%, 29.8 GB],Capacity Total: [53.7 GB], path=/usr/hdp"
			},
			...
		]
		"tenant1":[
			{
				"cluster_name": "adb_g1",
				"component_name": "AMBARI_AGENT",
				"definition_id": 13,
				"definition_name": "Host Disk Usage",
				"host_name": "m1.adb.g1.com",
				"id": 24,
				"instance": null,
				"label": "ADB Host Disk Usage",
				"latest_timestamp": 1492647572285,
				"maintenance_state": "OFF",
				"original_timestamp": 1489405052286,
				"scope": "SERVICE",
				"service_name": "AMBARI",
				"state": "CRITICAL",
				"text": "Capacity Used: [55.47%, 29.8 GB],Capacity Total: [53.7 GB], path=/usr/hdp"
			},
			...
		]
		"tenant2":[
			{
				"cluster_name": "adb_g1",
				"component_name": "AMBARI_AGENT",
				"definition_id": 13,
				"definition_name": "Host Disk Usage",
				"host_name": "m1.adb.g1.com",
				"id": 24,
				"instance": null,
				"label": "ADB Host Disk Usage",
				"latest_timestamp": 1492647572285,
				"maintenance_state": "OFF",
				"original_timestamp": 1489405052286,
				"scope": "SERVICE",
				"service_name": "AMBARI",
				"state": "CRITICAL",
				"text": "Capacity Used: [55.47%, 29.8 GB],Capacity Total: [53.7 GB], path=/usr/hdp"
			},
			...
		]
	}
}
```

* 获取当前告警的摘要

| URL                      | ACTION | DESCRIPTION |
| ------------------------ | ------ | ----------- |
| /notifices/alerts/summry | get    | 查看告警邮件      |

用于在用户登录的首页显示当前集群中所有error和warning的数量和简要信息

请求参数：

```json
无
```

返回参数：

```json
{
	"error":[
		{
			"cluster_name": "adb_g1",
			"component_name": "AMBARI_AGENT",
			"definition_id": 13,
			"definition_name": "Host Disk Usage",
			"host_name": "m1.adb.g1.com",
			"id": 24,
			"instance": null,
			"label": "ADB Host Disk Usage",
			"latest_timestamp": 1492647572285,
			"maintenance_state": "OFF",
			"original_timestamp": 1489405052286,
			"scope": "SERVICE",
			"service_name": "AMBARI",
			"state": "CRITICAL",
			"text": "Capacity Used: [55.47%, 29.8 GB],Capacity Total: [53.7 GB], path=/usr/hdp"
		},
		...
	]
	"warning":[
		{
			"cluster_name": "adb_g1",
			"component_name": "AMBARI_AGENT",
			"definition_id": 13,
			"definition_name": "Host Disk Usage",
			"host_name": "m1.adb.g1.com",
			"id": 24,
			"instance": null,
			"label": "ADB Host Disk Usage",
			"latest_timestamp": 1492647572285,
			"maintenance_state": "OFF",
			"original_timestamp": 1489405052286,
			"scope": "SERVICE",
			"service_name": "AMBARI",
			"state": "CRITICAL",
			"text": "Capacity Used: [55.47%, 29.8 GB],Capacity Total: [53.7 GB], path=/usr/hdp"
		},
		...
	]
}
```



### 4.2 告警邮件设置

- 查看告警邮件

| URL              | ACTION | DESCRIPTION |
| ---------------- | ------ | ----------- |
| /notifices/email | get    | 查看告警邮件      |
请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":
        {
        	"smtpServer"="smtp.company.com:25",
			"smtpUserid"="gpadmin@example.com",
            "smtpPassword": "",
            "emailFrom"="Greenplum Database <gpadmin@example.com>",
            "emailTo"="dba@example.com;John Smith <jsmith@example.com>"
        }
    }
```
- 修改告警邮件设置

| URL              | ACTION | DESCRIPTION |
| ---------------- | ------ | ----------- |
| /notifices/email | post   | 修改告警邮件      |
请求参数:

```
 		{
        	"smtpServer"="smtp.company.com:25",
			"smtpUserid"="gpadmin@example.com",
            "smtpPassword"="123456",
            "emailFrom"="Greenplum Database <gpadmin@example.com>",
            "emailTo"="dba@example.com;John Smith <jsmith@example.com>"
        }
```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":"master上运行gpstop -r 重启使修改生效"
    }
```

- 清空告警邮件设置

| URL              | ACTION | DESCRIPTION |
| ---------------- | ------ | ----------- |
| /notifices/email | DELETE | 清空告警邮件      |
请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":"master上运行gpstop -r 重启整集群使配置生效"
    }
```


## 5. 日志中心
### 5.1 服务日志列表

| URL           | ACTION | DESCRIPTION |
| ------------- | ------ | ----------- |
| /service/logs | GET    | 服务日志列表      |
请求参数:

```
{
	“start”:"1477030600(非必须)
    “end”:"1477030600",（非必须），格式为时间戳，单位为秒
    "key_word":"gpdb"（非必须）
}
```
返回参数:

```
{
      "code":"200",
      "message":"OK",
      "data":{
      	"total":100,
        "logs":[
          {
              "log_name":"/home/gpadmin/data/master/gpseg-1/pg_log/gpdb-2016-09-20_230544.csv",
              "host":"segment1.com",
              "component":"segment",
          },
          {
              "log_name":"/home/gpadmin/data/master/gpseg-1/pg_log/gpdb-2016-09-20_210511.csv",
              "host":"segment2.com",
              "component":"segment",
          }
        ]
      }
}
```

### 5.2 运维平台操作


| URL       | ACTION | DESCRIPTION |
| --------- | ------ | ----------- |
| /logs/ops | GET    | 操作日志列表      |
请求参数:

```
{
	“start”:"1477030600",（非必须）格式为时间戳，单位为秒
    “end”:"1477030600",（非必须）
    "key_word":"gpdb"（非必须）
}
```
返回参数:
```
	{
      "code": 200,
      "message": "OK",
      "data":
      {
      	"total":100,
        "logs":[
                    {
                      "msg": "Restart all components with Stale Configs for Greenplum",
                      "startTime": “2016-09-20”,
                      "userName": ""
                    },
                    {
                     "msg": "Restart all components with Stale Configs for Greenplum",
                     "startTime": “2016-09-20”,
                     "userName": ""
                    }
               ]
      }
   }
```

### 5.3 GP操作日志
| URL       | ACTION | DESCRIPTION |
| --------- | ------ | ----------- |
| /adb/logs | GET    | GP操作日志列表    |

请求参数:

```
{
	“start”:"1477030600",（非必须）
    “end”:"1477030600"（非必须）
}
```
返回参数：

```
	{
    	"code":0,
        "message":"OK",
        "data":{
        "total":100,
        "logs":[
        	{
            	"objname":"myqueue",
                "schemaname":null,
                "usename":"gpadmin",
                "usestatus":"CURRENT",
                "actionname":"CREATE",
                "subtype":"RESOURCE QUEUE",
                "statime":"2016-08-01 16:07:14"
            },
       		{
            	"objname":"gp_log_system",
                "schemaname":"gp_toolkit",
                "usename":"gpadmin",
                "usestatus":"CURRENT",
                "actionname":"CREATE",
                "subtype":"VIEW",
                "statime":"2016-08-01 16:07:14"
            }
        ]
        }
    }
```

## 5.4 租户的ADB操作日志 

| URL                                      | METHOD | DESCRIPTION    |
| ---------------------------------------- | ------ | -------------- |
| /logs/{tenantName}?start=1477030600,end=1477030600 | GET    | 获取当前租户的ADB操作日志 |

请求参数：

```json
无
```

返回参数：

```json
{
      "code":"0",
      "message":"OK",
      "data":{
      	"total":100,
        "logs":[
          {
              "log_name":"/home/gpadmin/data/master/gpseg-1/pg_log/gpdb-2016-09-20_230544.csv",
              "host":"segment1.com",
              "component":"segment",
          },
          {
              "log_name":"/home/gpadmin/data/master/gpseg-1/pg_log/gpdb-2016-09-20_210511.csv",
              "host":"segment2.com",
              "component":"segment",
          }
        ]
      }
}
```



## 6. 系统管理

### 6.1 用户管理
- 用户列表

| URL                 | ACTION | DESCRIPTION     |
| ------------------- | ------ | --------------- |
| /{tenantName}/users | GET    | 获取Greenplum用户列表 |
请求参数:

```

```
返回参数:

```
{
	"code":0,
    "message":"OK",
    "data":[{"resourceQueue":"pg_default","userName":"gpadmin","super":true}],
    {"resourceQueue":"rq1","userName":"test","super":true}]
}
```

- 添加用户

替换掉原有的添加用户接口。请求参数super表示是否为高级用户，高级用户多一个createdb的权限。密码为默认密码，需要该用户自行修改。


| URL                            | ACTION | DESCRIPTION |
| ------------------------------ | ------ | ----------- |
| /{tenantName}/users/{username} | POST   | 添加GP用户      |

请求参数:

```
request body:
	{
        "super":true,
        "resourceQueue":"pg_default",
    }
```

返回参数:
```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

- 删除用户

| URL                            | ACTION | DESCRIPTION |
| ------------------------------ | ------ | ----------- |
| /{tenantName}/users/{userName} | DELETE | 删除用户        |

请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

- 修改用户

替换掉1.0的修改GP用户，不能修改密码，只能重置密码

| URL                            | ACTION | DESCRIPTION |
| ------------------------------ | ------ | ----------- |
| /{tenantName}/users/{userName} | PUT    | 修改GP用户      |

请求参数:


 ```
 resourceQueue指向新的资源队列，restPassword重置回默认密码
	{
        "resourceQueue":null,
        "restPassword":true
    }
 ```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

### 6.2 资源队列管理
- 查看资源队列

| URL                    | ACTION | DESCRIPTION |
| ---------------------- | ------ | ----------- |
| /{tenantName}/resqueue | GET    | 查看资源队列      |
请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":[
        	{
            	"rsqname":"pg_default",
                "rsqcountlimit":20,
                "rsqcostlimit":"-1",
                "rsqcountvalue":0,
                "rsqcostvalue":0,
                "rsqmemorylimit":"8192MB",
                "rsqmemoryvalue":0,
                "rsqwaiters":0
        ]
    }
```

- 增加资源队列

| URL                    | ACTION | DESCRIPTION |
| ---------------------- | ------ | ----------- |
| /{tenantName}/resqueue | POST   | 增加资源队列      |
请求参数:
"priority":{MIN|LOW|MEDIUM|HIGH|MAX}

```
requestbody:
	{	"resqueueName":"myQueue",
    	"maxCost":-1,
        "minCost":0,
        "activeStatements":10,
 		"priority":MIN,
        "memoryLimit":"7000"//单位是MB 
    }
```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
		"data":null
    }
```

- 修改资源队列

| URL                                    | ACTION | DESCRIPTION |
| -------------------------------------- | ------ | ----------- |
| /{tenantName}/resqueue/{resqueue_name} | PUT    | 修改资源队列      |
请求参数:

```
requestbody:

	{
    	"maxCost":-1,
        "minCost":0,
        "activeStatements":10,
 		"priority":MIN,
        "memoryLimit":"7000"//单位是MB
    }
```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
		"data":null
    }
```
- 删除资源队列

| URL                                    | ACTION | DESCRIPTION |
| -------------------------------------- | ------ | ----------- |
| /{tenantName}/resqueue/{resqueue_name} | DELETE | 删除资源队列      |
请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

### 6.3 系统管理 

- 一键关机

| URL               | ACTION | DESCRIPTION |
| ----------------- | ------ | ----------- |
| /cluster/shutdown | POST   | 一键关机        |

请求参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":null
    }
```

- 关机状态查询
| URL                     | ACTION | DESCRIPTION |
| ----------------------- | ------ | ----------- |
| /cluster/shutdown_state | GET    | 关机状态查询      |

请求参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":[
        	{
        		“ip”:"192.168.128.45"
                "state":"success"
        	},{
        		“ip”:"192.168.128.46"
                "state":"success"
        	},{
        		“ip”:"192.168.128.47"
                "state":"failed"
        	}
        ]
    }
```

- 版本查看

| URL              | ACTION | DESCRIPTION |
| ---------------- | ------ | ----------- |
| /cluster/version | GET    | 版本查看        |

请求参数:
```

```

返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":"1.0.0"
    }
```

## 7 DEBUG

*用于快速查看运维平台和gp是否可用的端口，不对外提供*

| URL          | ACTION | DESCRIPTION                         |
| ------------ | ------ | ----------------------------------- |
| /debug/mysql | GET    | 查看mysql数据库中表是否可用，返回表数据记录数，-1表示该表不可用 |

请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":	{
            "ops_log":"5",
            "diagnoses":"7",
            "diagnoses_detail":"26",
            "diagnoses_status":"18",
            "smokes_status":"53",
            "cluster_info":"4"
        }
    }
```

| URL        | ACTION | DESCRIPTION                      |
| ---------- | ------ | -------------------------------- |
| /debug/adb | GET    | 查看gp数据库中表是否可用，返回表数据记录数，-1表示该表不可用 |

请求参数:

```

```
返回参数:

```
	{
    	"code":0,
        "message":"OK",
        "data":	{
            "pg_tables":"88"
        }
    }
```
## 8多租户

- 创建租户

创建租户

| URL               | ACTION | DESCRIPTION |
| ----------------- | ------ | ----------- |
| /tenants/{tenant} | POST   | 创建租户        |

请求参数:

```json
{
    "admin":"admin_name",			//当前租户管理员
  	"description":"xxxxxx",			//描述信息
	"resource":{					//当前租户资源
    	"type": "STANDARD",
    	"segments":2
	}
}
```

返回参数：

```json
{
	"code":0,
    "message":"OK",
	"data":null
}
```

* 租户视图

用来显示当前集群中所有租户信息

| URL      | ACTION | DESCRIPTION   |
| -------- | ------ | ------------- |
| /tenants | GET    | 查看当前集群中所有租户信息 |

请求参数:

```
无
```

返回参数:

```json
{
  "code":0,
  "message":null,
  "data":[  
    {
      "name":"testadb",
      "admin":"adminname",
      "stage":"ALLOCATING",
      "segmentType":"STANDARD",
      "segmentCount":8,
      "createTime":"2017-03-29 16:22:32.0"，
      "connectionUrl":"192.168.181.215:5432"
    },
    {
      "name":"testadb2",
      "admin":"adminame",
      "stage":"ALLOCATING",
      "segmentType":"STANDARD",
      "segmentCount":2,
      "createTime":"2017-03-29 16:22:37.0",
      "connectionUrl":"192.168.181.215:5432"
    }
  ]
}

```

- 租户详情（ADB实例）

显示指定租户所对应的ADB实例信息

| URL               | METHOD | DESCRIPTION |
| ----------------- | ------ | ----------- |
| /tenants/{tenant} | GET    | 查看指定租户实例的详情 |

```json
#在此版本处暂不考虑剩余资源量
{
  "code": "0",
  "message": "OK",
  "data": {
    "name": "testadb1",
    "admin": "user3",
    "masterCount": 2,
    "segmentCount": 2,
    "segmentType": "STANDARD",
	"state":"xxxx",
    "connectionUrl": "192.168.181.211:5432",
    "creator": "user3",
    "createdTime": "2017-04-20 14:14:26.0",
    "description": "for zhejiang",
    "resource": {
      "cpu": 20,
      "mem": 48,
      "disk": 1024
    }
  }
}
```

- 重启租户实例详情  

用来显示当前集群中所有租户信息

| URL（待确定）                  | METHOD | DESCRIPTION |
| ------------------------- | ------ | ----------- |
| /tenants/{tenant}/restart | PUT    | 重启租户实例      |

请求参数:

```
无
```

返回参数：

```json
{
	"code":0,
    "message":"OK",
	"data":null
}
```

- 删除租户实例

用来显示当前集群中所有租户信息

| URL               | ACTION | DESCRIPTION |
| ----------------- | ------ | ----------- |
| /tenants/{tenant} | DELETE | 删除租户        |

请求参数:

```
无
```

返回参数：

```json
{
	"code":0,
    "message":"OK",
	"data":null
}
```

- 扩容租户实例

用来显示当前集群中所有租户信息

| URL                          | ACTION | DESCRIPTION |
| ---------------------------- | ------ | ----------- |
| /tenants/{tenant}?segments=2 | PUT    | 扩容指定租户资源    |

请求参数:

```json
#填需要扩容的计算单元数目
#url参数
segments:2#扩容2个segment
例如一台机器的segments的信息
 "segments": [
   {
            "hostname": "seg1.com",
            "dir": "/gpadmin/adb_1",
            "segmentCount": 3
        },
        {
            "hostname": "seg2.com",
            "dir": "/gpadmin/adb_2",
            "segmentCount": 3
        }
    ]
如上所示，该adb实例占据了两台计算节点，每个计算节点上有3个segment，则该实例的扩容步长为2*n或3+n。2为所有的计算节点的数目。3为一台机器上总共有的segment数目。只要资源足够，n可以为任意数。
```

返回参数：

```json

```

- 修改租户实例

修改租户实例（这里仅限修改租户实例的管理员）

| URL                         | METHOD | DESCRIPTION |
| --------------------------- | ------ | ----------- |
| /tenants/{tenant}?admin=xxx | PUT    | 修改制定租户      |

请求参数:

```json
无
```

返回参数：

```json
{
	"code":0,
    "message":"OK",
	"data":null
}
```

- ADB实例资源详情

显示指定租户的资源详情

| URL                        | METHOD | DESCRIPTION   |
| -------------------------- | ------ | ------------- |
| /tenants/resource/{tenant} | GET    | 查看集群中指定租户资源详情 |

请求参数:

```
无
```

返回参数：

```json
#待确定，设计稿中没有相关信息
{
     "adbId": 23,
   	 "hostname":"c1.adb.com",
     "resource":{
       "cpu":8,
       "memory":1024,
       "diskSpace":512
     },
    "available":{
       "cpu":4,
       "memory":512,
       "diskSpace":256
     }
 }
```

- ADB资源使用详情

请求不定数目个ADB实例资源使用详情：

| URL                                | METHOD | DESCRIPTION   |
| ---------------------------------- | ------ | ------------- |
| /tenants/resource?name=user1,user2 | GET    | 查看集群中指定租户资源详情 |

请求参数:

```json

```

返回参数：

```json
#用来显示集群中所有主机的资源使用情况（下拉列表）；
#资源条百分比的显示只按照内存的使用情况（由于RM在分配资源的时候CPU，memory，dis是按照比例来分配的，所以拿内存来计算得到的百分比同样可以用来表征cpu和disk的使用情况）；
#备注：当鼠标放到资源条上某个租户信息时，要显示具体的CPU，memory，disk的使用量；
{  
  "code":0,
  "message": "OK",
  "data": [
    {
      "hostname": "master",
      "totall": {
        "cpu": 32,
        "mem": 98,
        "disk": 10240,
        "percent": 1.0
      },
      "resInUse": [
        {
          "tenantName": "testadb",
          "resource": {
            "cpu": 8,
            "mem": 16,
            "disk": 256,
            "percent": 0.1632653
          }
        },
        {
          "tenantName": "testadb2",
          "resource": {
            "cpu": 8,
            "mem": 16,
            "disk": 256,
            "percent": 0.1632653
          }
        }
      ],
      "other": {
        "cpu": 8,
        "mem": 16,
        "disk": 256,
        "percent": 0.1632653
      },
      "resRemain": {
        "cpu": 8,
        "mem": 50,
        "disk": 9472,
        "percent": 0.5102041
      }
    },
    {
      "hostname": "master2",
      "totall": {
        "cpu": 32,
        "mem": 192,
        "disk": 10240,
        "percent": 1.0
      },
      "resInUse": [
        {
          "tenantName": "testadb",
          "resource": {
            "cpu": 8,
            "mem": 16,
            "disk": 256,
            "percent": 0.083333336
          }
        },
        {
          "tenantName": "testadb2",
          "resource": {
            "cpu": 8,
            "mem": 16,
            "disk": 256,
            "percent": 0.083333336
          }
        }
      ],
      "other": {
        "cpu": 8,
        "mem": 16,
        "disk": 256,
        "percent": 0.083333336
      },
      "resRemain": {
        "cpu": 8,
        "mem": 144,
        "disk": 9472,
        "percent": 0.75
      }
    },
    {
      "hostname": "c1",
      "totall": {
        "cpu": 32,
        "mem": 255,
        "disk": 28001,
        "percent": 1.0
      },
      "resInUse": [
        {
          "tenantName": "testadb",
          "resource": {
            "cpu": 2,
            "mem": 8,
            "disk": 256,
            "percent": 0.03137255
          }
        },
        {
          "tenantName": "testadb2",
          "resource": {
            "cpu": 2,
            "mem": 8,
            "disk": 256,
            "percent": 0.03137255
          }
        }
      ],
      "other": {
        "cpu": 2,
        "mem": 8,
        "disk": 256,
        "percent": 0.03137255
      },
      "resRemain": {
        "cpu": 26,
        "mem": 231,
        "disk": 27233,
        "percent": 0.90588236
      }
    },
    {
      "hostname": "c2",
      "totall": {
        "cpu": 32,
        "mem": 255,
        "disk": 28001,
        "percent": 1.0
      },
      "resInUse": [
        {
          "tenantName": "testadb",
          "resource": {
            "cpu": 2,
            "mem": 8,
            "disk": 256,
            "percent": 0.03137255
          }
        },
        {
          "tenantName": "testadb2",
          "resource": {
            "cpu": 2,
            "mem": 8,
            "disk": 256,
            "percent": 0.03137255
          }
        }
      ],
      "other": {
        "cpu": 2,
        "mem": 8,
        "disk": 256,
        "percent": 0.03137255
      },
      "resRemain": {
        "cpu": 26,
        "mem": 231,
        "disk": 27233,
        "percent": 0.90588236
      }
    }
  ]
}
```

- 获取当前管理员下的所有的租户实例的列表

| URL                 | METHOD | DESCRIPTION          |
| ------------------- | ------ | -------------------- |
| /tenants/admin/list | GET    | 获取当前登录用户名下的所有的租户实例列表 |

获取当前登录用户名下的所有的租户实例列表，登录用户分为两种：运维管理员和租户管理员

请求参数:

```json
无
```

返回参数：	

```json
{
	"code":"0",
	"message":"ok",
	"data":{
		"loginname":"tenantadmin",
		"opsadmin":"true/false",
		"tenants":[
			"adb1",
			"adb2",
			"adb3"
		]
	}	
}
```



