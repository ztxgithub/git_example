各个系统环境部署
==================================================

--------------------------------------
> ## 子系统部署
```bash
	SEngine依赖子系统数据库mongoDB,broker服务器Emqtt.所有子系统及SEngine由进程管理工具Supervisor统一管理
```	
> ### 操作系统环境
```bash
	CentOS7
```
> ### 新建系统用户 yytd 统一管理SEngine,Emqtt.
```bash

	useradd yytd -m 

```

> ## mongoDB
> ### mongoDB版本号
```bash

	mongoDB-3.2
   
```
        
> ### 安装步骤

> #### 创建yum源
```bash

	cd /etc/yum.repos.d/
	vi mongodb-org-3.2.repo

```
- mongodb-org-3.2.repo内容

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


> ## Emqttd
- [Emqttd安装包下载](http://emqtt.com/downloads)

- 下载完成后解压至/yytd/
```bash

	cd /home/yytd
	unzip emqttd-centos64-v2.0-beta.3-20160918.zip

```

> ## SEngine 
> ### 创建sengine运行环境

> #### 创建sengine运行目录
```bash

	mkdir -p /home/yytd/sengine/
	
```

> #### 创建sengine运行时配置文件
```bash

    vi /home/yytd/sengine/sengine.config
	
	<?xml version="1.0" encoding="utf-8" ?>
    <Config>
	<BROKERIP>127.0.0.1:1883</BROKERIP>
	<SENGINEID>Sengine_test</SENGINEID>
	<SCLOUDID>Scloud_test</SCLOUDID>
	<DBURI>mongodb://test:YYtd_test@192.168.0.4:27017/scloud-test</DBURI>
	<DATABASENAME>scloud-test</DATABASENAME>
	<POLLING_UNIT_INTERVAL>60</POLLING_UNIT_INTERVAL>
	<LOGOUT_OVERTIME_THRESHOLD>180</LOGOUT_OVERTIME_THRESHOLD>
	<MQTT_KEEP_ALIVE_INTERVAL>60</MQTT_KEEP_ALIVE_INTERVAL>
	<MIN_LOG_LEVEL>ELOG_LVL_VERBOSE</MIN_LOG_LEVEL>
    </Config>

```
> #### 准备sengine运行时需要的动态链接库
- 动态库列表
```bash

        libpaho-mqtt3a.so
        libboost_serialization.so
        libbsoncxx.so
        libmongocxx.so
        libgtest.a

```
- 将上述动态链接库拷到 /usr/local/lib/ 目录下
- 设置好相关动态链接库
```bash
    1.vi /etc/ld.so.conf
	内容为： /usr/local/lib/
	
	2.ldconfig
	
```

> ## Supervisor
> ### 下载并安装supervisor
- [Supervisor安装包下载](https://pypi.python.org/pypi/supervisor)
- 安装supervisor
```bash

	tar -xvf supervisor-x.x.x.tar.gz
	cd supervisor-x.x.x/
	sudo python setup.py install   
	
```

> ### 配置supervisor为系统服务

> ### 生成supervisord.conf
```bash

	echo_supervisord_conf > /etc/supervisord.conf

```

> ### 修改supervisord.conf
```bash

	[include]
	files = /etc/supervisor/*.conf

```
> ### supeivisor配置各子系统

> #### supeivisor配置emqttd
```bash
	mkdir -p /home/yytd/logs/emqttd/
	
	mkdir /etc/supervisor
	cd /etc/supervisor
	vi yytd.conf

```
  
```bash
	yytd.conf内容:
	 
	[program:emqttd]
	directory = /home/yytd/emqttd/bin/
	command = /home/yytd/emqttd/bin/emqttd console
	priority = 9
	autostart = true
	startsecs = 3
	autorestart = true
	startretries = 3
	user = yytd 
	redirect_stderr = true
	stdout_logfile_maxbytes = 100MB
	stdout_logfile_backups = 20
	stdout_logfile = /home/yytd/logs/emqttd/emqttd_stdout.log
	environment = HOME=/home/yytd
	

```

> #### supeivisor配置sengine
```bash
	mkdir -p /home/yytd/logs/sengine/
	
	mkdir /etc/supervisor
	cd /etc/supervisor
	vi yytd.conf

```
  
```bash
	yytd.conf内容:
	 
	[program:sengine]
	directory = /home/yytd/sengine/ ; 程序的启动目录
	command = /home/yytd/sengine/sengine  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
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
	stdout_logfile = /home/yytd/logs/sengine/sc_stdout.log
	

```

> #### supeivisord服务开启
```bash
	
	systemctl enable supervisord.service
	systemctl start/restart/stop supervisord.service

```
> #### supervisor开启子系统
```bash
	
	supervisorctl start/stop/restart sc
	supervisorctl start/stop/restart emqttd

```

> #### 设置supervisord开机自启
- sudo vi /etc/rc.d/init.d/supervisord 
```bash

	#!/bin/sh
	#
	# /etc/rc.d/init.d/supervisord
	#
	# Supervisor is a client/server system that
	# allows its users to monitor and control a
	# number of processes on UNIX-like operating
	# systems.
	#
	# chkconfig: - 64 36
	# description: Supervisor Server
	# processname: supervisord
	
	# Source init functions
	. /etc/rc.d/init.d/functions
	
	prog="supervisord"
	
	prefix="/usr/"
	exec_prefix="${prefix}"
	prog_bin="${exec_prefix}/bin/supervisord"
	PIDFILE="/var/run/$prog.pid"
	
	start()
	{
		echo -n $"Starting $prog: "
		daemon $prog_bin --pidfile $PIDFILE -c /etc/supervisord.conf
		[ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
		echo
	}
	
	stop()
	{
		echo -n $"Shutting down $prog: "
		[ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
		echo
	}
	
	case "$1" in
	
	start)
	start
	;;
	
	stop)
	stop
	;;
	
	status)
		status $prog
	;;
	
	restart)
	stop
	start
	;;
	
	*)
	echo "Usage: $0 {start|stop|restart|status}"
	;;
	
	esac
	
```
(在Windows上编辑这内容有出现^M回车符，linux脚本识别不了该命令，可以用sed -i -e 's/\r//g' /etc/init.d/supervisord 命令去掉^M回车符)

- 执行以下命令
```bash

sudo chmod +x /etc/rc.d/init.d/supervisord
sudo chkconfig --add supervisord
sudo chkconfig supervisord on
sudo service supervisord start

```
