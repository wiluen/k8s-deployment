## kubeasz十分钟搭建集群
### step1:SSH免密登陆
- 允许远程登陆root用户
```vim /etc/ssh/sshd_config
# Authentication:
#LoginGraceTime 2m#PermitRootLogin prohibit-password
PermitRootLogin yes```
