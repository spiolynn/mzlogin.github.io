---
layout:     post
title:      005-Beats平台-FileBeats
subtitle:    "\"005-Beats平台-FileBeats\""
date:       2018-11-09
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - ELK
    - DevOps
---
### FileBeat

#### 0 引用

> https://www.elastic.co/guide/en/beats/filebeat/5.6/configuration-filebeat-options.html <br> 官方文档


#### 1 what

- Filebeat is a log data shipper for local files
- Filebeat is a Beat, and it is based on the libbeat framework

#### 2 how

![image](https://www.elastic.co/guide/en/beats/filebeat/5.6/images/filebeat.png)

- it starts one or more prospectors（探矿） that look in the local paths you’ve specified for log files
- Each harvester（收割机） reads a single log file for new content and sends the new log data to the spooler（后台）

#### 3 安装

##### 1 安装

```
docker pull docker.elastic.co/beats/filebeat:5.6.9
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.9-linux-x86_64.tar.gz

解压
```

##### 2 配置

```
grep -Evn "^$|#" filebeat.yml

12:filebeat.prospectors:
18:- input_type: log
21:  paths:
22:    - /home/ap/ci/ELK/elk5/data/logstash-tutorial.log
       - /var/log/*.log
       - /var/log/*/*.log //子目录级别获取所有文件*log(但不从目录本身进行)
24:  tags: ["test.log"]
63:name: "11.63.9.54"
89:output.logstash:
91:  hosts: ["localhost:5044"]
   output.elasticsearch:
     hosts: ["localhost:8021"]

#常用的配置output是logstach和elasticsearch
```

##### 3 测试配置

```
echo 'ZONE="Asia/Shanghai"' > /etc/sysconfig/clock
./filebeat -configtest -e filebeat.yml
./filebeat -configtest -e filebeat-el.yml

./filebeat -e -c filebeat-el.yml -d "publish"
./filebeat -c filebeat-el.yml

curl 'localhost:8021/_cluster/health?level=indices&pretty'
curl -X GET "localhost:8021/_cat/indices?v"
curl -XDELETE 'http://localhost:8021/filebeat*'
```

##### 4 导入index模板（elaticsearch）

- 默认已经提供了filebeat的模板,默认情况下在链接了elatics加载模板，同时也支持自定义

```
output.elasticsearch:
  hosts: ["localhost:9200"]
  template.name: "filebeat"
  template.path: "filebeat.template.json"
  template.overwrite: false
```

- 支持手工导入模板的方式(删除模板)

```
curl -H 'Content-Type: application/json' -XPUT 'http://localhost:9200/_template/filebeat' -d@filebeat.template.json
curl -XDELETE 'http://localhost:9200/filebeat-*'
```

##### 5 导入kibana模板

```
./scripts/import_dashboards -only-index -es http://localhost:8021
./scripts/import_dashboards -dir /home/ap/ci/ELK/elk5/data/beats-dashboards-5.6.9/filebeat/ -es http://localhost:8021  -only-index

```

#### 2 命令行参数

>https://www.elastic.co/guide/en/beats/filebeat/5.6/command-line-options.html

#### 3 filebeat 工作原理


#### 4 filebeat 相关配置

- 日志源支持文件目录、多个日志文件、不同字符集、exclude_lines排除行（支持多个）、include_lines、exclude_files，增加tags标签

> https://www.elastic.co/guide/en/beats/filebeat/5.6/configuration-filebeat-options.html



