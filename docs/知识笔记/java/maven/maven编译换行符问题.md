解决方案：

mavem添加插件antrun，配置fixcrlf



```xml
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <id>ant-test</id>
                        <phase>package</phase>
                        <configuration>
                            <tasks>
                                <fixcrlf srcdir="${basedir}/src/main/bin/" eol="unix"/>
                            </tasks>
                        </configuration>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```



参考：
https://stackoverflow.com/questions/2162275/convert-files-to-unix-format-using-maven

http://www.iteye.com/problems/69937





跳过测试
```shell
-DskipTests      # 不执行测试用例，但编译测试用例类生成相应的 class 文件至 target/test-classes 下
-Dmaven.test.skip=true    # 不执行测试用例，也不编译测试用例类
```

##  例如
```shell
mvn clean install -DskipTests 
mvn clean install -Dmaven.test.skip=true
```
从指定模块编译

-rf :moduleName  # 从 moduleName 模块开始编译
## 在编译到一半后，报错退出。使用这个参数指定从上次编译失败的模块处开始编译。不用从头开始编译，可以避免浪费不必要的时间

## 例如编译flink 时，从 flink-hadoop-fs 模块开始编译
mvn -rf :flink-hadoop-fs clean install
并行构建

使用多个线程，并行构建相互之间没有依赖关系的模块
# 4 表示用4个线程构建
$ mvn -T 4 clean install

# 1C 表示为机器的每个 core分配一个线程，如何4核4线程的机器，就是 1*4 个线程
$ mvn -T 1C clean install
跳过失败的模块，编译到最后再报错

mvn clean install --fail-at-end
————————————————
版权声明：本文为CSDN博主「好笨的菜鸟」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_38976805/article/details/103066020







ranger-hive-plugin

