# Vuepress安装

转自：https://blog.csdn.net/ussim/article/details/107845065

[VuePress](https://www.vuepress.cn/) 是一个基于 [Vue](https://cn.vuejs.org/) 的轻量级静态网站生成器，以及为编写技术文档而优化的默认主题。


VuePress 要求 Node.js 环境，且 Node.js 版本 >= 8.6。



## 环境搭建

#### 安装 Node.js

从 [官方网站](http://nodejs.cn/download/) 获取最新版本安装包下载链接

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172149123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



```bash
# 创建安装目录
sudo mkdir /usr/local/lib/nodejs

# 进入安装目录
cd /usr/local/lib/nodejs

# 下载安装包
wget  https://npm.taobao.org/mirrors/node/v14.7.0/node-v14.7.0-linux-x64.tar.xz

# 解压
sudo tar -xJvf node-v14.7.0-linux-x64.tar.xz
1234567891011
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080617214925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)





#### 配置环境变量

```bash
# 编辑环境变量文件，在末尾追加 Node.js 路径
echo export PATH=/usr/local/lib/nodejs/node-v14.7.0-linux-x64/bin:$PATH >>~/.bash_profile

# 重载该文件使设置生效
source ~/.bash_profile

# 查看版本信息
node -v
npm version
npx -v
12345678910
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172149104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)





## 安装 VuePress

```bash
# 使⽤淘宝镜像
npm config set registry https://registry.npm.taobao.org
# 安装 VuePress
npm install -g vuepress
1234
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172149163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



```bash
# 在 root 目录下创建并进入my_blogs目录
mkdir ~/my_blogs
cd ~/my_blogs

# 项目初始化 
npm init -y
123456
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/202008061721491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)





## 配置 VuePress

```bash
# 设置 package.json 的脚本配置 (推荐使用 FinalShell等工具直接编辑)
vim package.json
# 修改scripts中的内容如下
123
 "scripts": {
 "docs:dev": "vuepress dev docs",
 "docs:build": "vuepress build docs"
 },
1234
```


修改前

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172148960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)


修改后

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080617214961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



```bash
# 创建文档目录以及 .vuepress ⽬录
mkdir ~/my_blogs/docs ~/my_blogs/docs/.vuepress

# 在 docs 目录下新建⼀个md⽂件 
echo '# Hello VuePress - ﬁrst blog!' > ~/my_blogs/docs/README.md 

# 在 .vuepress 目录下创建 conﬁg.js 配置⽂件 
cd ~/my_blogs/docs/.vuepress 
echo >conﬁg.js 

# 在 .vuepress 目录下创建 public ⽬录 
mkdir ~/my_blogs/docs/.vuepress/public  
123456789101112
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080617214994.png)



在 my_blogs 路径下查看目录

```bash
tree -a

# 如果没有安装 tree
# centos 系统下使用
sudo yum install tree
# Ubuntu 系统下使用
sudo apt-get -y install tree
1234567
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172148950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



修改首页配置

编辑 **/root/my_blogs/docs** 目录下的 **README.md** 文件

```markdown
---
home: true
heroText: Vue技术博客初试
tagline: 项目结构，关注讨论，每日分享
actionText: 每日更新 →
actionLink: /testlink/
features:
- title: 项目结构
	- details: 以 Markdown 为中心的项目结构，以最少的配置帮助你专注于写作。
- title: 关注讨论
	- details: 享受 Vue + webpack 的开发体验，在 Markdown 中使用 Vue 组件，同时可以使用Vue 来开发自定义主题。
- title: 每日分享
	- details: VuePress 为每个页面预渲染生成静态的 HTML，同时在页面被加载的时候，将作为 SPA 运行。
footer: LearnVueonECS Licensed | Copyright © 2020-present
---
123456789101112131415
```





## 启动 VuePress

```bash
vuepress dev ~/my_blogs/docs
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172149110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



访问网站 **http://<你的公网IP地址>:8080/**

```bash
vuepress dev ~/my_blogs/docs
1
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806172149217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vzc2lt,size_16,color_FFFFFF,t_70)



至此，VuePress搭建完毕。