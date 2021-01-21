# Docker安装

## CentOS版安装

官方文档地址：https://docs.docker.com/engine/install/centos/

1、卸载旧版本

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2、安装`yum-utils`软件包（提供`yum-config-manager` 实用程序）

```bash
sudo yum install -y yum-utils
```

3、设置**稳定的**存储库

```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

4、Docker引擎安装

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

如果提示您接受GPG密钥，请验证指纹是否匹配 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果是，则接受它。

5、启动docker

```bash
sudo systemctl start docker
```

## 安装完成 测试：

通过运行`hello-world` 映像来验证是否正确安装了Docker Engine 。

```bash
sudo docker run hello-world
```

修改docker仓库地址