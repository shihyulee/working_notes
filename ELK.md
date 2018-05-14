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
> $ curl "http://localhost:9200/_cat/nodes"
 
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
    - /var/log/auth.log        
    - /var/log/syslog
    #- \Logs\*.*log*
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

## Nginx
1. 
> sudo apt-get install nginx 
2.
> sudo vi /etc/nginx/sites-available/default

replacing the original contest, and modify example.com
```
/etc/nginx/sites-available/default

- server {

-     listen 80;

-     server_name example.com;

-     auth_basic "Restricted Access";

-     auth_basic_user_file /etc/nginx/htpasswd.users;

-     location / {

-         proxy_pass http://localhost:5601;

-         proxy_http_version 1.1;

-         proxy_set_header Upgrade $http_upgrade;

-         proxy_set_header Connection 'upgrade';

-         proxy_set_header Host $host;

-         proxy_cache_bypass $http_upgrade;        

-     }

- }
```
3. sudo service nginx restart
## Reference
1. https://blog.johnwu.cc/article/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-red-hat.html
2. https://blog.csdn.net/zstack_org/article/details/60749112
3. https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/index.html
4. https://www.howtoing.com/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-16-04
5. https://www.linuxidc.com/Linux/2017-09/147092.htm
6. https://www.jianshu.com/p/43e3a2f437fd
7. https://ithelp.ithome.com.tw/articles/10186153