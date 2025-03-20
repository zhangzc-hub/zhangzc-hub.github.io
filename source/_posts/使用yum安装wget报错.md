---
title: 使用yum安装wget报错
date: 2024-01-13 16:42:09
tags:
- yum
categories:
- Linux
---

# 使用yum安装wget报错

使用yum安装报错：Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&

安装wget命令

```
yum -y install wget
```

报错，无法找到镜像

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_17-58-42.jpg)

测试是否是网络问题

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_17-59-35.jpg)

抓包正常，网络没有问题；尝试更新[yum](https://so.csdn.net/so/search?q=yum&spm=1001.2101.3001.7020)

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_18-00-21.jpg)

又开始报错

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_18-01-19.jpg)

尝试分析问题原因
出现这个错误是因为使用的 CentOS 7 仓库已经被归档，当前的镜像地址无法找到所需的文件。CentOS 7 的官方支持已经结束，部分仓库已被移至归档库。这导致了你的 yum 命令无法找到所需的元数据文件。CentOS 7 的官方仓库在 2024 年 6 月 30 日之后已经停止维护。因此，使用最新的 CentOS 7 官方仓库可能会遇到问题。


**尝试解决：**

进入/etc/yum.repos.d目录下找到 CentOS-Base.rep

执行下面操作

```
cp  CentOS-Base.repo   CentOS-Base.repo.backup
```

然后修改 CentOS-Base.repo 为

    # CentOS-Base.repo
        #
        # The mirror system uses the connecting IP address of the client and the
        # update status of each mirror to pick mirrors that are updated to and
        # geographically close to the client.  You should use this for CentOS updates
        # unless you are manually picking other mirrors.
        #
        # If the mirrorlist= does not work for you, as a fall back you can try the 
        # remarked out baseurl= line instead.
        #
        #
         
        [base]
        name=CentOS-$releasever - Base
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
        #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
        #baseurl=http://vault.centos.org/7.9.2009/x86_64/os/
        baseurl=http://vault.centos.org/7.9.2009/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
         
        #released updates 
        [updates]
        name=CentOS-$releasever - Updates
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
        #baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
        #baseurl=http://vault.centos.org/7.9.2009/x86_64/os/
        baseurl=http://vault.centos.org/7.9.2009/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
         
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
        #$baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
        #baseurl=http://vault.centos.org/7.9.2009/x86_64/os/
        baseurl=http://vault.centos.org/7.9.2009/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
         
        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
        #baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
        #baseurl=http://vault.centos.org/7.9.2009/x86_64/os/
        baseurl=http://vault.centos.org/7.9.2009/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
wq保存，再执行

```
sudo yum clean all
sudo yum makecache
```

等待加载成功，然后继续执行

```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_18-28-41.jpg)

然后执行

```
cat CentOS-Base.repo
```

发现镜像已改为了阿里云的

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_18-29-10.jpg)

再此尝试导入wget命令 好的成功了

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-18_18-29-36.jpg)