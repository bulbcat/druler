# Typora 配置



## 改了css 样式

*  搞了下 一级标题 不加序号

*  目录的字体颜色 保持同步

```po
C:\Users\miaoq\AppData\Roaming\Typora\themes
```



## 加了preference,显示代码行数

![1590990140871](.assets/1590990140871.png)







## md画图



```flow
st=>start: Start
st2=>start: start2
in1=>inputoutput: input1
in2=>inputoutput: input2
op=>operation: Your Opera
op2=>operation: xxxx
cond=>condition: Yes or No?
e=>end


st->in1->op->cond
st2->in2->op
cond(yes)->e
cond(no)->op






```



```mermaid
graph LR;
	master-->portal
　　client---core;
　　client---common;
　　core---common;
　　common---portal;
　　common---Biz;
　　Biz---ConfigService;
　　Biz---AdminService;
```

### 试着画etljc





```mermaid
graph TD

dcds文本--ftp-->ftp
subgraph FOP 

ftp--识别文件规则 异常-->lost
ftp--识别文件规则 正常-->recdir--解压-->ugdir--本地文件备份-->bakdir

end

subgraph etljc-master

subgraph cycle1-文本守护 

ugdir--datax上传 记录日志进mysql-->sys_file_stat
sys_file_cfg-.输入.->cycle1工作守护
sys_file_stat-.输入.->cycle1工作守护--满足条件 更新-->sys_file_stat
end

subgraph cycle2-表守护



cycle1工作守护--满足条件 插入-->sys_table_stat

sys_table_stat-.输入.->cycle2工作守护--条件满足判断 是-->分发调度--资源满足判断-->判断处理模式
cycle2工作守护--满足条件 更新-->sys_table_stat
sys_router-.输入.->分发调度
sys_table_cfg-.输入.->判断处理模式
判断处理模式-->时点表时段表
判断处理模式-->时点增量表



end


end


subgraph hdfs
ugdir--datax-->RAW
MID
ODS

end

subgraph SCH
subgraph 时点表时段表处理流程
%% RAW-.输入.->trim
时点表时段表--按资源饱和度分发-->trim--更新-->sys_table_stat
trim--写入-->ODS
end

subgraph 时点增量表处理流程
%% RAW-.输入.->trim*
时点增量表--按资源饱和度分发-->trim*--更新-->sys_table_stat
trim*--写入-->MID
trim*--master节点满足条件判定-->merge--写入-->ODS

end

end



```





## 图片设置相对路径



相对路径后，再分享模式会容易很多。直接扔对应的文件夹就可以用了。

git 上也可以直接引用。

目前设置在了 ./.assets

利用 . 使目录变为隐藏目录，typora文件树中不会显示，更方便的阅读条目。

如下图



![image-20200620144936365](.assets/image-20200620144936365.png)





git clean -f

![image-20200620155403318](.assets/image-20200620155403318.png)

