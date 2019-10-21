# /var/lib/docker/overlay2 占用很大，清理Docker占用的磁盘空间，
0. du -hs /var/lib/docker/
  命令查看磁盘使用情况。
1. docker system df
  命令，类似于Linux上的df命令，用于查看Docker的磁盘使用情况:
2. docker system prune
  命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。
3. docker system prune -a
  命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.。我没用过，因为会清理 没有开启的  Docker 镜像。


# 迁移 /var/lib/docker 目录
1. 停止docker服务。
systemctl stop docker
2. 创建新的docker目录，执行命令df -h,找一个大的磁盘。 我在 /home目录下面建了 /home/docker/lib目录，执行的命令是：
mkdir -p /home/docker/lib
3. 迁移/var/lib/docker目录下面的文件到 /home/docker/lib：
rsync -avz /var/lib/docker /home/docker/lib/
4. 配置?/etc/systemd/system/docker.service.d/devicemapper.conf。查看?devicemapper.conf 是否存在。如果不存在，就新建。
sudo mkdir -p /etc/systemd/system/docker.service.d/
sudo vi /etc/systemd/system/docker.service.d/devicemapper.conf
5. 然后在?devicemapper.conf 写入：（同步的时候把父文件夹一并同步过来，实际上的目录应在 /home/docker/lib/docker ）
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
6. 重新加载 docker
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
7. 为了确认一切顺利，运行
docker info
命令检查Docker 的根目录.它将被更改为 /home/docker/lib/docker
8. 启动成功后，再确认之前的镜像还在：
docker images