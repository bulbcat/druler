

```sql
set hive.execution.engine=spark;


set hive.spark.client.future.timeout=200;

set hive.spark.client.connect.timeout=80000ms;


## 给的太少会报 spark client 初始化问题
## 给的太多会 一直起不来 (driver 就起不来)，或者起来后 stage 一直不动（executor起不来）
## 多租户情况下考虑对应队列的上限的限制 (指定用户队列资源太少也会起不来)
## 下面的配置一般人物可以跑，参考 机器 16c 32g,混部。 yarn 的 配置 nodemanager.memory=12g ，scheluer.max=8g 
set spark.driver.memory=4g;
set spark.executor.memory=8g;

```



https://blog.csdn.net/benpaodexiaowoniu/article/details/105898501







