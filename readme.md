## kubeasz十分钟裸机搭建k8s集群
**以下所有的操作都在root用户下进行**
`sudo passwd root`先建root
### step0:永久关闭防火墙
`sudo systemctl disable ufw`
### step1:SSH免密登陆
- 允许远程登陆root用户
  `vim /etc/ssh/sshd_config` 中的修改为 `PermitRootLogin yes`,更新服务`sudo systemctl reload ssh.service`
- master用root用户和其他机器ssh免密登陆（包括和自己）
  `ssh-keygen -t rsa`
  `ssh-copy-id root@xxx`

### step2:下载kubeasz
准备就绪，开始用kubeasz安装k8s，直接第4点开始[https://github.com/easzlab/kubeasz/blob/9eb6232ddf354e8f11f1b42639a94b923a96c418/docs/setup/00-planning_and_overall_intro.md]
做一些替换：
- 由于docker的问题，不要下载原来的ezdown,手动下载替换为国内镜像的(ezdown)[https://github.com/bogeit/LearnK8s/blob/main/kubeasz-3.6.5/ezdown]
- 设置环境为`export release=3.6.5`，开始按照即可
- 网络使用`flannel`需下载[https://github.com/easzlab/kubeasz/blob/master/docs/setup/network-plugin/flannel.md] ,默认calico的话没法跨节点访问服务

注意：cloud ECS机器将/etc/kubeasz/clusters/k8s-01/hosts设置为私网ip(192.168.x)而不是公网ip

遇错误清理集群并reboot[https://github.com/easzlab/kubeasz/blob/master/docs/setup/quickStart.md]
k8s 1.3x版本之后其kube-proxy已经变成了进程，在之前是以pod运行在kube-system命名空间下。kube-proxy起到负载均衡和**服务发现**的作用。此外，由于我们的节点经过了一层跳板机，如果在本地游览器访问的话，需要在vscode里面进行端口的转发

## 安装监控系统
主要安装的有：node-exporter（监控节点），cadvisor（监控容器），prometheus时序数据库和查询接口（实测不出问题）

文件在monitor目录下，先创建monitor命名空间，按照一定的顺序安装即可，镜像全部换成了阿里云，在prometheus_configmap注意配置一些监控的端口，已换成静态

## docker镜像配置
- `sudo mkdir /etc/systemd/system/docker.service.d`
- `echo -e "[Service]\nEnvironment=\"HTTP_PROXY=127.0.0.1:7890\"\nEnvironment=\"HTTPS_PROXY=127.0.0.1:7890\"" > /etc/systemd/system/docker.service.d/proxy.conf`
- `sudo systemctl daemon-reload && systemctl restart docker`

containerd参考[https://developer.baidu.com/article/details/2807572]

## clash配置
配置海外代理，可以访问到dockerhub
首先上传clash文件到服务器，然后`sudo chmod 777 clash && sudo mv clash /usr/bin/`,然后执行`clash`,会在用户目录下生成`.config/clash/`，然后将自己购买的账号的Country.mmdb文件和config.yaml文件上传到这个目录下，执行clash就可以 

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
- 传递镜像`scp path/to/<imagename>.tar user@ip:path`
- 在从节点首先找到`find / -name ctr`然后软链接到可执行文件`ln /opt/kube/bin/containerd-bin/ctr /usr/bin/ctr`，然后执行`ctr -n k8s.io image import <imagename>.tar`加载镜像

## 部署微服务online-boutique
直接把 [https://github.com/yuxiaoba/Hipster-Shop/blob/master/release/kubernetes-manifests.yaml] 复制下来，然后将 frontend-external的service的type改成`NodePort`,直接部署即可

vscode端口转发出来，就可以在游览器访问了(转发机器ip:端口，不加机器ip还是访问不到)

## monitor
cadvisor，node-exporter,prometheus 均可部署成功，注意prometheus中的几个job_name的静态ip配置

## chaosblade
helm install chaosblade /root/chaosblade-operator-amd64-1.8.0.tgz --namespace chaosblade
cli直接下载chaosblade-1.8.0-linux_amd64压缩包并解压就可以直接使用

## istio
https://istio.io/latest/docs/ambient/getting-started/

kubectl apply -f /istio-1.27.3/samples/addons/prometheus.yaml
automatic collect node,container,istio-request

#### key step: enable istio in namespace
for sisdecar mode:
 - kubectl label namespace boutique istio-injection=enabled --overwrite
https://istio.io/latest/docs/setup/getting-started/

   
for ambient mode:
 - kubectl label namespace boutique istio.io/dataplane-mode=ambient --overwrite
https://istio.io/latest/docs/ambient/getting-started/secure-and-visualize/
