1、修改docker.service配置文件

修改/usr/lib/systemd/system/docker.service，将

```bash
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

改为

```bash
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

2、重启Docker

```bahs
# 1、重新加载配置信息
systemctl daemon-reload

# 2、重启Docker
systemctl  restart docker

# 3、测试监听端口
ss -lt  |grep 2375

#打印结果如下：
[root@~ system]# ss -lt  |grep 2375
LISTEN     0      128       :::2375                    :::*  
```

