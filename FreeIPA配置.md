# FreeIPA Server 安装

## 0. 环境说明

版本如下
```
$ cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

IP 如下:
```
172.16.100.106 bda106.example.com         # FreeIPA Server
172.16.100.107 bda107.example.com         # DNS Server
172.16.100.105 bda105.example.com         # FreeIPA Client
```
## 1. 准备IPA服务器

### 1.1 更新系统
```
yum –y update
```

### 1.2 配置防火墙

打开所需端口
```
firewall-cmd --permanent --add- port={80/tcp,443/tcp,389/tcp,636/tcp,88/tcp,464/tcp,53/tcp,88/udp,464/udp,53/udp,123/udp}
```

重新加载防火墙
```
firewall-cmd --reload
```

查看一下防火墙开放的服务
```
# firewall-cmd --list-all
public (active)
target: default
icmp-block-inversion: no
interfaces: enp0s3
sources:
services: dhcpv6-client http https kerberos kpasswd ldap ldaps ntp ssh
ports:
protocols:
masquerade: no
forward-ports:
sourceports:
icmp-blocks:
rich rules:
```

### 1.3 设置DNS

设置机器名，必须保障机器名正确,与 FQDN 一致。
1. 测试 DNS 服务,安装 bind-utils工具
```
# yum install -y bind-utils
```

2. 正向域测试
```
# dig +short bda106.example.com A
```
这应该返回 172.16.100.106

3. 反向域测试
```
# dig +short -x 172.16.100.106
```
这应该返回结果bda106.example.com

### 1.4 配置随机数生成工具

安装 FreeIPA 的加密操作需要很多随机数，我们使用 rngd。

安装
```
# yum install -y rng-tools
```
启动程序
```
# systemctl start rngd
```
设置开机启动
```
# systemctl enable rngd
```
最后，查看一下服务状态
```
# systemctl status rngd
```

**注意：如果rngd服务启动失败，查询：**
```
#rngd -v
    Unable toopenfile: /dev/tpm0
    can't open any entropy source
    Maybe RNG device modules  are not loaded
```
这样的话，rngd.service 启动会出错，报错信息：loaded failed。

**解决方法如下:**

a. 把rngd.service复制到自定义目录/etc/systemd/system
```
# cp /usr/lib/systemd/system/rngd.service /etc/systemd/system
```
b. 编辑vi /etc/systemd/system/rngd.service，改动ExecStart为如下:
```
ExecStart=/sbin/rngd -f -r /dev/urandom
```
c. 重新载入systemd.
```
 # systemctl daemon-reload
```
d. 重新启动rngd服务。
```
# systemctl restart rngd
```
查看状态启动成功 **。**

## 2. 安装 FreeIPA 服务端
安装ipa-server 这个是 FreeIPA 服务包
```
#yum install –y ipa-server
```
接下来输入 FreeIPA 服务端安装命令，这将执行一系列脚本来配置并安装。
```
# ipa-server-install
```
注意： 默认安装过程会安装 DNS 服务到本机，本例已经配置好 DNS 服务，所以不需要安装。

安装过程：
```
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.
This includes:
  \* Configure a stand-alone CA (dogtag) for certificate management
  \* Configure the Network Time Daemon (ntpd)
  \* Create and configure an instance of Directory Server
  \* Create and configure a Kerberos Key Distribution Center (KDC)
  \* Configure Apache (httpd)
To accept the default shown in brackets, press the Enter key.
Do you want to configure integrated DNS (BIND)? [no]: no #输入 no 不安装 dns 服务
Enter the fully qualified domain name of the computer
on which you&#39;re setting up server software. Using the form
&lt;hostname&gt;.&lt;domainname&gt;
Example: master.example.com.

Server host name [bda106.example.com]:  #回车
The domain name has been determined based on the host name.
Please confirm the domain name [example.com]: #回车
The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.
Please provide a realm name [example.com]: #回车
Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.
Directory Manager password: \*\*\*\*\*\* #输入密码
Password (confirm): \*\*\*\*\*\* #输入密码12345678
The IPA server requires an administrative user, named &#39;admin&#39;.
This user is a regular system account used for IPA server administration.
IPA admin password: \*\*\*\*\*\* #输入密码12345678
Password (confirm): \*\*\*\*\*\* #输入密码12345678
The IPA Master Server will be configured with:
Hostname:       bda106.example.com
IP address(es): 172.16.100.106
Domain name:    example.com
Realm name:     example.com
Continue to configure the system with these values? [no]: yes  #输入 yes 确认配置
The following operations may take some minutes to complete.
Please wait until the prompt is returned.
_..._
_..._
_..._
==============================================================================
Setup complete
Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                 \* 80, 443: HTTP/HTTPS
                 \* 389, 636: LDAP/LDAPS
                 \* 88, 464: kerberos
                UDP Ports:
                 \* 88, 464: kerberos
                 \* 123: ntp
        2. You can now obtain a kerberos ticket using the command: &#39;kinit admin&#39;
          This ticket will allow you to use the IPA tools (e.g., ipa user-add)
          and the web user interface.
Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
```
现在 FreeIPA 服务端已经安装完了，端口也开放了，我们需要测试一下

### 3. 测试 FreeIPA 服务端**

#### 测试 Kerberos
我们需要 Kerberos 的 token
```
# kinit admin
Password for admin@EXAMPLE.COM:
```
输入 admin 的密码12345678，如果成功，表明 Kerberos 安装正确。
### 测试 IPA 服务
得到 token之后，输入如下命令
```
# ipa user-find admin
--------------
1 user matched
--------------
  User login: admin
  Last name: Administrator
  Home directory: /home/admin
  Login shell: /bin/bash
  Principal alias: admin@EXAMPLE.COM
  UID: 626400000
  GID: 626400000
  Account disabled: False
----------------------------
Number of entries returned 1
----------------------------
```
如果正确，会返回一条记录。

### Web页面登录
访问  [http://bda106.example.com](http://bda106.example.com)

## 4. IPA Client

### 4.1 安装
```
# yum install –y freeipa-client
```
### 4.2 配置client
```
# ipa-client-install --mkhomedir
DNS discovery failed to determine your DNS domain
Provide the domain name of your IPA server (ex: example.com): example.com #输入
Provide your IPA server name (ex: ipa.example.com): bda106.example.com #输入
The failure to use DNS to find your IPA server indicates that your resolv.conf file is not properly configured.
Autodiscovery of servers for failover cannot work with this configuration.
If you proceed with the installation, services will be configured to always access the discovered server for all operations and will not fail over to other servers in case of failure.
Proceed with fixed values and no DNS discovery? [no]:yes #输入
Client hostname: bda105.example.com
Realm: example.com
DNS Domain: example.com
IPA Server: bda106.example.com
BaseDN: dc=cetc32,dc=com
Continue to configure the system with these values? [no]: yes #输入
Synchronizing time with KDC...
Attempting to sync time using ntpd.  Will timeout after 15 seconds
User authorized to enroll computers: admin #输入
Password for admin@EXAMPLE.COM: \*\*\*\* #输入密码12345678
...
...
Configuring example.com as NIS domain.
Client configuration complete.
```
至此 FreeIPA 客户端安装完成，这个服务器受 FreeIPA 服务端管理了。访问 [http://bda106.example.com](http://bda106.example.com)，可查看被管理的FreeIPA客户端。

参考： [https://www.howtoing.com/how-to-set-up-centralized-linux-authentication-with-freeipa-on-centos-7](https://www.howtoing.com/how-to-set-up-centralized-linux-authentication-with-freeipa-on-centos-7)
