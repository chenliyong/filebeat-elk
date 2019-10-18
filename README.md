# docker-compose ELK 读取docker中的日志
## 依赖组件
* docker.elastic.co/elasticsearch/elasticsearch:6.5.2
* docker.elastic.co/logstash/logstash:6.5.2
* docker.elastic.co/kibana/kibana:6.5.2
* docker.elastic.co/beats/filebeat:6.5.2

## 运行代码
```
doccker-compose up -d
```

## 错误处理
```
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
执行sysctl -w vm.max_map_count=262144
vm.max_map_count参数，是允许一个进程在VMAs拥有最大数量（VMA：虚拟内存地址， 一个连续的虚拟地址空间），当进程占用内存超过时， 直接OOM。
elasticsearch占用内存较高。官方要求max_map_count需要配置到最小262144。
max_map_count配置文件写在系统的/proc/sys/vm中
通过docker inspect命令， 可查看docker使用宿主机的/proc/sys作为只读路径之一。
说明镜像使用宿主机的max_map_count参数。因此直接修改宿主机的max_map_count参数即可
```
```
kibana添加index pattern卡住，通过浏览器查看请求返回状态为403 Forbidden，返回消息为：
{"message":"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];: [cluster_block_exception] blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];","statusCode":403,"error":"Forbidden"}
解决方法：设置es索引参数
# curl -XPUT -H 'Content-Type: application/json' http://es_server:9200/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'
# curl -XPUT -H 'Content-Type: application/json' http://es_server:9200/$index_name/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'
```