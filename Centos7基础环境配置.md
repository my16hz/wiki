# Centos 7基础环境配置
## 设置主机名
`hostnamectl set-hostname x.example.com`
退出链接后生效

## 绑定IP地址
使用`ifconfig -a`命令找到对应的网卡，如em1  
编辑配置文件`/etc/sysconfig/network-scripts/ifcfg-em1`，修改或添加对应项
```
ONBOOT=yes
BOOTPROTO=static
IPADDR=xxx.xxx.xxx.xxx
NETMASK=xxx.xxx.xxx.xxx
GATEWAY=xxx.xxx.xxx.xxx
```
重启网络`service network restart`

## 自动更新时间
使用crontab每天晚上0点自动更新系统时间
执行 crontab -e
```
0 0 * * * ntpdate cn.pool.ntp.org;hwclock -w;echo `date`" ntpdate" >> ~/ntp.log
```

## DNS配置
```
cat > /etc/resolv.conf <<EOF
nameserver 你的DNS服务器IP
EOF
```

# DNS 服务器配置
安装dnsmasq服务`yum install -y dnsmasq`  
编辑配置文件`/etc/dnsmasq.hosts`添加对应的主机  
重启服务生效 `service dnsmasq start`

