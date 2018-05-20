# ELK Stack Learning Notes
（Last Updated 2018.05.20）
1. 確認ElasticSearch, Logstash 和 Kibana版本一致。
2. 使用`filebeat` output to `logstash`時, 必須 load the template manually：

    deb and rpm:

    The First Step:
    > $ filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

    Second Step:    

    Delete the old documents from filebeat-* to force Kibana to look at the newest documents.
    > $ curl -XDELETE 'http://localhost:9200/filebeat-*'
    * Ref: https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-template.html
3. 修改`filebeat.yml`時，在output部份，elasticsearch和logstash只能二擇一。


## Useful Reference
* Ver 6.0 中文解說
> http://www.51niux.com/?id=201

> http://www.51niux.com/?id=202

> http://www.51niux.com/?id=203

> http://www.51niux.com/?id=204