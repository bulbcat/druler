# [离线部署yum依赖](https://www.cnblogs.com/oaks/p/13614599.html)

利用本地源解决在无网环境部署应用需要解决的问题：

1. 应用需要哪些软件包？
2. 如何把应用依赖的软件包制作成一个精简的本地源?
3. 如何使用本地源?

第一个问题使用`yum-utils`解决，它带的`repotrack` 命令可以把应用所依赖的软件包全部下载到本地，安装：

```bash
yum install -y yum-utils  # repotrack 工具用来下载yum依赖
```

以要离线部署gcc为例来下载所有相关依赖：

```
mkdir -p /home/oaksharks/install/yumRepo/packages
repotrack gcc -p /home/oaksharks/install/yumRepo/packages
```

`repotrack` 下载的都是 rpm包，如果用`rpm`安装不容易解决依赖关系，可以给这些包生成索引作为一个本地的yum源，可以使用`createrepo`完成
先安装：

```bash
yum install -y createrepo # 使用createrepo 创建私有yum源
```

给rpm包创建索引：

```bash
createrepo /home/oaksharks/install/yumRepo/packages
```

索引会创建到`createrepo /home/oaksharks/install/yumRepo/packages/repo`目录。

配置使用本地源：
创建文件`/etc/yum.repos.d/CentOS-Local.repo` 内容为：

```
[Local]
name=Local Yum
baseurl=file:///home/oaksharks/install/yumRepo/packages
gpgcheck=0
enabled=1
```

软件重建缓存：

```
yum clean all
yum makecache
```

可以使用yum利用本地源安装软件了：

```
yum install -y gcc
```

## 常见问题

### 怎么新加软件包？

1. 使用repotrack 下载新的包
2. 删除 repo目录，然后重新创建索引

已经验证此种方式切断网卡后可以正常使用，可放心食用。



