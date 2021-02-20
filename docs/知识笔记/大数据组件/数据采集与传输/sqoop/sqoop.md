# SQOOP



## sqoop部署

sqoop 部署依赖对应的jdbc连接jar包



连接 oracle 需要 oracle 的jar包 ojdbc6.jar

```sh
cp ojdbc6.jar /opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/sqoop/lib/
```





## sqoop从oracle抽到hive_orc

hive 默认是指的文本这个格式，其他的格式都要用到 hcatalog

https://www.cnblogs.com/EnzoDin/p/10653350.html

下面的是into 这种

```bash

updated="col1 > 3 "

sqoop import -D mapred.job.queue.name=root.default \
-D mapred.job.name=SQOOP \
--connect jdbc:oracle:thin:@//22.188.12.78:1521/orazyxf \
--username blck \
--password blck01! \
--query "SELECT * FROM BLCK.MQTEST WHERE ${updated} AND \$CONDITIONS"  \
--hcatalog-database bdp \
--hcatalog-table mqtest_orc \
--hcatalog-partition-keys day \
--hcatalog-partition-values 20210201 \
-m 1

```

* 这个 \$CONDITIONS 是 1=0 一定要填。

* 如果是oracle rac 对应的链接串需要修改，目前这个是 service 形式的串，如果是其他TNS 的串的化也要对应修改



> 官方文档说 hcatalog这个模式不能做overwrite
>
> Unsupported Sqoop Hive Import Options
>
> The following Sqoop Hive import options are not supported with HCatalog jobs.
>
> - `--hive-import`
> - `--hive-overwrite`



如果是要做 overwrite 

```sh
hdfs dfs -rm /user/hive/warehouse/bdp.db/mqtest_orc/day=20210201/
```



可选参数

```properties
--table BLCK.MQTEST 
```





## 官方文档

http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_new_command_line_options





## 已知约束

### hive分区字段问题

sqoop通过上述命令导数到hive分区表时，分区字段为 varchar时会出现异常。

建议分区字段使用 String