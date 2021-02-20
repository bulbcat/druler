# Kerberos



* 部署 kerberos 参考文档

https://blog.csdn.net/ytp552200ytp/article/details/109643832





* 应用程序 使用 kerberos

https://blog.csdn.net/qq_42790479/article/details/104040870





看了一个装 cdh 的

https://mp.weixin.qq.com/s?__biz=MzI4OTY3MTUyNg==&mid=2247495280&idx=1&sn=e1d09b47cc18a2d6e862d21501b44475&chksm=ec293e79db5eb76f6267a8ac4309b920e291f2f5c792eb02a64feec6af72892d625fdf4c1085&scene=21#wechat_redirect



对应装 kerberos 的

https://blog.csdn.net/yangbosos/article/details/88718135







**KDC服务安装及配置**

本文档中将KDC服务安装在Cloudera Manager Server所在服务器上（KDC服务可根据自己需要安装在其他服务器）

1.在Cloudera Manager服务器上安装KDC服务

```
[root@ip-172-31-6-83 ~]# yum -y install krb5-server krb5-libs krb5-auth-dialog krb5-workstation
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/eycocl9gcx.jpeg?imageView2/2/w/1620)

2.修改/etc/krb5.conf配置

```
[root@ip-172-31-6-83 ~]# vim /etc/krb5.conf
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = FAYSON.COM
 #default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 FAYSON.COM = {
  kdc = ip-172-31-6-83.ap-southeast-1.compute.internal
  admin_server = ip-172-31-6-83.ap-southeast-1.compute.internal
 }

[domain_realm]
 .ap-southeast-1.compute.internal = FAYSON.COM
 ap-southeast-1.compute.internal = FAYSON.COM
```

标红部分为需要修改的信息。

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/cvb28pw3ie.png?imageView2/2/w/1620)

3.修改/var/kerberos/krb5kdc/kadm5.acl配置

```
[root@ip-172-31-6-83 ~]# vim /var/kerberos/krb5kdc/kadm5.acl
*/admin@FAYSON.COM      *
```

4.修改/var/kerberos/krb5kdc/kdc.conf配置

```
[root@ip-172-31-6-83 ~]# vim /var/kerberos/krb5kdc/kdc.conf
[root@ip-172-31-6-83 ~]# cat /var/kerberos/krb5kdc/kdc.conf
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 FAYSON.COM = {
  #master_key_type = aes256-cts
  max_renewable_life= 7d 0h 0m 0s
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

标红部分为需要修改的配置。

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/a69q7t9i7o.png?imageView2/2/w/1620)

5.创建Kerberos数据库

```
[root@ip-172-31-6-83 ~]# kdb5_util create –r FAYSON.COM -s
Loading random data
Initializing database '/var/kerberos/krb5kdc/principal' for realm 'FAYSON.COM',
master key name 'K/M@FAYSON.COM'
You will be prompted for the database Master Password.
It is important that you NOT FORGET this password.
Enter KDC database master key: 
Re-enter KDC database master key to verify:
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/kw0m02y5so.png?imageView2/2/w/1620)

此处需要输入Kerberos数据库的密码。

6.创建Kerberos的管理账号

```
[root@ip-172-31-6-83 ~]# kadmin.local
Authenticating as principal root/admin@FAYSON.COM with password.
kadmin.local:  addprinc admin/admin@FAYSON.COM
WARNING: no policy specified for admin/admin@FAYSON.COM; defaulting to no policy
Enter password for principal "admin/admin@FAYSON.COM": 
Re-enter password for principal "admin/admin@FAYSON.COM": 
Principal "admin/admin@FAYSON.COM" created.
kadmin.local:  exit
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/s7lhy1a6l3.png?imageView2/2/w/1620)

标红部分为Kerberos管理员账号，需要输入管理员密码。

7.将Kerberos服务添加到自启动服务，并启动krb5kdc和kadmin服务

```
[root@ip-172-31-6-83 ~]# systemctl enable krb5kdc
Created symlink from /etc/systemd/system/multi-user.target.wants/krb5kdc.service to /usr/lib/systemd/system/krb5kdc.service.
[root@ip-172-31-6-83 ~]# systemctl enable kadmin
Created symlink from /etc/systemd/system/multi-user.target.wants/kadmin.service to /usr/lib/systemd/system/kadmin.service.
[root@ip-172-31-6-83 ~]# systemctl start krb5kdc
[root@ip-172-31-6-83 ~]# systemctl start kadmin
```

8.测试Kerberos的管理员账号

```
[root@ip-172-31-6-83 ~]# kinit admin/admin@FAYSON.COM
Password for admin/admin@FAYSON.COM: 
[root@ip-172-31-6-83 ~]# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: admin/admin@FAYSON.COM

Valid starting       Expires              Service principal
12/27/2018 22:05:56  12/28/2018 22:05:56  krbtgt/FAYSON.COM@FAYSON.COM
        renew until 01/03/2019 22:05:56
```

 

9.为集群安装所有Kerberos客户端，包括Cloudera Manager

使用批处理脚本为集群所有节点安装Kerberos客户端

```
[root@ip-172-31-6-83 shell]# sh ssh_do_all.sh node.list 'yum -y install krb5-libs krb5-workstation'
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/2vfcxqnbhn.jpeg?imageView2/2/w/1620)

10.在Cloudera Manager Server服务器上安装额外的包

```
[root@ip-172-31-6-83 shell]# yum -y install openldap-clients
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/ol3dhcworh.jpeg?imageView2/2/w/1620)

11.将KDC Server上的krb5.conf文件拷贝到所有Kerberos客户端

使用批处理脚本将Kerberos服务端的krb5.conf配置文件拷贝至集群所有节点的/etc目录下：

```
[root@ip-172-31-6-83 shell]# sh bk_cp.sh node.list /etc/krb5.conf /etc/
```

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/ium8hbtjps.png?imageView2/2/w/1620)

3