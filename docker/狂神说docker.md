> Docker学习

- Docker概述
- Docker安装
- Docker命令
  - 镜像命令
  - 容器命令
  - 操作命令
  - ...
- Docker镜像！
- 容器数据卷！
- DockerFile
- Docker网络原理
- IDEA整合Docker
- Docker Compose
- Docker Swarm
- CI/CD jenkins

# Docker概述

## Docker为什么会出现？

一款产品：开发---上线  两套环境！应用环境，应用配置！

环境切换，版本更新导致服务不可用。

环境配置是十分麻烦的，每一个机器都要部署环境（集群Redis、ES、Hadoop）

发布一个项目（jar +(Redis MySQL jdk ES)),项目能不能都带上环境安装打包！

之前在服务器配置一个应用的环境Redis MySQL jdk ES Hadoop，配置超麻烦，不可跨平台。

学会Docker:开发打包部署上线，一套流程做完！



java --- jar(环境) ---打包项目带上环境（镜像） ---（Docker仓库：商店） ---下载我们发布的镜像 ---直接运行



Docker的思想来自于集装箱！

![image-20200928111953873](狂神说docker.assets/image-20200928111953873.png)

JRE ---多歌应用（端口冲突） ---原来都是交叉的！

隔离：Docker核心思想！打包装箱！每个享资是互相隔离的。

Docker通过隔离机制，可以将服务器利用到极致。



本质：所有技术都是因为出现了一些问题，我们需要去解决，才去学习。



## Docker的历史

Docker为什么这么火？十分的轻巧！

在容器技术出来之前，我们都是用的虚拟机技术！

虚拟机：在windows中装一个Vmware，通过这个软件我们可以虚拟出一台或者多台电脑！笨重！

虚拟机也是属于虚拟化技术，Docker容器技术，也是一种虚拟化技术！

```shell
vm ,linux centos原生镜像（一个电脑！） 隔离，需要开启多个虚拟机！ 几个G  几分钟
docker:隔离，镜像（最核心的环境4m+jdk+mysql）十分的小巧，运行镜像就可以了！小巧！ 几个M  KB 秒级启动！
```

