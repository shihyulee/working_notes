# ELK Stack Install
## Prerequisite
* OS: Ubuntu 16.04
## Elasticsearch
1. Download
> https://www.elastic.co/downloads/elasticsearch
2. Install 
> $ sudo dpkg -i elasticsearch-*.deb
3. start 
> $ systemctl start elasticsearch
4. See the status
>$ systemctl status elasticsearch

>$ curl "http://localhost:9200/_cat/nodes"
5. Config
>$ sudo vi /etc/elasticsearch/elasticsearch.yml

modify
```
network.bind_host: 0.0.0.0
http.port: 9200
```
5. restart
> $ systemctl start elasticsearch
6. check
> $ curl "http://192.168.56.101:9200/_cat/nodes"
 
## Filebeat
1. Download
> https://www.elastic.co/downloads/beats/filebeat 
2. Install
> $ sudo dpkg -i filebeat-*.deb
3. Config
>$ sudo vi /etc/filebeat/filebeat.yml
```
filebeat.prospectors:
- input_type: log
  paths:
    # modify your LOG path
    - \Logs\*.*log*
output.elasticsearch:
  # modify your IP
  hosts: ["localhost:9200"]
  # index  
  index: "my-first-index"
```
4. check
> http://localhost:9200/_cat/indices

## Logstash
1. Download
> https://www.elastic.co/downloads/logstash
2. Install
> $ sudo dpkg -i logstash-*.deb
3. start 
> $ systemctl start logstash
4. See the status
>$ systemctl status logstash
5. Config
>$ sudo vi /etc/logstash/conf.d/pipeline.conf
Input
```
input {
  beats {
    port => 5044
  }
}
filter {
  grok {		
      match => [ "message", "%{TIMESTAMP_ISO8601:logTimestamp} \[%{NUMBER:thread}\] %{DATA:logType} %{DATA:logger} - %{GREEDYDATA:detail}" ]
  }
  mutate {
    add_tag => ["logstash"]
  }
}
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{+xxxx.ww}"
    document_type => "%{[@metadata][type]}"
  }
}
```
6. modify filebeat.yml
```
output.logstash:
  hosts: ["localhost:5044"]
  index: "my-second-index"
```
7. restart
> $ systemctl start logstash
8. add text.log in Log
```
2017-03-30 01:46:09,858 [1] INFO MyWebsite.Global - Application_Start
2017-03-30 01:46:10,311 [1] INFO MyWebsite.Global - Application_Start - Done - Spend time: [00:00:00.5069073]
2017-03-30 02:03:10,965 [14] INFO MyWebsite.Global - Application_End
```
9. open the browser
> http://localhost:9200/_search?pretty

## Kibana
1. Download
> https://www.elastic.co/downloads/kibana
2. Install 
> $ sudo dpkg -i kibana-*.deb
3. start 
> $ systemctl start kibana
4. See the status
> $ systemctl status kibana
5. config
> $ sudo vi /etc/kibana/kibana.yml

```
server.port: 5601
# 0.0.0.0 表示綁定所有 IP
server.host: "0.0.0.0"
```

6. open the browser
> http://localhost:5601

## Reference
1. https://blog.johnwu.cc/article/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-red-hat.html
2. https://ithelp.ithome.com.tw/articles/10186153
3. https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/index.html
4. https://www.howtoing.com/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-16-04
5. https://www.linuxidc.com/Linux/2017-09/147092.htm
6. https://www.jianshu.com/p/43e3a2f437fd