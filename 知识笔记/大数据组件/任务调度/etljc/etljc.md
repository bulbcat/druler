# etljc

## 初始化库

建表 source etljc.sql 

初始化 sys_cfg

初始化 sys_router



## 软链接



```shell


## 连走整个目录
## sudo sh -c 'mkdip -p /data/bdp ;ln -s /data/bdp /bdp ;chown bdp:bdp /data/bdp ;chmod 777 /bdp ;'

## 连走 etljc


sudo sh -c 'mkdir -p /data/bdp/bin/etljc /data/bdp/log/etljc ;  
                ln -s /data/bdp/bin/etljc /bdp/bin/etljc  ;
                ln -s /data/bdp/log/etljc /bdp/log/etljc  ;
                chmod 777 /bdp ; 
                chown bdp:bdp -R /data/bdp /bdp/bin/etljc  /bdp/log/etljc'




```





## 介质传输

使用bdp用户，依赖先创建工作空间



```shell
scp -r root@10.182.212.81:/data/setup/etljc*/*  /bdp/bin/etljc/
chmod 755 -R /bdp/bin/etljc/
```





## 修改配置



加hosts 



etljcmysql xx.xx.xx.xx



```properties
10.182.212.81   hsnn01
10.182.212.82   hsnn02
10.182.212.83   hsdn01
10.182.212.84   hsdn02
10.182.212.85   hsdn03
10.182.212.86   hshv01
10.182.212.87   hshv02
10.182.212.88   hsmsg01
10.182.212.89   hsmsg02
10.182.212.90   hsmsg03
10.182.212.91   hssp01
10.182.212.92   hssp02
10.182.212.93   hssp03
10.182.212.94   hszk01
10.182.212.95   hszk02
10.182.212.96   hszk03
10.182.212.97   hsdx01 hivemeta
10.182.212.98   hsjb01
10.182.212.99   hsfp01
10.182.212.100  hsdb01 etljcmysql
```



