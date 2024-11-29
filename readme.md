## kubeasz十分钟搭建集群
### step0:永久关闭防火墙
`sudo systemctl disable ufw`
### step1:SSH免密登陆
- 允许远程登陆root用户
  `vim /etc/ssh/sshd_config` 中的修改为 `PermitRootLogin yes`,更新服务`sudo systemctl reload ssh.service`
- 所有节点用root用户相互ssh免密登陆（包括和自己）
  `ssh-keygen -t rsa`
  `ssh-copy-id root@xxx`

### step2:下载kubeasz
准备就绪，开始用kubeasz安装k8s，直接第4点开始[https://github.com/easzlab/kubeasz/blob/9eb6232ddf354e8f11f1b42639a94b923a96c418/docs/setup/00-planning_and_overall_intro.md]
由于docker的问题，手动下载替换为国内镜像的ezdown(https://github.com/bogeit/LearnK8s/blob/main/kubeasz-3.6.5/ezdown)[https://github.com/bogeit/LearnK8s/blob/main/kubeasz-3.6.5/ezdown]
设置环境为`export release=3.6.5`，开始按照即可
