# 1.安装Docker
## 1.1Centos安装Docker
```shell
# 备份yum源
cd /etc/yum.repos.d ; mkdir backend; mv CentOS-Linux-* backend/
# centos7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 安装yum-config.yml.yml.yml-manager配置工具
yum -y install yum-utils
# 设置yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装docker-ce版本
yum install -y docker-ce
# 启动
systemctl start docker
# 开机自启
systemctl enable docker
# 查看版本号
docker --version
# Docker镜像源设置
# 修改文件 /etc/docker/daemon.json，没有这个文件就创建
# 添加以下内容后，重启docker服务：
cat >/etc/docker/daemon.json<<EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "registry-mirrors": ["https://mvrfhrpi.mirror.aliyuncs.com"]
 
}
EOF
# 加载
systemctl reload docker
# 查看
systemctl status docker containerd
# 安装docker—compose
yum install -y epel-release
yum install -y docker-compose
docker-compose --version
```



# docker network

## 参考资料

1.[基本用法]([docker network详解、教程-CSDN博客](https://blog.csdn.net/wangyue23com/article/details/111172076))

2.[Docker网络模式]([Docker四种网络模式 - 简书 (jianshu.com)](https://www.jianshu.com/p/22a7032bb7bd))

## 基本指令

```shell
# 查看network支持的参数
docker network --help
# 参数如下
* connect 将某个容器连接到一个docker网络
# 可以让某个未加入网络的已经运行的容器，加入指定容器
docker network connect mynet nginx
* disconnect 将某个容器退出某个局域网络
* create 创建一个docker局域网络
# 创建一个网络，不指定ip（可以指定ip）
docker network create mynet 
* inspect 显示某个局域网络信息
# inspect可以查看，网络的ip信息，已经加入到这个网络的容器信息
ls 显示所有docker局域网络
prune 删除所有未引用的docker局域网络
rm 删除docker网络
```

3.容器加入某个创建的网络

```shell
#运行redis容器
docker run -itd --name redis  --network mynet --network-alias redis -p 6379:6379 redis
#运行nginx容器
docker run -d --name nginx -p 80:80 --network mynet --network-alias nginx --privileged=true   -v /home/wwwroot:/home/wwwroot -v /home/wwwlogs:/home/wwwlogs  nginx
# --network mynet 表示该容器加入到mynet这个网络
# --network-alias redis表示这个容器网络的host名称，在这个网络里可以用host的方式进行网络连接
```



# docker-compose



# docker一些快捷指令

```she
# 停止所有容器
docker stop $(docker ps -q)
# 删除所有容器
docker rm $(docker ps -aq)
# 停用并删除
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
```

