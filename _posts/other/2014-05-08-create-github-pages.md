---
layout: post
title: "基于GitHub Pages搭建个人博客"
tagline: ""
keywords: "基于GitHub Pages搭建个人博客,GitHub,Git,GitHub Pages,SSH,Blog,博客"
description: ""
category: web
tags : [Git, GitHub]
---
{% include JB/setup %}

Github作为全球最流行的代码仓库，已经得到越来越多的公司和项目的青睐。如何让世界各地的程序员协作开发，
为此GitHub提供了GitHub Pages服务，我们可以利用这个服务建立站点，将你的使用文档，说明文档等放入其中，供他人阅读使用。
当然我们也可以用它来建立自己的个人博客站点。





**优点：**

- 轻量级的博客系统，你不需要做烦人的配置，不需要搭建服务器。
- jekyll使用Liquid模板语言，使用标记语言，比如Markdown。
- 不限空间。
- 可以绑定自己的域名。

**缺点：**

- 只能生成静态网站，适合博客，文档介绍等。
- 没有评论等动态交互程序，但是可以通过第三插件实现。
- 对于使用者要求有一定的开发基础，可能更适合程序员等使用。

**以下所有操作都是基于Windows操作系统环境。**

## 注册GitHub账号

使用[GitHub](https://github.com "GitHub")托管代码，必须现在[GitHub](https://github.com)上注册一个账号。

## 安装Git

[下载Git for Windows](http://git-scm.com/downloads "下载Git for Windows")，使用默认安装，安装完成后即可使用Git。

## 配置和使用GitHub

打开 Git Bash

### 1、检查SSH key设置

查看自己电脑上的SSH key设置：

    $ cd ~/.ssh

如果提示为“No such file or directory”，直接跳到第三步，否则继续。

### 2、备份以及删除旧的SSH key

为了保险起见，先备份后再重新生成：

    $ ls
    config  id_rsa  id_rsa.pub  known_hosts
    $ mkdir key_backup
    $ cp id_rsa* key_backup
    $ rm id_rsa*

### 3、生成新的SSH key

填写你自己的GitHub邮件地址，提示要输入信息的地方直接回车就OK：

    $ ssh-keygen -t rsa -C "邮件地址@youremail.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):

把生成的key文件复制到C:\Documents and Settings\Administrator\\.ssh\目录下。
必须把id_rsa、id_rsa.pub这两个文件放到当前用户目录的“.ssh”目录下才能生效。
在windows中只能在命令行下输入创建“.”开头的文件夹，命令为：

    $ mkdir .ssh

### 4、GitHub SSH key设置

设置Git用户信息
第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，
所以会随更新内容一起被永久纳入历史记录，用户名称必须是你GitHub真实的账号名称：

    $ git config --global user.name "username"
    $ git config --global user.email lisan@example.com

Git和GitHub之间使用SSH连接，必须在GitHub上设置好key，才能正常连接。

进入GitHub主页，进入账号设置：
![setting](/images/githubpages/github-setting.jpg)
点击左侧SSH keys选项，点击Add SSH key，然后打开C:\Documents and Settings\Administrator\\.ssh\目录下id_rsa.pub文件，
复制里面的内容粘贴到key文本框中，点击保存即可：
![add ssh key](/images/githubpages/add-sshkey.jpg)

### 5、测试SSH连接

输入下面命令测试SSH连接是正常：

    $ ssh -T git@github.com

提示以下信息，表示SSH连接成功：

    $ ssh -T git@github.com
    Hi username! You've successfully authenticated, but GitHub does not provide shell access.

恭喜你，现在你可以使用Git向GitHub提交东西。

## 建立自己的博客

使用[GitHub Pages](https://pages.github.com/ "GitHub Pages")建立自己的博客。
GitHub Pages提供两种方式建立博客站点：

1. 基于用户或组织的站点（User or organization site）。
2. 基于项目的站点（Project site）。

### 用户或组织站点（User or organization site）

你必须建立一个项目名称为username.github.io的项目，其中username是在Github上的用户名。
使用此种方式建立站点后，当你访问http://username.github.io/时，Github会解析用户username下的master分支下的项目源代码，
生成静态网站，并将生成的index.html文件展示给你。我就是使用这种方式建立的博客。

![setting](/images/githubpages/create-user-githubpages.jpg)

打开Git bash，在你需要保存项目的目录下clone刚才建立的项目username.github.io

    $ git clone https://github.com/username/username.github.io

使用下面命令，进入项目根目录，建立一个index.html文件，提交文件

    $ cd username.github.io
    $ echo "Hello World" > index.html
    $ git add --all
    $ git commit -m "Initial commit"
    $ git push

恭喜你，你的博客站点建立成功，访问http://username.github.io/即可，
第一次生成站点时，大概需要10左右延迟时间，后续修改提交后，会立即生效。

### 项目站点（Project site）

这种方式跟第一种方式后台程序处理机制是一样的，只是访问方式不一致。
使用http://username.github.io/repository访问，GitHub会自动解析username用户下的
repository项目的gh-pages分支源代码，生成静态网站并展示给你，所以你必须在项目下建立名为gh-pages的分支。
使用这种方式更适合为每个项目建立使用说明文档之类的站点。
你可以通过两种方式建立：

**1、自动生成站点：**

![setting](/images/githubpages/create-project-githubpages-11.jpg)
![setting](/images/githubpages/create-project-githubpages-12.jpg)
![setting](/images/githubpages/create-project-githubpages-13.jpg)
![setting](/images/githubpages/create-project-githubpages-14.jpg)

**2、建立gh-pages分支生成站点：**

建立gh-pages分支，添加index.html文件。

![setting](/images/githubpages/create-project-githubpages-21.jpg)
![setting](/images/githubpages/create-project-githubpages-22.jpg)
![setting](/images/githubpages/create-project-githubpages-23.jpg)

大概需要10左右延迟时间，你就可以使用http://username.github.io/repository访问你的站点了。


至此一个简单的个人博客框架搭建好了，你只需要往里面添加文件以及内容就行了。

后续我将继续写一个博客，如何搭建一个jekyll windows环境，这样就可以在本地进行内容修改添加，在本地预览无误后再发布到GitHub。