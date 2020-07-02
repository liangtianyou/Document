# 邮件服务器搭建

## 安装postfix dovecot
yum -y install postfix dovecot

## 配置postfix
/etc/postfix/main.cf

76行
```bash
myhostname = EServer1.perabytes.com
EServer1 为主机名
perabytes.com 为域名
```
83行
```bash
mydomain = perabytes.com
```
113行，放开注释
```bash
inet_interfaces = all
```
116行，注释
```bash
#inet_interfaces = localhost
```
165行，添加$mydomain
```bash
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```
**注意**: 一定要加入"$mydomain"，否则postfix可能从网络上查找域名，发送到错误的地址。

264行，修改为对应的网段
```bash
mynetworks = 10.0.0.0/8, 127.0.0.0/8
```
419行，设置收件箱地址
```bash
home_mailbox = Maildir/
```
## 配置dovecot
### 修改/etc/dovecot/dovecot.conf
24行，放开注释
```bash
protocols = imap pop3 lmtp
```
48行，设置认证信任IP
```bash
login_trusted_networks = 10.0.0.0/8
```
### 修改/etc/dovecot/conf.d/10-auth.conf
10行，取消注释
```bash
disable_plaintext_auth = yes
```
100行，增加login
```bash
auth_mechanisms = plain login
```
### 修改/etc/dovecot/conf.d/10-mail.conf
24行，放开注释
```bash
mail_location = maildir:~/Maildir
```
### 修改/etc/dovecot/conf.d/10-master.conf

## 重启服务
```bash
systemctl enable postfix
systemctl restart postfix
systemctl enable dovecot
systemctl restart dovecot
```

## 创建用户
```bash
useradd admin
passwd admin
```

### 禁止终端登录
```bash
usermod -s /sbin/nologin admin
```

### 禁止ssh登录
vim /etc/ssh/sshd_config
在文件最后添加只允许root登录
```bash
AllowUsers root
```

## 测试postfix
```bash
telnet EServer2.perabytes.com smtp
```

输出如下:
```bash
Trying 10.10.1.22...
Connected to EServer2.perabytes.com.
Escape character is '^]'.
220 EServer2.perabytes.com ESMTP Postfix
ehlo EServer2.perabytes.com     ## 输入该行 ##
220 EServer2.perabytes.com ESMTP Postfix
ehlo EServer2.peraabytes.com
250-EServer2.perabytes.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
mail from:<admin>     ## 输入该行 - 发件地址 ##
250 2.1.0 Ok
rcpt to:<admin>     ## 输入该行 - 收件地址 ##
250 2.1.5 Ok
data     ## 输入 data 然后输入邮件正文 ##
354 End data with <CR><LF>.<CR><LF>
welcome to EServer2.perabytes.com     ## 输入邮件正文 ##
.     ## 输入点 (.) 结束输入 ##
250 2.0.0 Ok: queued as E52FC181CA75B
quit     ## 输入 quit 退出 ##
221 2.0.0 Bye
Connection closed by foreign host.
```
## 测试dovecot
```basj
telnet EServer2.perabytes.com pop3
```
输出如下:
```bash
Trying 10.10.1.22...
Connected to EServer2.perabytes.com.
Escape character is '^]'.
+OK [XCLIENT] Dovecot ready.
user admin     ## 输入发件人 ##
+OK
pass XXXX     ## 输入密码 ##
+OK Logged in.
retr 1     ## 输入该命令接收邮件 ##
+OK 331 octets
Return-Path: <admin@EServer2.perabytes.com>
X-Original-To: admin
Delivered-To: admin@EServer2.perabytes.com
Received: from EServer2.peraabytes.com (unknown [10.10.10.15])
	by EServer2.perabytes.com (Postfix) with ESMTP id E52FC181CA75B
	for <admin>; Tue, 10 Apr 2018 16:22:23 +0800 (CST)

welcome to EServer2.perabytes.com
.
quit     ## 输入 'quit' 退出 ##
+OK Logging out.
Connection closed by foreign host.
```