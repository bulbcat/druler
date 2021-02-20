# hdfs



[toc]

## 压缩



https://blog.csdn.net/weixin_33905756/article/details/91394345



txt -- zlib

orc -- zlib

python zlib



## 小文件问题



https://blog.csdn.net/weixin_42582592/article/details/85084575



## har 

官方文档

https://hadoop.apache.org/docs/r3.1.3/hadoop-archives/HadoopArchives.html

生成

```shell

##             har包名字              压哪                             用几个副本 输出到哪，默认hdfs上
hadoop archive -archiveName zoo.har  -p /foo/bar  [dir1 dir2 dir3 ]  -r 3       /outputdir


```

生成结果 原始文件不会清除，上面的命令是新生成一个har，为了节约namenode要清除掉以前的。



查看

```shell
hdfs dfs -ls har:///project/ccc.har
```



其实就是路径 前缀要用har,紧跟hdfs目录



hive 直接做归档

```sql

 create table mqtest  (col1 string ,col2 string ) partitioned by (day string )  ;
 insert into table  mqtest  partition (day=20190101) values ('3','2') ,('4','3') ;
 insert into table  mqtest  partition (day=20190101) values ('1','2') ,('2','3') ;

 create table mqtest_orc  (col1 string ,col2 string ) partitioned by (day string )  ;
 insert into table  mqtest_orc  partition (day=20190101) values ('3','2') ,('4','3') ;
 insert into table  mqtest_orc  partition (day=20190101) values ('1','2') ,('2','3') ;



set hive.archive.enabled=true;
set hive.archive.har.parentdir.settable=true;
set har.partfile.size=1099511627776;


ALTER TABLE mqtest ARCHIVE PARTITION(day=20190101);
ALTER TABLE mqtest_orc ARCHIVE PARTITION(day=20190101);

insert into mqtest partition (day=20190102) values ('3','4') ;

```



执行结果

```sql
    > ALTER TABLE mqtest ARCHIVE PARTITION(day=20190101); 
intermediate.archived is hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101_INTERMEDIATE_ARCHIVED
intermediate.original is hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101_INTERMEDIATE_ORIGINAL
Creating data.har for hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101
in hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101/.hive-staging_hive_2020-06-30_18-39-17_895_9088142084365566128-1/-ext-10000/partlevel
Please wait... (this may take a while)
20/06/30 18:39:18 INFO client.RMProxy: Connecting to ResourceManager at hsnn01/22.188.12.56:8032
20/06/30 18:39:18 INFO client.RMProxy: Connecting to ResourceManager at hsnn01/22.188.12.56:8032
20/06/30 18:39:18 INFO client.RMProxy: Connecting to ResourceManager at hsnn01/22.188.12.56:8032
Moving hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101/.hive-staging_hive_2020-06-30_18-39-17_895_9088142084365566128-1/-ext-10000/partlevel to hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101_INTERMEDIATE_ARCHIVED
Moving hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101 to hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101_INTERMEDIATE_ORIGINAL
Moving hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101_INTERMEDIATE_ARCHIVED to hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest/day=20190101
OK
Time taken: 23.844 seconds
hive> select * from mqtest ;
OK
3	2	20190101
4	3	20190101
1	2	20190101
2	3	20190101
Time taken: 0.245 seconds, Fetched: 4 row(s)
hive> show create table mqtest ;
OK
CREATE TABLE `mqtest`(
  `col1` string, 
  `col2` string)
PARTITIONED BY ( 
  `day` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://hsnn01:8020/user/hive/warehouse/bdp.db/mqtest'
TBLPROPERTIES (
  'transient_lastDdlTime'='1593513248')
Time taken: 0.211 seconds, Fetched: 15 row(s)
hive> dfs -ls /user/hive/warehouse/bdp.db/mqtest/  ;
Found 1 items
drwxr-xr-x   - bdp hive          0 2020-06-30 18:39 /user/hive/warehouse/bdp.db/mqtest/day=20190101
hive> dfs -ls /user/hive/warehouse/bdp.db/mqtest/day=20190101  ;
Found 1 items
drwxr-xr-x   - bdp hive          0 2020-06-30 18:39 /user/hive/warehouse/bdp.db/mqtest/day=20190101/data.har

```



同时存在 har 文件和普通文件时 select * 不受影响







## text

https://sukbeta.github.io/hadoop-lzo-gz-bz2/

text 命令可以直接查看对应压缩包，支持格式可以看上面的文档

```shell
hdfs -text har:///project/ccc.har/ccc.gz
```





## 刷新角色组（可能没什么用）

  su - hdfs -s /bin/bash -c "hdfs dfsadmin -refreshUserToGroupsMappings"



## fsck 丢失block

```shell

  su - hdfs -s /bin/bash -c "hdfs fsck /"

```



```shell
 su - hdfs -s /bin/bash -c "hdfs debug recoverLease -path /hbase/oldWALs/pv2-00000000000000004078.log  " 
```







