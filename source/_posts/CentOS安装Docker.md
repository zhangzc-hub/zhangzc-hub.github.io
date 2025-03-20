---
title: CentOS安装Docker\Docker-compose
date: 2023-12-19 13:53:20
tags:
- docker
categories:
- Linux
---

# CentOS安装Docker\Docker-compose



# CentOS安装Docker

## 1、卸载（可选）

如果之前安装过旧版本的Docker，可以使用下面命令卸载：

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

## 2、安装docker

首先需要大家虚拟机联网，安装yum工具

```
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```

然后更新本地镜像源：

```
# 设置docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
#将所有的 download.docker.com 替换为 mirrors.aliyun.com/docker-ce
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

#输入以下命令来更新 yum 缓存
yum makecache fast
```

然后输入命令：

```
yum install -y docker-ce
```

docker-ce为社区免费版本。稍等片刻，docker即可安装成功。

## 3、启动docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。

这里展示直接关闭防火墙的命令

```
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```

通过命令启动docker：

```
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务

systemctl enable docker  # 开机自启docker服务

systemctl start docker  # 启动docker服务
```

然后输入命令，可以查看docker版本：

```
docker -v
```

## 4、配置镜像加速

docker官方镜像仓库网速较差，导致拉取镜像失败，如下面这种错误，

```
docker pull 报错 Error response from daemon: Get “https://registry-1.docker.io/v2/“
```

我们需要设置国内镜像服务：

解决方案如下：

```
vi /etc/docker/daemon.json
```

添加以下内容：

```
{
  "data-root": "/home/docker-root",
  "registry-mirrors": [
  	"https://registry.dockercn.com",
  	"https://0mmuy3ea.mirror.aliyuncs.com/","https://docker.m.daocloud.io/"
  	]
}
```

然后保存

```
:wq
```

重启docker

```
systemctl daemon-reload
systemctl restart  docker
```

# centos安装docker-compose

## 1、最新版本查询

Docker Compose 的最新版本会不断更新，你可以通过以下几种方式来查看目前的最新版本：

- **官方发布页面**：访问 [Docker Compose 的 GitHub 发布页面](https://github.com/docker/compose/releases)，该页面会列出所有的版本信息，页面上显示的最新版本就是目前可获取的最新版本。
- **命令行查看**：如果你已经安装了 Docker Compose，在终端中执行 `docker-compose version` 命令，就能看到当前安装的版本。要是想知道是否有更新版本，还是要参考官方发布页面。

## 2、安装最新版本

如果要安装最新版本的 Docker Compose，可以参考以下步骤（以 Linux 系统为例）：

```
# 获取最新稳定版本号
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)

# 下载指定版本的 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

上述脚本会自动获取最新版本号并完成 Docker Compose 的下载和安装，最后验证安装是否成功。

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-19_14-41-14.jpg)

## 3、获取最新版本补全脚本

若要使用最新版本的 Docker Compose 补全脚本，你可以把命令中的版本号替换成最新版本。获取最新版本号的方式有多种，例如访问 [Docker Compose 的 GitHub 发布页面](https://github.com/docker/compose/releases)。

假设你已经得知最新版本是 `v2.20.3`（实际使用时请替换为真正的最新版本），可以使用如下命令获取对应版本的补全脚本：

```
sudo curl -L https://raw.githubusercontent.com/docker/compose/v2.20.3/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-19_14-43-19.jpg)

运行完该命令后，重新打开一个终端让补全功能生效。

## 4、当使用命令无法安装时，可以下载源文件进行安装，步骤如下：

当你下载了 `compose-2.34.0.tar.gz` 文件后，通常意味着你下载的是 Docker Compose 的源码包，下面为你介绍安装 Docker Compose 的操作步骤：

### 1. 解压文件

你需要使用 `tar` 命令来解压下载的 `compose-2.34.0.tar.gz` 文件。在终端中执行以下命令：

bash

```
tar -zxvf compose-2.34.0.tar.gz
```

该命令会将压缩包解压到当前目录下，解压后会生成一个名为 `compose-2.34.0` 的文件夹。

### 2. 进入解压后的目录

使用 `cd` 命令进入解压后的文件夹：

bash

```
cd compose-2.34.0
```

### 3. 构建并安装 Docker Compose

一般情况下，你可以使用 `make` 命令来构建和安装 Docker Compose。不过在执行 `make` 命令之前，需要确保你的系统已经安装了 `make` 工具和其他必要的依赖项。执行以下命令：

bash

```
make
sudo make install
```

- `make` 命令会根据 `Makefile` 文件中的规则来编译和构建 Docker Compose。
- `sudo make install` 命令会将构建好的二进制文件安装到系统中，使其可以在全局范围内使用。

### 4. 验证安装

安装完成后，你可以使用以下命令来验证 Docker Compose 是否安装成功：

bash

```
docker-compose --version
```

如果输出显示 Docker Compose 的版本为 2.34.0，则说明安装成功。

### 可能遇到的问题及解决办法

- **缺少依赖项**：如果在执行 `make` 命令时出现错误，提示缺少某些依赖项，你需要安装这些依赖项。常见的依赖项包括 `go` 语言环境等。你可以根据错误提示安装相应的依赖项。
- **权限问题**：如果在执行 `sudo make install` 命令时遇到权限问题，确保你具有足够的权限。通常使用 `sudo` 命令可以解决权限问题。