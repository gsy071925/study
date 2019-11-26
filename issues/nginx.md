# 业务web系统将数据post到服务端进行业务，部分（数据小）可用，部分（数据大）不可用，F12查询网络交互，返回500错误
## 排查
1. 业务tomcat日志无错误
2. 查看nginx错误日志，发现返回500错误用例的，nginx打印如下日志
2019/11/15 10:30:39 [crit] 4600#0: *8784200 open() "/var/lib/nginx/tmp/client_body/0008015986" failed (13: Permission denied), client: 10.111.97.10, server: scheduler-test.niudatastats.nubia.com, request: "POST /api/hive/sql/parse HTTP/1.0", host: "scheduler-test.niudatastats.nubia.com", referrer: "http://scheduler-test.niudatastats.nubia.com/"

查看目录权限
ll /var/lib/
drwxrwx---  3 nginx     root      4096 Nov 13 09:47 nginx

ll /var/lib/nginx/tmp/
drwx------ 2 www root 4096 Nov 13 09:50 client_body

查看进程启动用户
ps aux|grep nginx
root       833  0.0  0.0  83280  6224 ?        Ss   Nov13   0:00 nginx: master process nginx
www       4599  0.4  0.0  87312 10096 ?        S    Nov13  14:04 nginx: worker process
www       4600  1.7  0.0  87312 10912 ?        S    Nov13  52:13 nginx: worker process
www       4601  0.8  0.0  87312 10112 ?        S    Nov13  23:33 nginx: worker process
www       4602  2.6  0.0  88080 11680 ?        S    Nov13  75:39 nginx: worker process

## 根因
关于client_body_temp目录的作用，简单说就是如果客户端POST一个比较大的请求，长度超过了nginx缓冲区的大小，需要把这个文件的部分或者全部内容暂存到client_body_temp目录下的临时文件。
本问题正好触发该场景，触发写nginx临时文件，对目前
/var/lib/nginx/tmp/client_body/0008015986" failed (13: Permission denied)
www 用户对 /var/lib/nginx 无权限

## 解决办法
sudo chown -R www:root /var/lib/nginx/