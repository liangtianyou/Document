# 使用gerrit进行code review

[TOC]

**搭建环境**: Ubuntu 14.04

## 1 环境准备
### 1.1 Java环境
gerrit依赖，用于安装gerrit环境。
#### 1.1.1 下载
[jdk-7u79-linux-x64.tar.gz](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

#### 1.1.2 安装
```bash
sudo tar zxvf ./jdk-7u79-linux-x64.tar.gz -C /opt
```
#### 1.1.3 配置
1. 针对当前用户: ```~/.bashrc```
2. 针对所有用户，推荐: /etc/profile

```bash
export JAVA_HOME=/opt/jdk1.7.0_79
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
#### 1.1.4 验证
```bash
java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

### 1.2 git环境
gerrit依赖，用来操作git repository。
#### 1.2.1 安装
```bash
sudo apt-get install git
```
#### 1.2.2 验证
```bash
git --version
git version 1.9.1
```

### 1.3 gerrit环境
#### 1.3.1 下载
[Gerrit 2.12.4](https://www.gerritcodereview.com/download/gerrit-2.12.4.war)


### 1.4 apache2环境
#### 1.4.1 安装
```bash
sudo apt-get install apache2 
```
#### 1.4.2 验证
```bash
sudo /etc/init.d/apache2 start
```

### 1.5 gerrit管理帐号
可选，使用独立账号配置gerrit。
```bash
sudo adduser gerrit
sudo passwd gerrit
```
将gerrit加入sudo权限
```bash
sudo visudo
gerrit  ALL=(ALL:ALL) ALL
```

## 2 安装与配置gerrit
### 2.1 配置gerrit
默认安装
```bash
java -jar gerrit-2.12.4.war init --batch -d ~/review_site
```

更新配置文件
```bash
sudo vim ~/review_site/etc/gerrit.config
```

内容如下：
```bash
[gerrit]
    basePath = git #默认gerrit对应的git库
    canonicalWebUrl = http://192.168.199.112:8081/ #gerrit web管理界面
[database]
    type = h2 #h2数据库
    database = db/ReviewDB #数据库路径
[index]
    type = LUCENE
[auth]
    type = HTTP #auth模式，默认为OPENID，配置为HTTP，需要apache配置反向代理
[receive]
    enableSignedPush = false
[sendemail]
    enable=false #关闭邮件提醒
[container]
    user = gerrit #linux user for gerrit
    javaHome = /opt/jdk1.7.0_79/jre #java home
[sshd]
    listenAddress = *:29418 #default gerrit port
[httpd]
    listenUrl = http://*:8081/
[cache]
    directory = cache
[http]
    proxy = http://192.168.199.112:8080 #proxy server
    proxyUsername = gerrit1 #proxy user & password
    proxyPassword = 123456
```

### 2.2 配置apache2反向代理
如果apache目录结构如下：
```bash
[username@hostname apache2]$ ls 
apache2.conf conf-enabled magic mods-enabled ports.conf sites-enabled conf-available envvars mods-available sites-available
```

开启SSL、Proxy、Rewrite等模块：
```bash
cd /etc/apache2/mods-enabled
ln -s ../mods-available/proxy.load
ln -s ../mods-available/proxy.conf
ln -s ../mods-available/proxy_http.load
ln -s ../mods-available/proxy_balancer.conf
ln -s ../mods-available/proxy_balancer.load
ln -s ../mods-available/rewrite.load
ln -s ../mods-available/ssl.conf
ln -s ../mods-available/ssl.load
ln -s ../mods-available/socache_shmcb.load #
ln -s ../mods-available/slotmem_shm.load #
```

更新配置文件：
```bash
sudo vim /etc/apache2/sites-enabled/gerrit-httpd.conf
```

内容如下：
```bash
ServerName 192.168.199.112 #your server ip
<VirtualHost *:8080>
    ProxyRequests Off
    ProxyVia Off
    ProxyPreserveHost On
    AllowEncodedSlashes On
    RewriteEngine On
    RewriteRule ^/(.*) http://192.168.199.112:8081/$1 [NE,P] #rewrite rule for proxy

    <Proxy *>
          Order deny,allow
          Allow from all
    </Proxy>

    <Location /login/>
        AuthType Basic
        AuthName "Gerrit Code Review"
        Require valid-user
        AuthBasicProvider file
        AuthUserFile /etc/apache2/passwords #password file for gerrit
    </Location>

    ProxyPass / http://192.168.199.112:8081/

</VirtualHost>
```

如果apache目录结构如下：
```bash
[username@hostname apache2]$ ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  lib  logs  man  manual  modules
```
开启SSL、Proxy、Rewrite等模块：
```bash
[username@hostname apache2]$ vi conf/http.conf
# Open LoadModule
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule ssl_module modules/mod_ssl.so
LoadModule rewrite_module modules/mod_rewrite.so
# Gerrit config
Include conf/extra/gerrit-httpd.conf 
```
其中 ```apache2/conf/extra/gerrit-httpd.conf``` 内容同上面的 ```apache2/sites-enabled/gerrit-httpd.conf```。

### 2.3 配置gerrit账户密码
```bash
touch /etc/apache2/passwords
# 管理员 admin
htpasswd -b /etc/apache2/passwords admin 123456
# 普通用户 gerrit1
htpasswd -b /etc/apache2/passwords gerrit1 123456

### 2.4 启动gerrit&启动apache2
```bash
sudo ~/review_site/bin/gerrit.sh start
sudo /etc/init.d/apache2 start
```

### 2.5 访问gerrit 管理界面 

http://192.168.199.112:8080/

第一次访问，需要输入第3步设置的 ```admin``` 及密码，该账户将作为 ```gerrit``` 管理员账户。进入后可设置 ```FullName: GerritAdmin```。

## 3 如何使用gerrit
**前提**：需要 git客户端 / gerrit服务端配合使用。

### 3.1 添加项目
**gerrit 服务端**

#### 3.1.1 使用gerrit添加新项目
**适用于开启新项目并使用 gerrit**
```bash
# 建议采用管理界面添加
ssh -p 29418 gerrit1@192.168.199.112 gerrit create-project --empty-commit --name demo-project
```
使用gerrit管理界面

#### 3.1.2 使用gerrit添加已有项目
**适用于已有项目下移植到 gerrit中**
```bash
# 建议采用管理界面添加
ssh -p 29418 gerrit1@192.168.199.112 gerrit create-project --name exist-project
```
使用gerrit管理界面

然后将已有项目与 ```gerrit``` 上建立的 ```exist-project``` 关联，即将已有代码库代码 ```push``` 到 ```gerrit``` 中进行管理。
```bash
cd ~/gitcode/exist-project
git push ssh://gerrit1@192.168.199.112:29418/exist-project *:*
```

### 3.2 生成sshkey
**git客户端**

在开发账户中生成sshkey，用作与gerrit服务器连接。
```bash
# 生成sshkey
ssh-keygen -t rsa
# 可查看sshkey
ls ~/.ssh/
# 查看sshkey
cat /.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCj1XDqjNXbn39oeacJOxe8FklBJRpGS1CcHRThWBytZ4A5KXAaxYzcD94GUd9UxxZzKCr6y90mwuJ+jkKxCTlqUiwj73QIiPWQ3Re08M049W4XxdfGnu/jyTI9DptWBsF0dwFJlQquUtitS+b1Tkz7Jr7+WipbZ22aiHwRvY4VcvCCdIHy/BnCCbVcfgk9u8f+X+ROm+DkOGfUcBNXWEJydqF0wy/D13Q5gp9OAXMIOD05T3GToJRwYtx2cVFmK4jE9HtcudOrrZNFVHqeblbA7EMKpIuDpLho7esmpwJ/woB1wnKTwHoUzbMt0a6hHPMNWyP2WIJebEA1KzThLixt gerrit@RylanYan-ThinkPad
```

### 3.3 添加sshkey到gerrit服务器
**gerrit 服务端**
此步骤与 ```git``` 流程类似，即将 ```id_rsa.pub``` 内容上传到 ```git repository``` ，```gerrit``` 中帮我们管理 ```git repository```。

### 3.4 拉取代码＆配置git hooks
**git客户端**

验证sshkey是否配置成功：
```bash
ssh gerrit1@192.168.199.112 -p 29418

The authenticity of host '[192.168.199.112]:29418 ([127.0.0.1]:29418)' can't be established.
  RSA key fingerprint is db:07:3d:c2:94:25:b5:8d:ac:bc:b5:9e:2f:95:5f:4a.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added '[192.168.199.112]:29418' (RSA) to the list of known hosts.


  ****    Welcome to Gerrit Code Review    ****

  Hi user, you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://gerrit1@192.168.199.112:29418/REPOSITORY_NAME.git
```

拉取代码： 
```bash
git clone ssh://gerrit1@192.168.199.112:29418/demo-project
```

更新githooks：
```bash
gitdir=$(git rev-parse --git-dir); scp -p -P 29418 gerrit1@192.168.199.112:hooks/commit-msg ${gitdir}/hooks/
```

**说明**：该过程用来在commit-msg中加入change-id，gerrit流程必备。

修改代码并提交，推送时与原有git流程不一致，采用 ```git push origin HEAD:refs/for/master```。
```bash
git push origin HEAD:refs/for/master
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 381 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: new: 1, refs: 1, done    
remote: 
remote: New Changes:
remote:   http://localhost:8081/4 Append date to testfile
remote: 
To ssh://gerrit1@localhost:29418/demo-project
 * [new branch]      HEAD -> refs/for/master
```

### 3.5 使用gerrit website完成code review
1. 当完成push后，可在gerrit管理界面看到当前提交code review的change。
2. 查看需要code review的提交。
3. 查看某次提交的详细信息（审核者+2可通过本次提交，提交者可通过Abandon本次提交）。
4. 如果审核者+2通过后，可提交该次commit。

### 3.6 gerrit注意事项
* 需要为每个使用者分配 ```gerrit``` 账号，不要都使用 ```admin``` 账号，因为 ```admin``` 账号可直接 ```push master```。
* pull代码后需要配置githooks文件，以便在commit时自动生成change-id，否则无法push。
* push代码时需要使用git push origin HEAD:refs/for/master(branch),gerrit默认关闭非admin账号的push direct权限。
* push代码时需要commit email与gerrit account email一致，否则无法push成功，可选择关闭email notify，并开启forge user权限，或者通过修改gerrit数据库account email信息
* gerrit数据库与gitlab同步，需要安装replication插件，并开启该功能 参考：http://www.cnblogs.com/tesky0125/p/5973642.html


## 4 参考链接
[Java SDK Download](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
[Gerrit Code Review](Releases Download: https://gerrit-releases.storage.googleapis.com/index.html)
[Gerrit Code Review](Quick get started guide: https://git.eclipse.org/r/Documentation/install-quick.html)
[Gerrit代码审核服务器搭建全过程](http://blog.csdn.net/ganshuyu/article/details/8978614)
[Gerrit代码审核流程](http://www.worldhello.net/gotgit/images/gerrit-workflow.png)
[Gerrit代码审核原理](http://www.worldhello.net/gotgit/05-git-server/055-gerrit.html)
[Gerrit代码审核权限管理](https://gerrit-review.googlesource.com/Documentation/access-control.html#category_forge_committer)
[Gerrit修改数据库email信息](http://www.cnblogs.com/kevingrace/p/5624122.html)
[Gerrit安装replication插件](https://gerrit-review.googlesource.com/Documentation/cmd-plugin-install.html)
[gerrit代码Review入门实战](http://geek.csdn.net/news/detail/85681)
