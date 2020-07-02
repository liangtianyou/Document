# Gerrit WEB登录用户及邮件用户

## 创建WEB登录用户
密码文件位于/data/gerrit/etc/passwd，使用htpasswd创建WEB登录用户。
第一个用户添加时使用-c参数创建文件，后续添加请勿使用-c参数，否则会清空原内容
```bash
username='liangtianyou'
userpass='123456'
gerritpassfile='/data/gerrit/etc/passwd'
if [ -f $gerritpassfile ]; then
    htpasswd -b $gerritpassfile $username $userpass
else
    htpasswd -c $gerritpassfile $username $userpass
fi
```

## 删除用户
```bash
htpasswd -D $gerritpassfile  $username
```

## 修改密码
先删除用户，再创建用户
```bash
htpasswd -D $gerritpassfile  $username
htpasswd $gerritpassfile $username $userpass
```

## 创建邮件用户
邮件用户实际就是系统用户，创建时不允许其登录操作系统。
```bash
username='liangtianyou'
userpass='123456'
useradd $username -s /sbin/nologin
( echo $userpass ; echo $userpass) | passwd $username
```

