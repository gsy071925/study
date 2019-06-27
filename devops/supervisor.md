supervisor
# 安装
## 
yum install supervisor
supervisord -c /etc/supervisord.conf

## /etc/supervisord.conf
[include]
files = /etc/supervisor.d/*.conf

## supervisor开机启动
新建一个vim {path}/supervisord.service文件
----------------------------------------------------------------------------------
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
----------------------------------------------------------------------------------

mv {path}/supervisord.service /usr/lib/systemd/system/supervisord.service

systemctl enable supervisord

systemctl is-enabled supervisord

# supervisorctl命令
supervisorctl help
supervisordctl start {program} 启动进程
supervisordctl stop {program} 关闭进程
supervisordctl restart {program} 重启进程
supervisordctl status 查看进程运行状态
supervisordctl update 重新载入配置文件

# ini配置详解
----------------------------------------------------------------------------------
[program:theprogramname]
command=/bin/cat              ; 程序运行命令，建议使用绝对路径。
process_name=%(program_name)s ; 程序名称，可用的变量有 `group_name`, `host_node_name`, `process_num`, `program_name`, `here`（配置文件目录）。 一般程序需要运行多个副本的情况会使用。后面会有例子。
numprocs=1                    ; 程序运行的副本个数，默认为1，如果值大于1，则`process_name` 必须包含 `%(process_num)s`
numprocs_start=0              ; `%(process_num)s`起始数字，默认为0
directory=/tmp                ; 程序运行的所在目录，相当于先cd到指定目录，然后运行程序。
umask=022                     ; umask for process (default None)
priority=999                  ; 程序操作的的优先级，例如在start all/stop all,高优先级的程序会先关闭和重启。
autostart=true                ; 在supervisord启动时自动启动，默认为true
startsecs=1                   ; 程序启动前等待时间等待时间。默认为1。
startretries=3                ; 尝试重启最大次数。默认为3。
autorestart=unexpected        ; 是否自动重启，可选参数为 false, unexpected, true。如果为false则不自动重启，如果为unexpected表示如果程序退出信号不在 `exitcodes` 中，则自动重启。默认为unexpected
exitcodes=0,2                 ; 程序退出码。配合`autorestart`使用。默认为 0,2
stopsignal=QUIT               ; 杀死进程是发送的信号，默认为TREM。
stopwaitsecs=10               ; 发送SIGKILL信号前最大等待时间。默认为10。
user                          ; 以指定用户身份启动程序。默认为当前用户。
stopasgroup=false             ; 是否向子进程发送停止信号，这对于Flask的debug模式很有用处，如果设置为true，则不向子进程发送停止信号。默认为false
killasgroup=false             ; 是否向子进程发送kill信号，默认为false
redirect_stderr=false         ; 将错误输出定向到标准输出，默认为false
stdout_logfile=/a/path        ; 标准输出日志路径，可选参数为 `自定义` `AUTO` `NONE`，`自定义`将日志写到自定义路径，可用的变量有`group_name`, `host_node_name`, `process_num`, `program_name`, `here`（配置文件目录）；`NONE`不创建日志；`AUTO` 又supervisord自动选择路径，并且当supervisord服务重新启动时原来自动创建的日志以及日志的备份文件会被删除。默认为AUTO
stdout_logfile_maxbytes=1MB   ; 标准输出日志单个文件最大大小，如果超过指定大小会将日志文件备份，可用的单位 KB MB GB。如果设置为0则表示不限制文件大小。默认为50MB
stdout_logfile_backups=10     ; 标准输出日志文件最大备份数。默认为10
stdout_capture_maxbytes=1MB   ; 当进程处于“stdout capture mode”模式下写入到FIFO队列最大字节数，可用单位 KB MB GB。默认为0，详细说明见[capture-mode](http://supervisord.org/logging.html#capture-mode)
stdout_events_enabled=false   ; 
                              ;以下配置项配置错误输出的日志参数。和上面标准输出配置相同。
stderr_logfile=/a/path        ;
stderr_logfile_maxbytes=1MB   ;
stderr_logfile_backups=10     ;
stderr_capture_maxbytes=1MB   ;
stderr_events_enabled=false   ;
environment=A="1",B="2"       ; 环境变量设置，可用的变量有 `group_name`, `host_node_name`, `process_num`, `program_name`, `here`。 默认为空。
serverurl=AUTO                ; override serverurl computation (childutils)
----------------------------------------------------------------------------------

## 示例
cat /etc/supervisord.d/nginx.ini 
----------------------------------------------------------------------------------
[program:nginx]
command=/data/program/openresty/nginx/sbin/nginx -c /data/program/openresty/nginx/sbin/nginx/conf/nginx.conf
autorestart=true
killasgroup=true
stopasgroup=true
redirect_stderr=true
logfile_maxbytes=10MB
stdout_logfile=/data/logs/nginx/nginx.log
stderr_logfile=/data/logs/nginx/nginx.err
----------------------------------------------------------------------------------

cat /etc/supervisord.d/news.ini 
----------------------------------------------------------------------------------
[program:news]
user=www
directory=/data/program/tomcat/bin
command=/data/program/tomcat/bin/gsy-news_8001.sh run
autorestart=true
killasgroup=true
stopasgroup=true
redirect_stderr=true
logfile_maxbytes=10MB
stdout_logfile=/data/logs/tomcat/gsy-news.log
stderr_logfile=/data/logs/tomcat/gsy-news.err
----------------------------------------------------------------------------------