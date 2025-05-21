---
title: yuque-dl
date: 2025-05-21 14:59:27
tags:
- 语雀
categories:
- cmd
---

# **语雀文档下载到本地**

1.下载nodejs 点此直接下载Node.js v22.13.1 去官网查看：

```
https://nodejs.org/zh-cn 
```

2.win+r打开运行，输入cmd打开命令提示符 

3.更换镜像源 

```
npm config get registry   
npm config set registry https://registry.npmmirror.com
```

4.安装插件 

```
npm i -g yuque-dl 
```

 5.查看是否下载完成 

```
yuque-dl --help  
```

6.下载文档 yuque-dl  示例： 

```
yuque-dl "https://www.yuque.com/yuque/thyzgp" 
```

================================= 修改默认路径 在cmd里输入以下命令 mklink/J "C:\Users\你的用户名\download" "目标位置"  例如：

```
mklink/J "C:\Users\hp\download" "D:\download" 
```

当显示：为xxx<<===>>xxx创建的连接即可 然后正常下载

7、出现下面图片中报错的情况的，按下面的方法解决
![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-05-21_14-49-33.jpg)

### 私有知识库

通过别人私有知识库 分享的链接，需使用`-t`添加token才能下载

```
yuque-dl "https://www.yuque.com/yuque/thyzgp" -t "abcd..."
```

### 公开密码访问的知识库

⚠️ 公开密码访问的知识库两种情况:

- 已经登录语雀，访问需要密码的知识库 输入密码后使用`_yuque_session`这个cookie

  ```
  yuque-dl "url" -t "_yuque_session的值"
  ```

- 未登录语雀，访问需要密码的知识库 输入密码后需要使用`verified_books`/`verified_docs`这个cookie

  ```
  yuque-dl "url" -k "verified_books" -t "verified_books的值"
  ```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-05-21_14-54-24.jpg)

## 内置启动web服务可快速预览

使用[`vitepress`](https://gitee.com/link?target=https%3A%2F%2Fvitepress.dev%2F)快速启动一个web服务提供可预览下载的内容	

```
yuque-dl server ./download/知识库/

➜  Local:   http://localhost:5173/
➜  Network: use --host to expose
```

