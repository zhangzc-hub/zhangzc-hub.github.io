---
title: github配置ssh秘钥
date: 2023-11-06 09:47:21
tags:
- ssh
categories:
- git
---

# github配置ssh秘钥

### 第一步：在 **PowerShell** 中，使用

```
dir ~\.ssh\
```

```
~ 在 PowerShell 中代表你的用户目录，比如 `C:\Users\18713
```

### 示例输出

如果已有 SSH 密钥，你会看到类似：

```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2025/11/4     11:30            411 id_ed25519
-a----         2025/11/4     11:30            102 id_ed25519.pub
-a----         2025/11/4     11:34            831 known_hosts
-a----         2025/11/3     18:11             92 known_hosts.old
```

如果没有 `.ssh` 文件夹，会提示：

```
Get-ChildItem: 找不到路径“C:\Users\18713\.ssh”，因为该路径不存在。
```

### 第二步：如果没有 `.ssh` 文件夹或密钥 → 生成一个！

| 算法                        | 公钥           | 私钥       |
| --------------------------- | -------------- | ---------- |
| ED25519（首选）             | id_ed25519.pub | id_ed25519 |
| RSA（至少 2048 位密钥大小） | id_rsa.pub     | id_rsa     |

在 **PowerShell** 中运行（推荐 ed25519 算法）：

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

生成`2048`位`RSA`

```
ssh-keygen -t rsa -b 2048 -C "your_email@example.com" 
```

文件生成到 `~/.ssh/` 目录

 把 `your_email@example.com` 换成你 **GitHub 账号注册的邮箱**

然后按三次回车（使用默认路径和空密码）：

```
Enter file in which to save the key (~/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

完成后，再运行：

```
dir ~\.ssh\
```

你应该就能看到 `id_ed25519` 和 `id_ed25519.pub` 了。

**启动 ssh-agent 服务**：

```
Start-Service ssh-agent
```

**添加私钥到 agent**：

```
ssh-add ~\.ssh\id_ed25519
```



### 第三步：把公钥内容添加到 GitHub

你现在需要把 **公钥**（`.pub` 文件）的内容复制到 GitHub 上。

#### 1. 复制公钥内容

powershell

```
Get-Content ~/.ssh/id_ed25519.pub
```

或者：

powershell

```
cat ~/.ssh/id_ed25519.pub
```

你会看到一长串文本，类似：

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG... your_email@example.com
```

👉 **全选并复制这一整行**（包括开头 `ssh-ed25519` 和结尾邮箱）。

#### 2. 添加到 GitHub

1. 打开浏览器，登录 GitHub
2. 点右上角头像 → **Settings**
3. 左侧菜单选 **SSH and GPG keys** → 点击 **New SSH key**
4. 填写：
   - **Title**: 比如 `My Windows PC`
   - **Key type**: 选 `Authentication`
   - **Key**: 粘贴你刚刚复制的那整行公钥
5. 点击 **Add SSH key**

### 第四步：再次测试连接

```
ssh -T git@github.com
```

正确响应应该是：

```
Hi zhangzc-hub! You've successfully authenticated, but GitHub does not provide shell access.
```

这说明你已经通过 SSH 认证成功，现在可以正常使用 `git pull`, `git push` 了！
