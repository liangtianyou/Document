## 1. 使用方法
```bash
python gen_pass.py <用户名称全拼>
```
## 2 脚本内容
```python
#-*- coding:utf-8 -*-
import os
import sys
import random
import traceback
import subprocess

randlist = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','1','2','3','4','5','6','7','8','9','0']
randlen = len(randlist) - 1
passlen = 8

gerritpassconf= "/data/gerrit/etc/passwd"
gerritpassfirstcmd = "htpasswd -bc %s %s %s"
gerritpasscmd = "htpasswd -b %s %s %s"
addusercmd = "useradd %s -s /sbin/nologin"
userpasscmd = "( echo %s ; echo %s) | passwd %s"
#-------------------
# 生成密码
#-------------------
def cust_popen(cmd):
    proc = None
    retcode = None
    try:
        proc = subprocess.Popen(cmd,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE,close_fds=True)
        retcode = proc.wait()
    except Exception,e:
        print >> sys.stderr,traceback.print_exc()
    return retcode,proc
#-------------------
# 生成密码
#-------------------
def gen_pass():
    outlist = []
    random.shuffle(randlist)
    for i in range(passlen):
	    outlist.append(randlist[random.randint(0,randlen)])

    return ''.join(outlist)
#-------------------
# 创建用户
#-------------------
def set_user(username,password):
    if not os.path.exists(gerritpassconf):
        retcode,proc = cust_popen(gerritpassfirstcmd % (gerritpassconf,username,password))
    else:
        retcode,proc = cust_popen(gerritpasscmd % (gerritpassconf,username,password))
    retcode,proc = cust_popen(addusercmd % username)
    retcode,proc = cust_popen(userpasscmd % (password,password,username))
    print username,password
    
if __name__ == '__main__':
    if len(sys.argv) == 1:
        print "Usage:python %s username" % sys.argv[0]
    else:
        username = sys.argv[1].strip()
        password = gen_pass()
        set_user(username,password)
```

## 3 示例
```bash
python gen_pass.py liangtianyou
```

