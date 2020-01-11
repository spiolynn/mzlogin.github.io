---
layout:     post
title:      006-Beats平台-MetricBeat
subtitle:    "\"006-Beats平台-MetricBeat\""
date:       2018-11-09
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - ELK
    - DevOps
---

### Beats平台

[toc]

#### 0 引用

> https://www.elastic.co/guide/en/beats/metricbeat/5.6/metricbeat-getting-started.html <br> 官方文档

> http://docs.flycloud.me/docs/ELKStack/beats/index.html

> https://www.jianshu.com/p/2c0fb021497c <br> 使用说明

#### 1 metricbeat

##### 1 做什么

- 性能日志监控


```
Apache
HAProxy
MongoDB
MySQL
Nginx
PostgreSQL
Redis
System
Zookeeper
```

##### 2 安装

###### 1 下载

```
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.6.9-linux-x86_64.tar.gz
```

###### 2 配置

- Metricbeat uses modules to collect metrics（modules收集指标）默认只收集system module的信息

```
grep -Evn "^$|#" metricbeat.yml

11:metricbeat.modules:
14:- module: system
15:  metricsets:
17:    - cpu
20:    - load
26:    - diskio
29:    - filesystem
32:    - fsstat
35:    - memory
38:    - network
41:    - process
44:    - socket
45:  enabled: true
46:  period: 30s
47:  processes: ['.*']
55:name: "11.63.9.54"
72:output.elasticsearch:
74:  hosts: ["localhost:8021"]
75:  template.enabled: true
76:  template.path: "metricbeat.template.json"
77:  template.overwrite: false
78:  index: "top-%{+yyyy.MM.dd}"
```

- 默认情况下，Metricbeat loads the template automatically after successfully connecting to Elasticsearch，If the template already exists, it’s not overwritten。同时也可以人工指定

- 配置index模板导入
```
output.elasticsearch:
  hosts: ["localhost:9200"]
  template.name: "metricbeat"
  template.path: "metricbeat.template.json"
  template.overwrite: false
```
- 手工导入index模板

```
curl -H 'Content-Type: application/json' -XPUT 'http://localhost:9200/_template/metricbeat' -d@/etc/metricbeat/metricbeat.template.json

docker run --rm docker.elastic.co/beats/metricbeat:5.6.9 curl -H 'Content-Type: application/json' -XPUT 'http://localhost:9200/_template/metricbeat' -d@metricbeat.template.json
```

- 删除oldindex模板

```
curl -XDELETE 'http://localhost:9200/metricbeat-*'
```


###### 3 启动

```
     ./metricbeat  -c metricbeat.yml -e   启动命令
     ./metricbeat -e -c metricbeat.yml -d "publish"
```

###### 4 测试

```
curl -XGET 'http://localhost:9200/metricbeat-*/_search?pretty'
```
###### 5 导入dashboards

> Metricbeat comes packaged with the scripts/import_dashboards script that you can use to import the example dashboards, visualizations, and searches for Metricbeat. The script also creates an index pattern, metricbeat-*, for Metricbeat.

    ./scripts/import_dashboards  --load the dashboards
    ./scripts/import_dashboards -es http://localhost:8021
    ./import_dashboards -h
    ./scripts/import_dashboards -only-index
    ./scripts/import_dashboards -dir kibana/metricbeat //from a local directory: 
    ./scripts/import_dashboards -file metricbeat-dashboards-1.1.zip //from a local zip archive: 
    
    
    
> https://www.elastic.co/guide/en/beats/libbeat/5.6/import-dashboards.html


##### 3 metricbeat 命令

> https://www.elastic.co/guide/en/beats/metricbeat/5.6/command-line-options.html

```
./metricbeat -e -c metricbeat.yml -d "publish" -E name=mybeat
./metricbeat -c metricbeat.yml -d "publish" -E name=mybeat -path.logs=日志目录
./metricbeat -c metricbeat.yml -configtest 测试配置
```



```
-E <setting>=<value> : 覆盖metricbeat.yml里的参数信息
-N testing the Beat 不输出到output
-c <file> 
-e Log to stderr and disable syslog/file output.  (本地输出)
-d <selectors> Enable debugging  -d “*”
-httpprof [<host>]:<port> Start http server for profiling. This option is useful for troubleshooting and profiling the Beat. 
```

##### 4 metricbeat 工作原理

###### 1 event structure 事件构成

> Every event sent by Metricbeat has the same basic structure. It contains the following fields:

```
 @timestamp
    Time when the event was captured 
beat.hostname
    Hostname of the server on which the Beat is running 
beat.name
    Name given to the Beat  -- 建议修改
metricset.module
    Name of the module that the data is from 
metricset.name
    Name of the metricset that the data is from 
metricset.rtt
    Round trip time of the request in microseconds  -- 请求的往返时间 (微秒)

eg：
{
  "@timestamp": "2016-06-22T22:05:53.291Z",
  "beat": {
    "hostname": "host.example.com",
    "name": "host.example.com"
  },
  "metricset": {
    "module": "system",
    "name": "process",
    "rtt": 7419
  },
  .
  .
  .

  "type": "metricsets"
}
```

###### 2 event error structure 事件构成（error）
- The following example shows an error event sent when the Apache server is not reachable:
```
{
  "@timestamp": "2016-03-18T12:18:57.124Z",
  "apache-status": {},
  "beat": {
    "hostname": "host.example.com",
    "name": "host.example.com"
  },
  "error": "Get http://127.0.0.1/server-status?auto: dial tcp 127.0.0.1:80: getsockopt: connection refused",
  "metricset": {
    "module": "apache",
    "name": "status",
    "rtt": 1082
  },
  .
  .
  .

  "type": "metricsets"
```

###### 3 关键特性

> https://www.elastic.co/guide/en/beats/metricbeat/5.6/key-features.html


##### 5 metricbeat 配置

##### 变更配置
> https://www.elastic.co/guide/en/beats/metricbeat/5.6/using-environ-vars.html

```
ES_HOSTS="10.45.3.2:9220,10.45.3.1:9230"

output.elasticsearch:
  hosts: '${ES_HOSTS}'
```

- ${VAR}
- ${VAR:default_value}
- ${VAR:?error_text}



> metricbeat.full.yml 的配置
> https://www.elastic.co/guide/en/beats/metricbeat/5.6/metricbeat-configuration-options.html

###### 1 modules
```
栗子
metricbeat.modules:

#---------------------------- Apache Status Module ---------------------------
- module: apache
  metricsets: ["status"]
  enabled: true
  period: 1s
  hosts: ["http://127.0.0.1/"]
#---------------------------- MySQL Status Module ----------------------------
- module: mysql
  metricsets: ["status"]
  enabled: true
  period: 2s
  hosts: ["root@tcp(127.0.0.1:3306)/"]  
```


```
module:module name
metricsets: 要执行的 metricsets 的列表
enabled
period metricsets的收集时间
hosts：要从中获取信息的主机的列表
fields: dictionary field  will be sent with the metricset event
tags: list of tags that will be sent with the metricset event
```


###### 2 General

beat 本身的参数


```
name: "my-shipper"  --beat.name信息

```


###### 3 output

- Elasticsearch out

When you specify Elasticsearch for the output, the Beat sends the transactions directly to Elasticsearch by using the Elasticsearch HTTP API.

>https://www.elastic.co/guide/en/beats/metricbeat/5.6/elasticsearch-output.html#_compatibility

```
eg:

output.elasticsearch:
  enabled: true
  hosts: ["10.45.3.2:9220", "10.45.3.1:9230"]
  template.enabled: true
  template.path: "metricbeat.template.json"
  template.overwrite: false
  index: "metricbeat"  -- The default is "metricbeat-%{+yyyy.MM.dd}"
  indices:
    - index: "critical-%{+yyyy.MM.dd}"
      when.contains:
        message: "CRITICAL"
    - index: "error-%{+yyyy.MM.dd}"
      when.contains:
        message: "ERR"  
  ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
  ssl.certificate: "/etc/pki/client/cert.pem"
  ssl.key: "/etc/pki/client/cert.key"
  
To enable SSL, just add https to all URLs defined under hosts.
output.elasticsearch:
  hosts: ["https://localhost:9200"]
  username: "admin"
  password: "s3cr3t"
```

- logstach out

> 先决条件:  logstach 要安装了beats input 插件 `bin/logstash-plugin list`

```
output.logstash:
  hosts: ["localhost:5044"]
```

- fileout
输出是一个json格式，一般用来测试，同时也可以作为logstach的输入

```
output.file:
  path: "/tmp/metricbeat"
  filename: metricbeat-1
  rotate_every_kb: 10240
  codec.json:
    pretty: true  
    
  codec.format:
    string: '%{[@timestamp]} %{[message]}'
```

- Console output

```
output.console:
  pretty: true
```

- logging

```
logging.level: warning
logging.to_files: true
logging.to_syslog: false
logging.files:
  path: /var/log/mybeat
  name: mybeat.log
  keepfiles: 7
  permissions: 0600
```
###### 4 SSL

###### 5 LOGGNG PROCESSORS


### 2.3 troubleshooting

#### 1 官方论坛
>https://discuss.elastic.co/c/beats/metricbeat <br> 官方论坛

#### 2 debug
```
metricbeat -e -c mymetricbeatconfig.yml
metricbeat -e -d "publish"
metricbeat -e -d "*"
```



### 2 filebeat：日志监控

#### 1 configure

- filebeat.prospectors

```
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/apache/httpd-*.log

- input_type: log
  paths:
    - /var/log/messages
    - /var/log/*.log
```


### 3 modules system

```
 @timestamp 	
	May 25th 2018, 07:01:12.024
t _id 	
	AWOUY0JYGJ-l7YFt78zE
t _index 	
	racs_pm-2018.05.24
# _score 	
	3.137
t _type 	
	doc
t beat.hostname 	
	wn4oreligap1002
t beat.name 	
	11.63.9.40
t beat.version 	
	5.6.9
t metricset.module 	
	system
t metricset.name 	
	filesystem
# metricset.rtt 	
	412
# system.filesystem.available 	
	3,931,992,064
t system.filesystem.device_name 	
	/dev/mapper/vg00-vg00l4300
# system.filesystem.files 	
	327,680
# system.filesystem.free 	
	4,200,427,520
# system.filesystem.free_files 	
	313,306
t system.filesystem.mount_point 	
	/home/mw/weblogic
# system.filesystem.total 	
	5,150,212,096
# system.filesystem.used.bytes 	
	949,784,576
# system.filesystem.used.pct 	
	0.184
t tags 	
	racs_pm, ap
t type 	
	metricsets
```	



