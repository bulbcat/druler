# 启动

```shell
service mysql start 
systemctl mysql start 
systemctl mysql enabled 



nohup hive --service metastore &

nohup  hive --service hiveserver2 &


hadoop
start-all.sh
```





# 内部表与外部表的区别





hive默认存储内部表存储路径，不指定路径存储的话，路径为 hdfs 路径

```shell
/user/hive/warehouse/bdp.db/mqtest ;
```



drop 会删除内部表 元数据 和 数据

外部表很多 命令不支持，比如 truncate 





# 数据导入



txt 数据样式

```shell
vim ccc
```

1,2000000000000000000000000
2,2000000000000000000000000



hive默认分隔符

数据样式例子

AAAA\001BBBB\001CCCC1\002CCCC2\002CCCC3



实际shell 造数 ，

```shell
echo AAAA$'\001'BBBB$'\001'CCCC1$'\002'CCCC2$'\002'CCCC3
```





```shell

gzip  ccc 
## hdfs dfs -put -f ccc.gz hdfs://bocfs/project/mqtest/
hdfs dfs -put -f ccc.gz //mqtest/
```



## 建表



### 内部表

```sql
create table mqtest 
(col1 string ,col2 string )
location '/user/bdp/mqtest'
```



### 外部表



```sql
create external table mqtest_par
(col1 decimal(13,8) ,col2 string 

) 

partitioned by (day string)
location '//mqtest 
stored as orc 

delimeter by ','

;

explain insert overwrite  mqtest_par partition (day=20190101) values (1.1,"ssss");


explain insert into mq values ('1') ;
insert overwrite mqtest_par partition (day) values (1.1,"ssss",20190101),(1.1,"ssss",20190120),();



```





### 压缩格式

ZLIB

gz

bzip2

lzo



| Storage Format               | Description                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| STORED AS TEXTFILE           | Stored as plain text files. TEXTFILE is the default file format, unless the configuration parameter [hive.default.fileformat](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat) has a different setting.Use the DELIMITED clause to read delimited files.Enable escaping for the delimiter characters by using the 'ESCAPED BY' clause (such as ESCAPED BY '\') Escaping is needed if you want to work with data that can contain these delimiter characters.  A custom NULL format can also be specified using the 'NULL DEFINED AS' clause (default is '\N'). |
| STORED AS SEQUENCEFILE       | Stored as compressed Sequence File.                          |
| STORED AS ORC                | Stored as [ORC file format](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax). Supports ACID Transactions & Cost-based Optimizer (CBO). Stores column-level metadata. |
| STORED AS PARQUET            | Stored as Parquet format for the [Parquet](https://cwiki.apache.org/confluence/display/Hive/Parquet) columnar storage format in [Hive 0.13.0 and later](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.13andlater); Use ROW FORMAT SERDE ... STORED AS INPUTFORMAT ... OUTPUTFORMAT syntax ... in [Hive 0.10, 0.11, or 0.12](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.10-0.12). |
| STORED AS AVRO               | Stored as Avro format in [Hive 0.14.0 and later](https://issues.apache.org/jira/browse/HIVE-6806) (see [Avro SerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)). |
| STORED AS RCFILE             | Stored as [Record Columnar File](https://en.wikipedia.org/wiki/RCFile) format. |
| STORED AS JSONFILE           | Stored as Json file format in Hive 4.0.0 and later.          |
| STORED BY                    | Stored by a non-native table format. To create or link to a non-native table, for example a table backed by [HBase](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration) or [Druid](https://cwiki.apache.org/confluence/display/Hive/Druid+Integration) or [Accumulo](https://cwiki.apache.org/confluence/display/Hive/AccumuloIntegration). See [StorageHandlers](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers) for more information on this option. |
| INPUTFORMAT and OUTPUTFORMAT | in the file_format to specify the name of a corresponding InputFormat and OutputFormat class as a string literal.  For example, 'org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat'.  For LZO compression, the values to use are 'INPUTFORMAT "com.hadoop.mapred.DeprecatedLzoTextInputFormat" OUTPUTFORMAT "[org.apache.hadoop.hive.ql.io](http://org.apache.hadoop.hive.ql.io/).HiveIgnoreKeyTextOutputFormat"'  (see [LZO Compression](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO)). |











```sql
CREATE TABLE `mq`(
  `col1` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://localhost:9000/user/hive/warehouse/mq'
  'hdfs://localhost:9000/user/hive/warehouse/kk.db/mq'
  
  
  
  
TBLPROPERTIES (
  'transient_lastDdlTime'='1582077774')



```

