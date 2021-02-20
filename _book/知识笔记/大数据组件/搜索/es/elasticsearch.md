# Elasticsearch



CDH 继承 es 管理


> 参考文档
>
> https://blog.csdn.net/guoliduo/article/details/105072857
>
> https://blog.csdn.net/weixin_39039055/article/details/113769701





问题分析：



```sh
## jdk问题
ln -s /data/app/jdk /usr/java/latest

## 启动权限问题
chmod 777 -R /opt/cloudera/parcels/ELASTICSEARCH-0.0.5.elasticsearch.p0.5

## 
## chown elasticsearch:elasticsearch -R /opt/cloudera/parcels/ELASTICSEARCH-0.0.5.elasticsearch.p0.5/


```





## 配置集群添加主节点配置



* 重点关注 参考文档2 中包含

错误2：使用时候报错，如http://cdh3:9200/_cat/nodes?pretty 时候报错

master_not_discovered_exception
解决：修改ES配置 增加masternodes 配置：cluster.initial_master_nodes: [“cdh3”]



CDH启动对应服务即可







# Kibana



