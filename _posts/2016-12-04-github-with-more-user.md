---
layout: post
title: Mac下配置多个github用户SSH认证
date: 2016-12-04 19:30
categories: github ssh mac git
---

github使用SSH与客户端连接。如果是单用户（first），生成密钥对后，将公钥保存至github，每次连接时SSH客户端发送本地私钥（默认~/.ssh/id_rsa）到服务端验证。单用户情况下，连接的服务器上保存的公钥和发送的私钥自然是配对的。但是如果是多用户（first，second），我们在连接到second的帐号时，second保存的是自己的公钥，但是SSH客户端依然发送默认私钥，即first的私钥，那么这个验证自然无法通过。不过，要实现多帐号下的SSH key切换在客户端做一些配置即可。  

首先cd到~/.ssh 使用 ssh-keygen -t -rsa -C ‘second@mail.com’ 生成新的SSH key：id_rsa_second，生成完后将新的SSH public key添加到github。  

```
ssh-keygen -t rsa -C 'second@mail.com'
```

默认SSH只会读取id_rsa(**注意：但是我Mac上面没有读取，所以还是要添加**)，所以为了让SSH识别新的私钥，需要将其添加到SSH agent  

```
ssh-add ～/.ssh/id_rsa
ssh-add ～/.ssh/id_rsa_second

➜  .ssh ssh-add -l           
2048 SHA256:oUkeFCsL31Y/jNTEmdJsxMGovAXP2G1QyrUGnC+S5dE /Users/zhangwei/.ssh/id_rsa_zhanwe (RSA)
2048 SHA256:Vt5ayPNg2xcBUxzrrQFPhfcCJjY2+/v3YcfNQ3x5jlE /Users/zhangwei/.ssh/id_rsa (RSA)
```

该命令如果报错：Could not open a connection to your authentication agent.无法连接到ssh agent，可执行ssh-agent bash命令后再执行ssh-add命令。  

完成以上步骤后在~/.ssh目录创建config文件，该文件用于配置私钥对应的服务器。内容如下： 

``` 
# Default github user(zw900808@gmail.com)
Host github.com
HostName github.com
User zhangwei900808
IdentityFile /Users/zhangwei/.ssh/id_rsa

# second user(zhangwei900808@outlook.com)
Host github2
HostName github.com
User iewgnahz
IdentityFile /Users/zhangwei/.ssh/id_rsa_zhanwe

# gitlab (zw900808@gmail.com)
Host gitlab.com
HostName gitlab.com
User awbeci
IdentityFile /Users/zhangwei/.ssh/id_rsa
```

测试是否连接成功：

```
➜  .ssh ssh -T git@gitlab.com
Welcome to GitLab, awbeci!
```

Host随意即可，方便自己记忆，后续在添加remote是还需要用到。  
配置完成后，在连接非默认帐号的github仓库时，远程库的地址要对应地做一些修改，比如现在添加second帐号下的一个仓库test，则需要这样添加：  

```
git remote add test git@github-second:second/test.git #并非原来的git@github.com:second/test.git  
```

这样每次连接都会使用id_rsa_second与服务器进行连接。至此，大功告成！  
注意：github根据配置文件的user.email来获取github帐号显示author信息，所以对于多帐号用户一定要记得将user.email改为相应的email(second@mail.com)。  

**注意：**

```
1. 第2个账户在生成ssh的时候要注意重命名id_rsa_second
2. 在添加ssh-add的时候一定要记住要执行两次：ssh-add ~/.ssh/id_rsa和ssh-add ~/.ssh/id_rsa_second否则在你ssh -T的时候两次用户名相同
3. 因为设置了多用户所以全局的用户名和邮箱就可以不用设置，只要在相应的项目里面设置就可以了，如：

1.取消global
git config --global --unset user.name
git config --global --unset user.email

2.设置每个项目repo的自己的user.email还有remote
git config  user.email "xxxx@xx.com"
git config  user.name "suzie"
```

参考：

> <http://jingyan.baidu.com/article/948f592414ad67d80ef5f966.html>

> <http://blog.csdn.net/workformywork/article/details/16850463>
