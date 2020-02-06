# WSL配置手册



## 超级用户免密



```shell
sudo vi /etc/sudoers
```

```xml
找到这行 root ALL=(ALL) ALL,在他下面添加xxx ALL=(ALL) ALL (注：这里的xxx是你的用户名)
你可以根据实际需要在sudoers文件中按照下面四行格式中任意一条进行添加：
youuser            ALL=(ALL)                ALL
%youuser           ALL=(ALL)                ALL
youuser            ALL=(ALL)                NOPASSWD: ALL
%youuser           ALL=(ALL)                NOPASSWD: ALL
第一行：允许用户youuser执行sudo命令(需要输入密码)。
第二行：允许用户组youuser里面的用户执行sudo命令(需要输入密码)。
第三行：允许用户youuser执行sudo命令,并且在执行的时候不输入密码。
第四行：允许用户组youuser里面的用户执行sudo命令,并且在执行的时候不输入密码。
```



## 替换阿里源

替换

sudo vim /etc/apt/sources.list



使用vim打开，参考[这里](https://www.sunzhongwei.com/mip/modify-the-wsl-ubuntu-1804-default-source-for-ali-cloud-images)。在vim中输入如下的控制代码（需要先熟悉上古神器vim的操作）：

```
:%s/security.ubuntu/mirrors.aliyun/g
:%s/archive.ubuntu/mirrors.aliyun/g
```

sudo apt update
sudo apt upgrade



## 配置ssh server

自带的ssh server不好用，先卸载再安装即可。



```csharp
// 卸载
sudo apt-get remove openssh-server
// 安装
sudo apt-get install openssh-server
// 编辑配置文件
// vim /etc/ssh/sshd_config
    
    Port 36000  # 默认的是22，但是windows有自己的ssh服务用的也是22端口，修改一下
    UsePrivilegeSeparation no

// 重启ssh服务
sudo service ssh --full-restart
```

配置中有一项`PasswordAuthentication`， 改为`yes`可以使用密码登录，这里我们使用ssh秘钥对登录，所以使用默认`no`



这里需要设置 开机服务自启动。。。

亲测 不设置 登陆还是无法用



## 修改ubuntu 的dash shell 为 bash

sudo dpkg-reconfigure dash 

选no



## maven 替换阿里源



* 目前我用 unbuntu 里的 maven，这方面我也纠结需要讨论

 装直接 aptget 

```shell
sudo vim/etc/maven/settings.xml
```



找到 mirrors位置里添加

```xml
<mirror>
<id>alimaven</id>
<name>aliyun maven</name>
<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
<mirrorOf>central</mirrorOf>
</mirror>
<mirror>
<id>alimaven</id>
<mirrorOf>central</mirrorOf>
<name>aliyun maven</name>
<url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

指定maven 本地库目录与 win 上相同

```xml
<localRepository>这里填你所要放的文件夹的绝对路径,以resp结尾</localRepository>
```







## git ,maven

ubuntu直接有git。但是和本地的会有冲突，感觉直接用本地windows 的 bash shell 会简单的多.

- 强烈推荐使用 windows 的git bash 做git 相关的命令。毕竟后续开发用ide 还是会在 win上。

  

设置全局git 缓存，解决git clone 一半会终止的问题

```shell
git config --global http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```



如果git clone 还是慢

https://blog.csdn.net/cpongo3/article/details/93624283

没有具体试过。最后的

```shell
git clone --depth=1 https://github.com/xxx/xxx.git
cd xxx
git fetch --unshallow
```

这个看起来很有用的样子









## mysql

apt 直接装



```bash
echo $ss
mkdir -p 
```

