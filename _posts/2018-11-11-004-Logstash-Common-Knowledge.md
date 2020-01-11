---
layout:     post
title:      004-Logstash-Common-Knowledge
subtitle:    "\"004-Logstash-Common-Knowledge\""
date:       2018-11-09
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - ELK
    - DevOps
---
## 004-Logstash-Common-Knowledge
[TOC]

### 0 引用

> https://www.elastic.co/cn/products/logstash <br> 官方网站

> https://www.elastic.co/guide/en/logstash/5.6/introduction.html <br> 官方文档

> https://xuliugen.gitbooks.io/logstash-5-1/content/23%E3%80%81%E4%BD%BF%E7%94%A8logstash%E8%A7%A3%E6%9E%90%E6%97%A5%E5%BF%97.html <br> 中文文档 <br> https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/index.html

> https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns <br> grok已有的patterns信息

> http://grokdebug.herokuapp.com/ <br> grok debuger

> https://github.com/kkos/oniguruma/blob/master/doc/RE <br> 正则表达式

> https://blog.csdn.net/u011019726/article/details/78716553
### 1 what

#### 1 Logstash是具有实时流水线功能的开源数据收集引擎(统一来自不同来源的数据并规范化)

1 集中、转换和存储数据
Logstash 是开源的服务器端数据处理管道，能够同时 从多个来源采集数据、转换数据，然后将数据发送到您最喜欢的 “存储库” 中。（我们的存储库当然是 Elasticsearch。）

![image](https://www.elastic.co/guide/en/logstash/current/static/images/basic_logstash_pipeline.png)


#### 2 可以做哪些事情

- 处理所有类型的日志记录数据
    - 各种web logs apache、log4j
    - 支持syslog、event logs 等等
- 与filebeat结合、将日志进行安全的转发
- 完成各类性能日志的收集


### 1 安装

#### 0 前提

- java 需要是java8
`java -version`

#### 1 下载
```
- 下载
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.6.8.tar.gz
```

#### 2 安装以及安装检查

```
tar -zxvf logstash-5.6.8.tar.gz

在终端中，像下面这样运行命令来启动 Logstash 进程：

bin/logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
bin/logstash -e ''
bin/logstash -e 'input { stdin { } } output { stdout {} }'

sdf
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2018-05-11T02:17:39.477Z,
       "message" => "sdf"
}
```

#### 3 下载一个例子

```
wget https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz

```


### 2 测试
```
input {
  stdin {}
}
output {
  elasticsearch {
    action => "index"
    hosts => "192.168.56.102"
    index  => "test"  
  }
}

./logstash -f logstash-es-simple.conf

```

### 3 logstach基础


#### 0 语法

Logstash 设计了自己的 DSL 

- 区段（section）

Logstash 用 {} 来定义区域。区域内可以包括插件区域定义，你可以在一个区域内定义多个插件。插件区域内则可以定义键值对设置。示例如下：

```
input {
    stdin {}
    syslog {}
}
```

#### 0 数据类型
```
debug => true
host => "hostname"
port => 514
match => ["datetime", "UNIX", "ISO8601"]
options => {
    key1 => "value1",
    key2 => "value2"
}
```
#### 1 命令行参数

- -e

意即执行。我们在 "Hello World" 的时候已经用过这个参数了。事实上你可以不写任何具体配置，直接运行 bin/logstash -e ' ' 达到相同效果。这个参数的默认值是下面这样：
```
input {
    stdin { }
}
output {
    stdout { }
}
```

- -f 意即文件

```
bin/logstash -f agent.conf
```

- -t 意即测试
- -l 意即日志
```
bin/logstash -l logs/logstash.log
```

- -w

运行 filter 和 output 的 pipeline 线程数量。默认是 CPU 核数。

-  -b

每个 Logstash pipeline 线程，在执行具体的 filter 和 output 函数之前，最多能累积的日志条数。默认是 125 条。越大性能越好，同样也会消耗越多的 JVM 内存

- -u

每个 Logstash pipeline 线程，在打包批量日志的时候，最多等待几毫秒。默认是 5 ms。


#### 2 logstach 有的插件信息
```
[ci@localhost logstash-5.6.8]$ bin/logstash-plugin list
logstash-codec-cef
logstash-codec-collectd
logstash-codec-dots
logstash-codec-edn
logstash-codec-edn_lines
logstash-codec-es_bulk
logstash-codec-fluent
logstash-codec-graphite
logstash-codec-json
logstash-codec-json_lines
logstash-codec-line
logstash-codec-msgpack
logstash-codec-multiline
logstash-codec-netflow
logstash-codec-plain
logstash-codec-rubydebug
logstash-filter-cidr
logstash-filter-clone
logstash-filter-csv
logstash-filter-date
logstash-filter-dissect
logstash-filter-dns
logstash-filter-drop
logstash-filter-fingerprint
logstash-filter-geoip
logstash-filter-grok
logstash-filter-json
logstash-filter-kv
logstash-filter-metrics
logstash-filter-mutate
logstash-filter-ruby
logstash-filter-sleep
logstash-filter-split
logstash-filter-syslog_pri
logstash-filter-throttle
logstash-filter-translate
logstash-filter-urldecode
logstash-filter-useragent
logstash-filter-uuid
logstash-filter-xml
logstash-input-beats
logstash-input-couchdb_changes
logstash-input-dead_letter_queue
logstash-input-elasticsearch
logstash-input-exec
logstash-input-file
logstash-input-ganglia
logstash-input-gelf
logstash-input-generator
logstash-input-graphite
logstash-input-heartbeat
logstash-input-http
logstash-input-http_poller
logstash-input-imap
logstash-input-irc
logstash-input-jdbc
logstash-input-kafka
logstash-input-log4j
logstash-input-lumberjack
logstash-input-pipe
logstash-input-rabbitmq
logstash-input-redis
logstash-input-s3
logstash-input-snmptrap
logstash-input-sqs
logstash-input-stdin
logstash-input-syslog
logstash-input-tcp
logstash-input-twitter
logstash-input-udp
logstash-input-unix
logstash-input-xmpp
logstash-output-cloudwatch
logstash-output-csv
logstash-output-elasticsearch
logstash-output-file
logstash-output-graphite
logstash-output-http
logstash-output-irc
logstash-output-kafka
logstash-output-nagios
logstash-output-null
logstash-output-pagerduty
logstash-output-pipe
logstash-output-rabbitmq
logstash-output-redis
logstash-output-s3
logstash-output-sns
logstash-output-sqs
logstash-output-statsd
logstash-output-stdout
logstash-output-tcp
logstash-output-udp
logstash-output-webhdfs
logstash-output-xmpp
logstash-patterns-core
```

#### 3 input配置

##### 1 读取文件方式

###### 1 原理

Logstash 使用一个名叫 FileWatch 的 Ruby Gem 库来监听文件变化。这个库支持 glob 展开文件路径，而且会记录一个叫 .sincedb 的数据库文件来跟踪被监听的日志文件的当前读取位置。所以，不要担心 logstash 会漏过你的数据。

sincedb 文件中记录了每个被监听的文件的 inode, major number, minor number 和 pos。




#### 4 grok 过滤配置

- grok debugger 在线网站
- kibana xpact提供dev tools工具中支持该模式

##### 1 例子

- racs日志
```
[panzi][racs][com.panzi.springboot.log.CommonLogger.info(CommonLogger.java:24)][2018-05-17 15:34:50 123][INFO ][userMessage=登录用户ID41071106]

\[%{WORD:company}\]\[%{WORD:project}\]\[%{JAVAFILE:javapacket}\(%{JAVAFILE:classfile}\:%{NUMBER:classfileline}\)\]\[%{DEVTIMESTAMP_ISO8601:time}\]\[%{LOGLEVEL:level}%{GREEDYDATA:info}\]

```
- 其他日志
```
2017-03-07 00:03:44,373 4191949560 [          CASFilter.java:330:DEBUG]  entering doFilter()

\s*%{TIMESTAMP_ISO8601:time}\s*%{NUMBER:num} \[\s*%{JAVAFILE:class}\s*\:\s*%{NUMBER:lineNumber}\s*\:%{LOGLEVEL:level}\s*\]\s*(?<info>([\s\S]*))
```


##### 2 测试

- 写一个logstach config
```
vi test.conf
    
input {
  file {
    path => "/home/ap/ci/ELK/elk5/logstash-5.6.8/log.log"
  }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "\[%{WORD:company}\]\[%{WORD:project}\]\[%{JAVAFILE:javapacket}\(%{JAVAFILE:classfile}\:%{NUMBER:classfileline}\)\]\[%{DEVTIMESTAMP_ISO8601:time}\]\[%{LOGLEVEL:level}%{GREEDYDATA:info}\]" }
  }
  date{
    match => ["time", "yyyy-MM-dd HH:mm:ss SSS"]
    target => "@timestamp"      
  }
}
output{
    stdout{ codec => rubydebug }
}
```
- 写一个自定义patterns

```
DEVSECOND (?:(?:[0-5]?[0-9]|60)(?:[:.,[:space:]][0-9]+)?)
DEVTIMESTAMP_ISO8601 %{YEAR}-%{MONTHNUM}-%{MONTHDAY}[T ]%{HOUR}:?%{MINUTE}(?::?%{DEVSECOND})?%{ISO8601_TIMEZONE}?
```

- 运行
```
bin/logstash -f test.conf --config.test_and_exit
bin/logstash -f test.conf

echo "[panzi][racs][com.panzi.springboot.log.CommonLogger.info(CommonLogger.java:24)][2018-05-17 15:34:50 123][INFO ][userMessage=登录用户ID41071106]" >> log.log
```

- 将output改成数据库

```
input {
  file {
    path => "/home/ap/ci/ELK/elk5/logstash-5.6.8/log.log"
  }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "\[%{WORD:company}\]\[%{WORD:project}\]\[%{JAVAFILE:javapacket}\(%{JAVAFILE:classfile}\:%{NUMBER:classfileline}\)\]\[%{DEVTIMESTAMP_ISO8601:time}\]\[%{LOGLEVEL:level}%{GREEDYDATA:info}\]" }
  }
  date{
    match => ["time", "yyyy-MM-dd HH:mm:ss SSS"]
    target => "@timestamp"      
  }
}
output {
  elasticsearch {
    hosts => [ "localhost:8021" ]
    index => "aplog-%{+yyyy.MM.dd}"
  }
}
```

- 将input使用filebeat进行接入


- filebeat端配置
```
vi filebeatdevops.yml


filebeat.prospectors:
- input_type: log
  paths:
    - /home/ap/dev/log/racs_app.log
output.logstash:
  hosts: ["**.**.**.25:8055"]
```
- 启动filebeat
```
nohup ./filebeat -c filebeatdevops.yml &
```

- logstach端配置
```
input {
    beats {
        port => "8055"
    }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "\[%{WORD:company}\]\[%{WORD:project}\]\[%{JAVAFILE:javapacket}\(%{JAVAFILE:classfile}\:%{NUMBER:classfileline}\)\]\[%{DEVTIMESTAMP_ISO8601:time}\]\[%{LOGLEVEL:level}%{GREEDYDATA:info}\]" }
  }
  date{
    match => ["time", "yyyy-MM-dd HH:mm:ss SSS"]
    target => "@timestamp"      
  }
}
output {
  elasticsearch {
    hosts => [ "localhost:8021" ]
    index => "aplog-%{+yyyy.MM.dd}"
  }
}
```

- 启动logstach
```
curl -XDELETE 'http://localhost:8021/aplog*'
bin/logstash -f racsdev.conf --config.test_and_exit
nohup bin/logstash -f racsdev.conf &
```

- 验证
```
curl -X GET "localhost:8021/_cat/indices?v" | grep ap
```
