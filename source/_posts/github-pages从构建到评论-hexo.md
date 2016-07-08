---
title: Github Pages从构建到评论(HEXO)
date: 2016-05-17 15:35:58
tags: [github pages, ssh key, hexo, hexo-next]
categories: [构建工具]
---

这是我第一次搭建博客，本来是准备自己买服务器，然后写代码来搞一套，后来无意中发现有人提到github的博客，然后搜索了一下发现是github pages,就开始跟着教程一步步搭建博客，之间走过了很多的坑，因为现在网上的很多教程都是很久以前的资料，而且到处搬来搬去，在这里总结一下。

## GitHub Pages是什么？

GitHub Pages 本用于介绍托管在 GitHub 的项目， 不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。

## 使用 GitHub Pages 建立博客
### 注册GitHub
访问：http://www.github.com/
注册你的username和邮箱，邮箱十分重要，GitHub上很多通知都是通过邮箱的。

### 建立博客项目
与 GitHub 建立好链接之后，就可以方便的使用它提供的Pages服务，GitHub Pages分两种，一种是你的GitHub用户名建立的username.github.io这样的用户&组织页（站），另一种是依附项目的pages。

想建立个人博客是用的第一种，形如gaojianqi6.github.io这样的可访问的站，每个用户名下面只能建立一个。

在github.com页面选择 New repository, 然后在Repository name里填写username.github.io（必须为这个仓库名）

## 开始Hexo
首先本地得装上了Node.js、Git和Hexo
	安装Node.js: [Nodejs官网](https://nodejs.org/en/)
	安装Git: [Git](http://git-scm.com/) 
			[安装Git - 廖雪峰的官方网站](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000/ "安装Git - 廖雪峰的官方网站")
	安装Hexo: [Hexo](https://hexo.io/ "hexo官网")

下载安装之后可以在本地启动查看一下，默认监听4000端口，打开浏览器，输入 localhost:4000 就可以看到一个hexo的默认博客主题页面出现在你面前。
## 配置 SSH keys
我们如何让本地git项目与远程的github建立联系呢？用SSH keys。
### 检查 SSH keys的设置
首先我们需要检查你电脑上现有的ssh key：
``` bash
cd ~/.ssh 检查本机的ssh密钥
```
如果提示：No such file or directory 说明你是第一次使用git。

### 生成新的SSH Key：
``` bash
ssh-keygen -t rsa -C "邮件地址@youremail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>
```
然后系统会要你输入密码：
```bash
Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
```
在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。

注意：输入密码的时候没有*字样的，你直接输入就可以了。
### 添加 SSH Key 到 GitHub

在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。

1、打开本地~\id_rsa.pub文件。此文件里面内容为刚才生成人密钥。如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。
2、登陆github系统。点击右上角的 Account Settings—>SSH Public keys —> add another public keys
3、把你本地生成的密钥复制到里面（key文本框中）， 点击 add key 就ok了
### 测试

可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：
``` bash
ssh -T git@github.com
```
### 多个SSH Key
1.生成指定名字的密钥
```bash 
ssh-keygen -t rsa -C "邮箱地址" -f ~/.ssh/id_rsa_github
```

	会生成id_rsa.pub和id_rsa_github两个文件
2.密钥复制到托管平台上
```bash 
$  vim ~/.ssh/id_rsa_github.pub 
```
3.修改config文件
```bash
vim ~/.ssh/config #修改config文件，如果没有创建 config
```
	#Github  (github 配置)
	Host github.com
	    HostName github.com
	    User git
	    IdentityFile ~/.ssh/id_rsa_github
	 
	#oschina.net (开源中国 配置)
	Host oschina.net
	    HostName git.oschina.net
	    User git
	    IdentityFile ~/.ssh/id_rsa
配置完之后：
![配置SSH完成](http://o7bfyflfi.bkt.clouddn.com/ssh.jpg)
修改完之后可以测试一下。
``` bash
ssh -T git@github.com   #github.com 对应Host
```

## 部署到github
到github新建一个项目，项目名为：*你的用户名*.github.io必须为这个名字

配置文件*_config.yml*

``` yml
deploy:
  type: git
  repository: git@github.com:你的帐号/你的帐号.github.com.git
  例如我的：repository: git@github.com:gaojianqi6/gaojianqi6.github.io.git
  branch: master
```
之前有的教程deploy的type是github,这里需要改成git，然后运行下

``` bash
npm install hexo-deployer-git --save
```

然后执行命令：
``` bash
hexo clean
hexo generate
hexo deploy
```
命令缩写
```bash
hexo n #写文章
hexo g #生成
hexo d #部署 # 可与hexo g合并为 hexo d -g
```
在source文件夹里面添加CNAME文件，里面填写自己的独立域名。
有的使用Hexo和Github部署好之后但是一直404，有可能就是没有在根目录下面添加CNAME文件。

PS:以后的404页面，favicon.ico也是直接在source下面添加

## 将独立域名与 GitHub Pages 的空间绑定
选择ping命令之后出现的地址，在您自己的域名管理里面更改一下域名解析，这个可能需要几分钟之后才能生效。
``` bash
$ ping username.github.io
```

## 选择Hexo主题

Hexo有很多可以选择的主题，可以在下面的链接中选择自己喜欢的主题：
[有哪些好看的 Hexo 主题？](https://www.zhihu.com/question/24422335)
[Hexo themes](https://hexo.io/themes/ "Hexo官网主题站")

我选择了[Next](https://github.com/iissnan/hexo-theme-next)主题，在这里查看怎么安装Next -> [安装Next](http://theme-next.iissnan.com/getting-started.html)

## 目录介绍

默认目录结构：

	.
	├── .deploy
	├── public
	├── scaffolds
	├── scripts
	├── source
	|   ├── _drafts
	|   └── _posts
	├── themes
	├── _config.yml
	└── package.json

	.deploy：执行hexo deploy命令部署到GitHub上的内容目录
	public：执行hexo generate命令，输出的静态网页内容目录
	scaffolds：layout模板文件目录，其中的md文件可以添加编辑
	scripts：扩展脚本目录，这里可以自定义一些javascript脚本
	source：文章源码目录，该目录下的markdown和html文件均会被hexo处理。该页面对应repo的根目录，404文件、favicon.ico文件，CNAME文件等都应该放这里，该目录下可新建页面目录。
	_drafts：草稿文章
	_posts：发布文章
	themes：主题文件目录
	_config.yml：全局配置文件，大多数的设置都在这里
	package.json：应用程序数据，指明hexo的版本等信息，类似于一般软件中的 关于 按钮


## 404页面
有很多用作公益的404接入地址，我选择了腾讯的。404页面，每个人可以做的更多一点。
	[腾讯公益404](http://www.qq.com/404)
	[404公益_益云(公益互联网)社会创新中心](http://yibo.iyiyun.com/Index/web404)
	[失蹤兒童少年資料管理中心404](http://404page.missingkids.org.tw/)
在source目录下添加新建404.html，然后复制内容：
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>宝贝，公益404带你们回家</title>
</head>
<body>
    <script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="http://yoursitename.com" homePageName="回到我的主页"></script>
</body>
</html>
```

## 评论插件
流行的评论插件有: [多说](http://duoshuo.com/)、[友言](http://www.uyan.cc/)、[Disqus](https://disqus.com/)等
安装多说主题又是Next的，可以在这里[查看](http://theme-next.iissnan.com/getting-started.html#comment-system-duoshuo)。
## 图床
考虑到博客的速度，同时也为了便于博客的迁移，图床是必须的。可以看一下[七牛](http://www.qiniu.com/)，访问速度极快，支持日志、防盗链和水印。
也还可以考虑下面的图床服务 [FarBox](http://www.farbox.com/) ， [Dropbox](http://www.dropbox.com/) ， [又拍云](http://www.upyun.com/) 。

