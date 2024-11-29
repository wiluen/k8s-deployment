## kubeasz十分钟搭建集群
### step1:SSH免密登陆
- 允许远程登陆root用户
  `vim /etc/ssh/sshd_config` 中的修改为 PermitRootLogin yes,更新服务`sudo systemctl reload ssh.service`
