## Shadowsocks 客户端
### 安装
```
pip install --upgrade pip
pip install shadowsocks
```

### 配置
创建配置文件 `/etc/shadowsocks.json`,添加对应内容:
```
{
  "server":"x.x.x.x",             #你的 ss 服务器 ip
  "server_port":0,                #你的 ss 服务器端口
  "local_address": "127.0.0.1",   #本地ip
  "local_port":4321,              #本地端口
  "password":"password",          #连接 ss 密码
  "timeout":300,                  #等待超时
  "method":"aes-256-cfb",         #加密方式
  "workers": 1                    #工作线程数
}
```

### 启动
创建自启动文件 `/etc/systemd/system/shadowsocks.service`
```
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```

### 验证
运行 `curl --socks5 127.0.0.1:4321 http://httpbin.org/ip`，如果返回你的 ss 服务器 ip 则测试成功：
```
{
  "origin": "x.x.x.x"       #你的 ss 服务器 ip
}
```

## Docker 配置socks5
创建配置文件写入shadowsocks代理
```
cat > /lib/systemd/system/docker.service.d/socks5-proxy.conf <<EOF
[Service]
Environment="ALL_PROXY=socks5://127.0.0.1:4321"
EOF
```
重新加载配置
```
systemctl daemon-reload
systemctl restart docker
```
测试 pull gcr.io 镜像
```
docker pull gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.9.3
```