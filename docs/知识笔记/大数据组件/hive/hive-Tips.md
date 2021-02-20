# hive-Tips

[TOC]

## hive 

    网上说的很多知识是针对老版本的，包括很多看起来很新的博客，视频教程中提到的很多点，在新的版本中可能已经修复了这些不易用的点
    2.0+ 的hive在很多已经优化了之前的问题。
    常见的：
### 已经不是问题的问题

* hive 两个表关联，小表放在前面。（已经优化了，放前放后都一样）
* 小表使用 hint 强制走 mapjoin。(默认对于什么是小表有个配置，在这个配置下的表会自动走mapjoin)
* 数据倾斜走  skewindata。 (这个hint 走起来大部分场景都会显著拖慢整个运行过程，因为它会首先进行全数据打散，这个通常无比的慢，然后处理，然后再聚合。通常比不加慢很多，如果有数据倾斜，大部分场景都可以代码调整逻辑来处理，使 skewindata 用建议测试有明显效果后使用，不要随便用，更不能每个都加)
* 搜索单条记录会扫描全表，导致很慢。（orc等列式存储出现后，orc本身的结构存在一定的类似索引的头文件信息，会根据数据值范围进行范围搜索，范围搜索单条记录，类比以前的思路比生成大临时表，然后处理这种思路会有比较新的变化）





### 压缩存储  直读压缩文件

https://www.jianshu.com/p/afd978026661

* 所有常见的压缩格式 包括 gz lzo bz2 snappy 对应hive都可以直接读取
* zlib 待研究



传三个gz文本（逗号为英文，显示怪怪的要淡定）

```sql
hive> dfs -ls /project/BDP/gztest ;
Found 3 items
-rw-rw-r--   3 bdp BDP-ALL         80 2036-06-30 19:22 /project/BDP/gztest/txt1.txt.gz
-rw-rw-r--   3 bdp BDP-ALL         34 2036-06-30 19:22 /project/BDP/gztest/txt2.txt.gz
-rw-rw-r--   3 bdp BDP-ALL         33 2036-06-30 19:22 /project/BDP/gztest/txt3.txt.gz


hive> dfs -text  /project/BDP/gztest/* ;
OpenSSH_6.6.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013
ssssOpenSSH_6.6.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013
2222
333
```



建表语句 直接建 txt 

```sql
  create table mqtest_gz3 (col1 string ,col2 string ) row format delimited fields terminated by ','   location '/project/BDP/gztest/' ;
```



直接可以查

```sql
hive> select * from mqtest_gz3 ;
hook status=true,operation=QUERY
OK
OpenSSH_6.6.1p1	 OpenSSL 1.0.1e-fips 11 Feb 2013
ssssOpenSSH_6.6.1p1	 OpenSSL 1.0.1e-fips 11 Feb 2013
2222	NULL
333	NULL
```





### 多字节分隔符

https://blog.csdn.net/u014307117/article/details/82428598

官方例子

```sql

CREATE TABLE test (
id string,
hivearray array<binary>,
hivemap map<string,int>) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.MultiDelimitSerDe'                  
WITH SERDEPROPERTIES ("field.delim"="[,]","collection.delim"=":","mapkey.delim"="@");


```



```sql
CREATE TABLE mqtest_mul (

col1 string ,
col2 string 
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.MultiDelimitSerDe' 
WITH SERDEPROPERTIES ("field.delim"="OpenSSL","collection.delim"=":","mapkey.delim"="@");

```















### 小文件合并





#### 新增合并步骤（Flie Merge）

```sql
set hive.execution.engine=tez;

set hive.merge.tezfiles=true;    -- tez 引擎开启小文件合并
set hive.merge.mapredfiles=true; -- mr引擎开启小文件合并
set hive.merge.smallfiles.avgsize=128000000; 
set hive.merge.size.per.task=128000000; 

set hive.support.quoted.identifiers=none ; -- 开启正则表达式，下文中会用到 `(day)?+.+`
```



如果没有上述配置设置，生成的文件数与reduce 个数有关，默认在资源充足的情况下，会尽可能的起动很多个reduce,当前环境默认的单个任务的 reduce 上限为1024 。

所以在大部分情况下，每个hql任务的将生成1024个结果文件。

使用上述配置后，当结果文件小于128M 时回进行小文件合并。



```shell
hive> insert overwrite table inner_ods_bocs_inve  partition (day='20350825') select  `(day)?+.+`  from inner_ods_bocs_inve where day='20350825'
    > ;
Query ID = bdp_20360617150220_eab386e4-7eca-4113-90f7-7fb55e466463
Total jobs = 3
Launching Job 1 out of 3
Status: Running (Executing on YARN cluster with App id application_2094365163684_198629)

----------------------------------------------------------------------------------------------
        VERTICES      MODE        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED  
----------------------------------------------------------------------------------------------
Map 1 .......... container     SUCCEEDED      2          2        0        0       0       0  
VERTICES: 01/01  [==========================>>] 100%  ELAPSED TIME: 9.87 s     
----------------------------------------------------------------------------------------------
Stage-5 is filtered out by condition resolver.
Stage-4 is selected by condition resolver.
Stage-6 is filtered out by condition resolver.
Launching Job 3 out of 3
----------------------------------------------------------------------------------------------
        VERTICES      MODE        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED  
----------------------------------------------------------------------------------------------
File Merge ..... container     SUCCEEDED      1          1        0        0       0       0  
VERTICES: 01/01  [==========================>>] 100%  ELAPSED TIME: 0.01 s     
----------------------------------------------------------------------------------------------
Loading data to table bdp.inner_ods_bocs_inve partition (day=20350825)
Loading data to table bdp.inner_ods_bocs_inve partition (day=20350825)
Moved: 'hdfs://hdfsCluster/project/BDP/data/inner/ODS/01/INVE/20350825/000006_0' to trash at: hdfs://hdfsCluster/user/bdp/.Trash/Current
.......此处有省略........
Moved: 'hdfs://hdfsCluster/project/BDP/data/inner/ODS/01/INVE/20350825/000044_0' to trash at: hdfs://hdfsCluster/user/bdp/.Trash/Current
hook status=true,operation=QUERY
OK
Time taken: 12.718 seconds
hive> dfs -ls /project/BDP/data/inner/ODS/01/INVE/20350825/ ;
Found 1 items
-rw-rw-r--   3 bdp hdfs     502566 2036-06-17 15:02 /project/BDP/data/inner/ODS/01/INVE/20350825/000000_0
hive> 
```



可以看到会新增一个File Merge 的stage

众多小文件会被合并成1个，具体文件个数由表实际文件大小和 上述配置参数决定。





除此之外还有一种官方的聚合手段

```sql
alter table inner_ods_bocs_inve partition (day=20350823) concatenate
```

可以使用上述方法对历史数据进行小文件合并，实测1024 第一次运行后生成 18个结果文件

多次执行后会越来越少，具体使用影响参数未知。



目前看 

```sql
set mapreduce.input.fileinputformat.split.minsize=128000000 ;
```

这个参数时影响这个的，但是是有bug的，2018年提出目前仍然未解决。所以不建议使用 concatenate 做相关的精准控制。参考 

https://issues.apache.org/jira/browse/HIVE-19090





#### hive 直接做归档



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

har结果无法 insert overwrite 







### 分区表





#### 查看分区表命令

```sql
desc extended tb partition(dt=20170612)
```





#### 分区表分区异常查询

```sql

MSCK REPAIR TABLE table_name;

```





#### 常用知识

    分区表每个分区相当于一个独立数据
    对于只会使用基本sql的同事，需要充分掌握 where on 两个关键字在sql解析中的方式
    对于常见的数据倾斜的场景请一定避免

* 对于分区表理解不是很充分的，建议使用粗暴的模式编写，使用子查询。

```sql
  select * from aaa
  left join 
    ( select * from bbb where bbb.day=20190101 ) bbb 
    on aaa.col1=bbb.col1 
    where aaa.day='20190101'
  ----------------  
    select * from aaa
  left join bbb

    on aaa.col1=bbb.col1 
    where aaa.day='20190101'
  and bbb.day=20190101
  
  
  ------------------
      select * from aaa
  left join bbb

    on aaa.col1=bbb.col1 
    and   bbb.day=20190101
    where aaa.day='20190101'
 
  
```

  上述sql等同于

```sql
select * from aaa
left join 
   bbb  
  on aaa.col1=bbb.col1
  and bbb.day='20190101'
  where aaa.day='20190101'
```

上面的写法是正确的，常见的错误写法

```sql
select * from aaa
left join 
   bbb  
  on aaa.col1=bbb.col1
  
  where aaa.day='20190101'
  and bbb.day='20190101'
```



这个就是没有分清 where 和 on 的基本逻辑，因为分区表是独立的元数据信息，目前hive2.0 解析器没有对这个逻辑进行自动修正。需要在写法上明确数据情况，进行规避。



    PS: 
    
    where 里写关联先关联，用关联结果再过滤
    
    on里写关联，选出能关联的进行关联  
    
    上述情况也可以视作处理数据量太大，数据倾斜。





### 数据倾斜

#### 拒绝使用 count(distinct)

distinct 结果将会输出一个reduce,对于大量distinct 结果时，比如统计有多少种人名。 对于10亿级的人，count(distinct name) ,最终上千万种name 将会被一个map读取处理，发生数据倾斜。 单一map 读取千万条人名数据，引发倾斜。



简单避免可以写 tmp表，输出到表里分两个sql 来做。或拆分对应语句使用 group by去做。



#### 尽可能不用full join 

绝大部分的 full join 都可以使用 union group by 替换，

full join 极易发生数据倾斜。需理解运行原理，并详细观察执行计划后使用。





#### 多表关联的数据倾斜

* 对于有一种最常见的数据倾斜的场景请一定避免

```sql
select * from aaa 
left join bbb
on aaa.col1=bbb.col1
left join ccc
on bbb.col2=ccc.col2




```

上述是一个经典的数据倾斜。而且在传统数据库中并没有任何的问题。但在hive中基本一定会出现数据倾斜

这里弄不懂的话讲下基本原理

常见的面试中的什么是数据倾斜回答无外乎，某某数值太多，空值过高。

原理上默认的join操作时类似于hash 后打散，数据倾斜基本是由于关联键hash分布不均



上述经典场景中 通过 查看解析计划

```sql
explain xxxx
```



执行计划 是先处理 left join 

假设 A 1000w 行，B 100w行，都能关联上

A * B 后 B.col2 字段只有100w行有数

使用B.col2 字段继续关联 C表900w 空在做hash 关联，



所以常见的场景下这种写法，一定会数据倾斜。

根据逻辑改成如下，用以规避相关倾斜问题，具体问题需要根据数据实际情况和执行计划情况进行分析

```sql
select * from aaa 
left join (select * from bbb
left join ccc
on bbb.col2=ccc.col2)  tmp
on aaa.col1=tmp.col1
```





#### map端倾斜



    常见的map端倾斜只要由于计算引擎是按照container进行计算资源划分的，有默认值
    默认值通常为128M
    对于只有两列三列这种列数比较少的数据，如果是txt这种格式引入的，将会形成单个块处理超级多的条数的数据
    对于这种数据如果处理sum(a+b)+sum(c+d),类似这种出现了500次-1000次时将会出现明显的map端倾斜


上述场景的经典场景是 类似hbase的标签数据，不同标签只有两列，客户号，是否、

如果要把他拉平成一个大宽表。

处理时如果使用 case when tag=sex , isvalid=1 then 1 else 0 , 这种逻辑500个标签重复500次，1000个标签重复1000次的话，将会直接引起map端倾斜，表现为map 长时间无法结束。



* 目前bdp采用的是比较蠢的解决方案，属于短时内对问题的应急处理，将对应1000个标签每50个拆分成1组，对这50个标签进行 case when 这种处理逻辑，每次降低轮扫数据后的处理逻辑复杂度。然后将200组结果进行join,200组的case when 在hive解析时将会进行分开处理，充分利用计算资源。 处理优化 5-6小时->40分钟-1小时
* 感觉比较好的方式是转成orc处理，待验证
* 通过配置拉大mapper资源这种方式，目前在hive中由于session级别安全限制，只有客户端模式可以提交，hiveserver模式无法修改默认影响mapper数量相关配置



### 其他

* hive的元数据库是一个宝库。如有能力请尽可能多的了解。hive相关内容推荐lxw 的博客，很靠谱。

* datax 3.0 可以支持直接传orc落表，后续bdp 可能会尝试使用替换主干代码，目前仅少量下发任务使用。datax 3.0 hdfsreader 读取orc有bug,一定确认相关版本bug是否修复。

  https://www.jianshu.com/p/12c635abdd30 

* hive的字段长度定义无法按照 字符定义，使mysql 下发 varchar(10) 可能存10个汉字。对于utf-8 的hive需要定义成为 varchar(30) 。BDP目前对上游数据长度统一扩为3倍保证数据不丢失，租户使用时需知道相关内容。

* hive中 txt格式下 varchar 性能明显由于 string。orc格式待验证，个人认为区别不大。



* 目前对于yarn中的队列资源是完全隔离的，类似与烟囱，并没有超分的机制。租户内对于任务资源使用分为 cpu紧张和内存紧张。运行时可以观察yarn 进行相关配置项调整。

  

### 建表格式

```sql


row format delimited fields terminated by '\-128' stored as textfile;
```













