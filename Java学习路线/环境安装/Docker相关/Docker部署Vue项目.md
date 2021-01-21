# Docker部署Vue项目

### 1、将项目打包，然后将dist文件夹传到服务器上

### 2、获取 nginx 镜像

```bash
docker pull nginx
```

### 3、创建 nginx config配置文件

在项目根目录下创建`nginx`文件夹，该文件夹下新建文件`default.conf`

```bash

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

该配置文件定义了首页的指向为 `/usr/share/nginx/html/index.html`, 所以我们可以一会把构建出来的index.html文件和相关的静态资源放到`/usr/share/nginx/html`目录下。

### 4、项目根目录下创建Dockerfile文件

```bash
FROM nginx
COPY dist/ /usr/share/nginx/html/
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
```

>自定义构建镜像的时候基于Dockerfile来构建。
>
>`FROM nginx` 命令的意思该镜像是基于 nginx:latest 镜像而构建的。
>
>`COPY dist/ /usr/share/nginx/html/` 命令的意思是将项目根目录下dist文件夹下的所有文件复制到镜像中 /usr/share/nginx/html/ 目录下。
>
>`COPY nginx/default.conf /etc/nginx/conf.d/default.conf` 命令的意思是将nginx目录下的default.conf 复制到 etc/nginx/conf.d/default.conf，用本地的 default.conf 配置来替换nginx镜像里的默认配置。

5、基于该Dockerfile构建vue应用镜像

```bash
docker build -t vuenginxcontainer .
```

> `-t` 是给镜像命名 `.` 是基于当前目录的Dockerfile来构建镜像

6、启动容器

```bash
docker run -p 8000:80 -d --name vueApp vuenginxcontainer
```

> - `docker run` 基于镜像启动一个容器
> - `-p 8000:80` 端口映射，将宿主的8000端口映射到容器的80端口
> - `-d` 后台方式运行
> - `--name` 容器名 查看 docker 进程

```bash

docker run --name consul -p 8500:8500 -v /data/consul/conf/:/consul/conf/ -v /data/consul/data/:/consul/data/ -d consul

```

