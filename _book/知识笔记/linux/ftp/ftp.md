# 	ftp



## 安装服务





```shell
#安装
yum install -y vsftpd

#设置开机启动
systemctl enable vsftpd.service

#启动
systemctl start vsftpd.service

#停止
systemctl stop vsftpd.service

#查看状态
systemctl status vsftpd.service
```









## ftp创建用户

```shell
useradd -d /ftp_data -g ftp -s /sbin/nologin ftp_test1
passwd ftp_test1 --设置密码

## echo "qwert" | passwd --stdin rusky
```



创建时对于共享数据 应归到同一个组里

## 

