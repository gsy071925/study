supervisor
# ��װ
## 
yum install supervisor
supervisord -c /etc/supervisord.conf

## /etc/supervisord.conf
[include]
files = /etc/supervisor.d/*.conf

## supervisor��������
�½�һ��vim {path}/supervisord.service�ļ�
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

# supervisorctl����
supervisorctl help
supervisordctl start {program} ��������
supervisordctl stop {program} �رս���
supervisordctl restart {program} ��������
supervisordctl status �鿴��������״̬
supervisordctl update �������������ļ�

# ini�������
----------------------------------------------------------------------------------
[program:theprogramname]
command=/bin/cat              ; ���������������ʹ�þ���·����
process_name=%(program_name)s ; �������ƣ����õı����� `group_name`, `host_node_name`, `process_num`, `program_name`, `here`�������ļ�Ŀ¼���� һ�������Ҫ���ж�������������ʹ�á�����������ӡ�
numprocs=1                    ; �������еĸ���������Ĭ��Ϊ1�����ֵ����1����`process_name` ������� `%(process_num)s`
numprocs_start=0              ; `%(process_num)s`��ʼ���֣�Ĭ��Ϊ0
directory=/tmp                ; �������е�����Ŀ¼���൱����cd��ָ��Ŀ¼��Ȼ�����г���
umask=022                     ; umask for process (default None)
priority=999                  ; ��������ĵ����ȼ���������start all/stop all,�����ȼ��ĳ�����ȹرպ�������
autostart=true                ; ��supervisord����ʱ�Զ�������Ĭ��Ϊtrue
startsecs=1                   ; ��������ǰ�ȴ�ʱ��ȴ�ʱ�䡣Ĭ��Ϊ1��
startretries=3                ; ����������������Ĭ��Ϊ3��
autorestart=unexpected        ; �Ƿ��Զ���������ѡ����Ϊ false, unexpected, true�����Ϊfalse���Զ����������Ϊunexpected��ʾ��������˳��źŲ��� `exitcodes` �У����Զ�������Ĭ��Ϊunexpected
exitcodes=0,2                 ; �����˳��롣���`autorestart`ʹ�á�Ĭ��Ϊ 0,2
stopsignal=QUIT               ; ɱ�������Ƿ��͵��źţ�Ĭ��ΪTREM��
stopwaitsecs=10               ; ����SIGKILL�ź�ǰ���ȴ�ʱ�䡣Ĭ��Ϊ10��
user                          ; ��ָ���û������������Ĭ��Ϊ��ǰ�û���
stopasgroup=false             ; �Ƿ����ӽ��̷���ֹͣ�źţ������Flask��debugģʽ�����ô����������Ϊtrue�������ӽ��̷���ֹͣ�źš�Ĭ��Ϊfalse
killasgroup=false             ; �Ƿ����ӽ��̷���kill�źţ�Ĭ��Ϊfalse
redirect_stderr=false         ; ������������򵽱�׼�����Ĭ��Ϊfalse
stdout_logfile=/a/path        ; ��׼�����־·������ѡ����Ϊ `�Զ���` `AUTO` `NONE`��`�Զ���`����־д���Զ���·�������õı�����`group_name`, `host_node_name`, `process_num`, `program_name`, `here`�������ļ�Ŀ¼����`NONE`��������־��`AUTO` ��supervisord�Զ�ѡ��·�������ҵ�supervisord������������ʱԭ���Զ���������־�Լ���־�ı����ļ��ᱻɾ����Ĭ��ΪAUTO
stdout_logfile_maxbytes=1MB   ; ��׼�����־�����ļ�����С���������ָ����С�Ὣ��־�ļ����ݣ����õĵ�λ KB MB GB���������Ϊ0���ʾ�������ļ���С��Ĭ��Ϊ50MB
stdout_logfile_backups=10     ; ��׼�����־�ļ���󱸷�����Ĭ��Ϊ10
stdout_capture_maxbytes=1MB   ; �����̴��ڡ�stdout capture mode��ģʽ��д�뵽FIFO��������ֽ��������õ�λ KB MB GB��Ĭ��Ϊ0����ϸ˵����[capture-mode](http://supervisord.org/logging.html#capture-mode)
stdout_events_enabled=false   ; 
                              ;�������������ô����������־�������������׼���������ͬ��
stderr_logfile=/a/path        ;
stderr_logfile_maxbytes=1MB   ;
stderr_logfile_backups=10     ;
stderr_capture_maxbytes=1MB   ;
stderr_events_enabled=false   ;
environment=A="1",B="2"       ; �����������ã����õı����� `group_name`, `host_node_name`, `process_num`, `program_name`, `here`�� Ĭ��Ϊ�ա�
serverurl=AUTO                ; override serverurl computation (childutils)
----------------------------------------------------------------------------------

## ʾ��
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