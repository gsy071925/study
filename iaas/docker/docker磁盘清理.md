# /var/lib/docker/overlay2 ռ�úܴ�����Dockerռ�õĴ��̿ռ䣬
0. du -hs /var/lib/docker/
  ����鿴����ʹ�������
1. docker system df
  ���������Linux�ϵ�df������ڲ鿴Docker�Ĵ���ʹ�����:
2. docker system prune
  �����������������̣�ɾ���رյ����������õ����ݾ�����磬�Լ�dangling����(����tag�ľ���)��
3. docker system prune -a
  ��������ø��ӳ��ף����Խ�û������ʹ��Docker����ɾ����ע�⣬����������������ʱ�رյ��������Լ���ʱû���õ���Docker����ɾ���ˡ�����ʹ��֮ǰһ��Ҫ�����.����û�ù�����Ϊ������ û�п�����  Docker ����


# Ǩ�� /var/lib/docker Ŀ¼
1. ֹͣdocker����
systemctl stop docker
2. �����µ�dockerĿ¼��ִ������df -h,��һ����Ĵ��̡� ���� /homeĿ¼���潨�� /home/docker/libĿ¼��ִ�е������ǣ�
mkdir -p /home/docker/lib
3. Ǩ��/var/lib/dockerĿ¼������ļ��� /home/docker/lib��
rsync -avz /var/lib/docker /home/docker/lib/
4. ����?/etc/systemd/system/docker.service.d/devicemapper.conf���鿴?devicemapper.conf �Ƿ���ڡ���������ڣ����½���
sudo mkdir -p /etc/systemd/system/docker.service.d/
sudo vi /etc/systemd/system/docker.service.d/devicemapper.conf
5. Ȼ����?devicemapper.conf д�룺��ͬ����ʱ��Ѹ��ļ���һ��ͬ��������ʵ���ϵ�Ŀ¼Ӧ�� /home/docker/lib/docker ��
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
6. ���¼��� docker
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
7. Ϊ��ȷ��һ��˳��������
docker info
������Docker �ĸ�Ŀ¼.����������Ϊ /home/docker/lib/docker
8. �����ɹ�����ȷ��֮ǰ�ľ����ڣ�
docker images