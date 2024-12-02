## kubeasz十分钟裸机搭建k8s集群
**以下所有的操作都在root用户下进行**
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
做一些替换：
- 由于docker的问题，不要下载原来的ezdown,手动下载替换为国内镜像的(ezdown)[https://github.com/bogeit/LearnK8s/blob/main/kubeasz-3.6.5/ezdown]
- 设置环境为`export release=3.6.5`，开始按照即可

注意：k8s 1.3x版本之后其kube-proxy已经变成了进程，在之前是以pod运行在kube-system命名空间下。kube-proxy起到负载均衡和**服务发现**的作用。此外，由于我们的节点经过了一层跳板机，如果在本地游览器访问的话，需要在vscode里面进行端口的转发

## 安装监控系统
主要安装的有：node-exporter（监控节点），cadvisor（监控容器），prometheus时序数据库和查询接口（实测不出问题）

文件在monitor目录下，先创建monitor命名空间，按照一定的顺序安装即可，镜像全部换成了阿里云，在prometheus_configmap注意配置一些监控的端口，已换成静态

## 安装helm
[https://helm.sh/docs/intro/install/]
## 安装miniconda
参考[https://blog.csdn.net/weixin_43651674/article/details/134880766]
然后将conda配置进环境变量`export PATH="/root/miniconda3/bin:$PATH"`然后`source ~/.bashrc`
## docker镜像操作
由于限制，无法从docker拉取镜像，
1. 第一种方法是每个节点配置clash代理，直接拉docker hub
2. 第二种方法是手动拉下来并上传到阿里云镜像[https://cr.console.aliyun.com/cn-hangzhou/instance/dashboard]
3. 第三种方法是主节点的镜像可以传递到其他的节点,但由于主节点是docker，而从节点的容器运行时是containerd
- 主节点执行 `docker save -o <imagename>.tar <full_imagesname:version>`
- 在从节点首先找到`find / -name ctr`然后软链接到可执行文件`ln /opt/kube/bin/containerd-bin/ctr /usr/bin/ctr`，然后执行`ctr -n k8s.io image import <imagename>.tar`加载镜像


