各个系统环境部署
==================================================

--------------------------------------
> ## 子系统部署

- SEngine依赖子系统数据库mongoDB,broker服务器Emqtt.所有子系统及SEngine由进程管理工具Supervisor统一管理
	
> ## 操作系统环境
- CentOS7
> ## 新建系统用户 yytd 统一管理SEngine,Emqtt.
```bash
useradd yytd -m 
```

> ## mongoDB
mongoDB-3.2
        
> ### 安装步骤

> #### 创建yum源
```bash
cd /etc/yum.repos.d/
vi mongodb-org-3.2.repo
```
mongodb-org-3.2.repo内容

```bash
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=0
enabled=1
```

> #### yum 安装mongoDB
```bash
sudo yum install -y mongodb-org
```

- Disable SELinux by setting the SELINUX setting to disabled in /etc/selinux/config.
```bash
SELINUX=disabled
```

- MongoDB服务开机自启
```bash
sudo chkconfig mongod on
```

参考:https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/

###Emqttd
安装包下载:http://emqtt.com/downloads

下载完成后解压至/yytd/
```bash
cd /home/yytd
unzip emqttd-centos64-v2.0-beta.3-20160918.zip
```

###Supervisor
1.安装python-pip python包管理工具
```bash
sudo yum install python-pip
```
2.安装supervisor
```bash
sudo pip install supervisor
yum -y install supervisor
```
3.配置supervisor为系统服务
```bash
vim /lib/systemd/system/supervisord.service
```
supervisord.service内容
```bash
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
SysVStartPriority=99

LimitCORE=infinity
LimitNOFILE= 65535
LimitNPROC= 65535

[Install]
WantedBy=multi-user.target
```

4.修改supervisord.conf
```bash
[include]
files = /etc/supervisor/*.conf
```
5.supeivisor配置各子系统
```bash
mkdir /etc/supervisor
cd /etc/supervisor
vi yytd.conf
```
yytd.conf内容
```bash
[program:emqttd]
directory = /yytd/emqttd/bin/
command = /yytd/emqttd/bin/emqttd console
priority = 9
autostart = true
startsecs = 3
autorestart = true
startretries = 3
user = yytd 
redirect_stderr = true
stdout_logfile_maxbytes = 100MB
stdout_logfile_backups = 20
stdout_logfile = /yytd/logs/emqttd/emqttd_stdout.log
environment = HOME=/home/yytd

[program:mongod]
directory = /usr/bin/
command = /usr/bin/mongod -f /etc/mongod.conf
priority = 9
autostart = true
startsecs = 3
autorestart = true
startretries = 3
user = mongod
redirect_stderr = true
stdout_logfile_maxbytes = 100MB
stdout_logfile_backups = 20
stdout_logfile = /yytd/logs/mongodb/mongodb_stdout.log

[program:nginx]
directory = /yytd/server/nginx/sbin/
command = /yytd/server/nginx/sbin/nginx
priority = 7
autostart = true
startsecs = 3
autorestart = true
startretries = 3
user = root
redirect_stderr = true
stdout_logfile_maxbytes = 100MB
stdout_logfile_backups = 20
stdout_logfile = /yytd/logs/nginx/nginx_stdout.log

[program:tomcat1]
directory = /yytd/server/tomcat1/bin/
command = /yytd/server/tomcat1/bin/catalina.sh run
priority = 6
autostart = true
startsecs = 5
autorestart = true
startretries = 3
user = yytd
redirect_stderr = true
stdout_logfile_maxbytes = 100MB
stdout_logfile_backups = 20
stdout_logfile = /yytd/logs/tomcat1/tomcat1_stdout.log
environment = JAVA_HOME=/yytd/jdk1.8

[program:sc]
directory = /yytd/sengine/ ; 程序的启动目录
command = /yytd/sengine/sengine  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
priority = 8
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 3        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
;autorestart = false   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = yytd          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /yytd/logs/sengine/sc_stdout.log

```
6.supeivisord服务开启
```bash
systemctl enable supervisord.service
systemctl start/restart/stop supervisord.service
```
7.supervisor开启子系统
```bash
supervisorctl start/stop/restart sc
supervisorctl start/stop/restart emqttd
```

