---
layout: post
title: GitHub 不再支持用账户密码进行 Git 项目操作
subtitle: 正巧让我给赶上了…
date: 2021-08-14 17:30:00
categories: [Operation]
author: MowChan
tags: [GitHub, Git]
nightly: true
---

GitHub 考虑到账户安全，从 2021年8月13日（北京时间14日）起不再允许用 GitHub 账户密码对 Repository 进行操作，也就是无法用 GitHub 账号密码或是 SSH 地址携带密码 Clone 访问受限的 Repository。GitHub 官方推荐使用基于 RSA 的 SSH Key 或者账号的 Token Key 替代。

但是因为我有两个账户在使用，为了方便主账号采用 SSH Keys 作为全局账号使用，副账号使用账号密码直接操作，但是 Git 全局配置只能有一个全局账号，所以需要做一些调整。

### 解决方法

#### 方法一、使用 SSH Keys

首先，分别为两个 GitHub 账号生成 SSH 密钥

```shell
$ cd ~/.ssh
$ ssh-keygen -t rsa -f id_rsa_1 -C "username1@mail.com"
$ ssh-keygen -t rsa -f id_rsa_2 -C "username2@mail.com"
```
在`~/.ssh/`（Linux） `C:\Users\用户名\.ssh\`（Windows）路径下会出现四个文件：

```
id_rsa_1
id_rsa_1.pub
id_rsa_2
id_rsa_2.pub
```
然后在 GitHub 上 [https://github.com/settings/keys](https://github.com/settings/keys) 配置 SSH Keys，将公钥（`*.pub`文件）的所有内容填到 GitHub 上。

之后在`.ssh`目录下配置`config`文件

```yaml
# 第一个账号
Host github.com
    HostName github.com
    User username1
    IdentityFile ~/.ssh/id_rsa_1

# 第二个账号
Host github.com
    HostName github.com
    User username2
    IdentityFile ~/.ssh/id_rsa_2
```

最后在项目目录下配置该项目的用户名、邮箱

```shell
$ git config user.name 'your_username'
$ git config user.email 'your_email'
```

或者直接编辑项目目录下的 `.git/config` 文件

```yaml
[user]
	name = your_username
	email = your_email
```


#### 方法二、使用 Token Keys

在 [https://github.com/settings/tokens](https://github.com/settings/tokens) 配置 Token Keys，如果只需要 Repository 的 Git 基本操作，权限列表中只勾选 Repo 即可，确认后会生成 Token Keys。注意 Token 仅此显示一次，以后不再显示，所以要保存好 Token。

在项目的目录下配置该项目的用户名、邮箱以及密码

```shell
$ git config user.name 'your_username'
$ git config user.email 'your_email'
$ git config user.password 'your_token'
```

或者直接编辑 `.git/config` 文件，增加如下几行

```yaml
[user]
	name = your_username
	email = your_email
	password = your_token
```

即可以该账户操作。

### 参考来源

- [Token authentication requirements for Git operations | The GitHub Blog](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)
- [Creating a personal access token - GitHub Docs](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [同一个电脑配置两个github账号_weixin_30883271的博客-CSDN博客](https://blog.csdn.net/weixin_30883271/article/details/96297742)

