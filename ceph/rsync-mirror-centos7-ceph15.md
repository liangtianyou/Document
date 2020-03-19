# 从清华源同步ceph

**ceph版本**：15.1.1

**操作系统**：CentOS 7.7

```bash
mkdir -p ceph/el7/x86_64/ ceph/el7/noarch/

rsync -avz --exclude "*debuginfo*" --delete --progress rsync://rsync.mirrors.ustc.edu.cn/ceph/rpm-15.1.1/el7/x86_64/ ceph/el7/x86_64/
rsync -avz --exclude "*debuginfo*" --delete --progress rsync://rsync.mirrors.ustc.edu.cn/ceph/rpm-15.1.1/el7/noarch/ ceph/el7/noarch/
```
