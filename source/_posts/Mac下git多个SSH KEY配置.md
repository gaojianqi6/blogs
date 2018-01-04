自己的电脑上有用到几个不同的git，之前只是clone github上http的地址拿下来看，后来个人项目也需要开发，就需要好几个ssh key，写下这个记录一下过程，省的每次忘记流程还重新记录。

###### 生成SSH KEY
- 生成SSH KEY时，首先进入`~/.ssh/`目录下，使用`ssh-keygen -t rsa -C "your_email@example.com"`生成ssh key信息，第一次提示"Enter file in which to save the key (/Users/JeromeGao/.ssh/id_rsa)"时，可以使用默认文件名字，但是之后生成ssh key记得填写相应的文件名，，以便区分，"Enter passphrase (empty for no passphrase)"是私钥密码，可以直接回车也可以输入密码，如果输入密码请记住私钥的密码， 下面导入私钥的时候要用到。
命令：
```
cd ~/.ssh/
➜  .ssh ssh-keygen -t rsa -C "your_email@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/JeromeGao/.ssh/id_rsa): id_rsa_gittest
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_gittest.
Your public key has been saved in id_rsa_gittest.pub.
The key fingerprint is:
SHA256:5/5+BIyhVKH3MRoGIs5vn36aKmuppQqgIBJ69KTHND8 gaojianqi6@126.com
The key's randomart image is:
+---[RSA 2048]----+
|    . . ..o.     |
|   o . ..o.      |
|. . *  ...++o    |
|.o * +  .o.+oo   |
|* o + E S o ..   |
|*. . . o +    .  |
|o   ..  o .  .   |
|.  o+  . o.   .  |
|..oo.o..+o.oo.   |
+----[SHA256]-----+
```
- 将SSH 私钥增加到ssh-agent: `ssh-add ~/.ssh/xxx_rsa`， 这里会提示输入一次私钥的密码;
- 查看已经添加过的SSH KEY： ssh-add -l；
- 在相对应的网站上找到SSH KEYS，然后打开`~/.ssh/id_rsa.pub`, 复制公钥内容到Key中，然后点击添加就完成了

###### 一台机器上管理多个GIT帐号的SSH KEY
- 如果你在一台机器上，使用多个git账号，则重复上面步骤之后，需要编辑文件`~/.ssh/config`:
```
#Github  (github 配置)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github

#company  (公司git 配置)
Host company
    HostName git.company.com
    User git
    IdentityFile ~/.ssh/id_rsa

  #gitlab  (gitlab 配置)
  Host gitlab
      HostName gitlab.com
      User git
      IdentityFile ~/.ssh/id_rsa_gitlab
```
解释一下这个配置文件：

Host：一个"别名"，可以随意命名, 像git、company这样的命名也可以；
HostName：工作的git仓储地址;
IdentityFile： 所使用的公钥文件
配置完毕，用下面的命令测试一下：

###### 测试SSH KEY
```
# 测试是否联通，出现welcome就表示成功
➜  .ssh ssh -T git@gitlab.com
Welcome to GitLab, !

# 可以出来调试信息
➜  .ssh ssh -v git@gitlab.com
```

记得配置自己的username和email，一般在网站新建项目时会提示你配置
```
git config user.name your_name
git config user.email your_email
```
###### ssh命令收集
```
# 将SSH 私钥增加到ssh-agent
ssh-add ~/.ssh/xxx_rsa
# 查看已经添加过的SSH KEY
ssh-add -l
# 运行ssh-agent，它会打印出来它使用的环境和变量
ssh-agent
```
