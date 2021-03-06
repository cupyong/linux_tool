## 安装：

```
# 如果没有 easy_install 需要安装 python-setuptools
$ yum install python-setuptools
$ easy_install supervisor

```

## 配置

```
# 设置默认配置
$ echo_supervisord_conf > /etc/supervisord.conf
$ vim /etc/supervisord.conf

```

## 将此项加入末尾，supervisor 默认启动加载下列配置

```
[include]
files = /www/supervisor/*.conf

```

## 创建文件夹

```
$ mkdir -p /www/supervisor/

```

## Laravel队列示列

```
$ vim /www/supervisor/laravel.conf

```

```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /www/website/base/artisan queue:work --queue=default,wechat --sleep=3 --tries=3 --timeout=110 --daemon
autostart=true
autorestart=true
user=nobody
numprocs=5
redirect_stderr=true
stdout_logfile=/www/website/base/worker.log

```

> --timeout 应该永远都要比 config/queue.php- retry_after 短至少几秒钟的时间。这样就能保证任务进程总能在失败重试前就被杀
> 死了。如果你的 --timeout 选项大于 retry_after 配置选项，你的任务可能被执行两次

## Elasticsearch示例

```
$ vim /www/supervisor/elastic.conf

```

```
[program:elastic]
command= sh /usr/elasticsearch-6.6.1/bin/elasticsearch ;  //启动脚本
autorestart=true ;
autostart=true ;
stderr_logfile=/www/elasticsearch/Elastic.err.log ;
stdout_logfile=/www/elasticsearch/Elastic.out.log ;
environment=JAVA_HOME=/usr/jdk1.8.0_201 ;
user=es ; //启动用户，elasticsearch默认不能以root用户启动
startsecs=1 ;//检测重启时间
~                  

```

## 操作

### 设置服务脚本

#### CentOS 6

```
$ vim /etc/init.d/supervisord


```

写入如下内容

```
#! /bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin:
PROGNAME=supervisord
DAEMON=/usr/bin/$PROGNAME
CONFIG=/etc/$PROGNAME.conf
PIDFILE=/tmp/$PROGNAME.pid
DESC="supervisord daemon"
SCRIPTNAME=/etc/init.d/$PROGNAME
# Gracefully exit if the package has been removed.
test -x $DAEMON || echo "$DAEMON is not exists" || exit 0

start()
{
        echo -n "Starting $DESC: $PROGNAME"
        $DAEMON -c $CONFIG
        echo "..."
}
stop()
{
        echo -n "Stopping $DESC: $PROGNAME"
        supervisor_pid=$(cat $PIDFILE)
        kill -15 $supervisor_pid
        echo "..."
}
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart)
        stop
        start
        ;;
  *)
        echo "Usage: $SCRIPTNAME {start|stop|restart}" >&2
        exit 1
        ;;
esac
exit 0


```

> 或者参考下面的文本
> <https://github.com/cedricporter/supervisor_conf/blob/master/init.d/supervisor>
> 需要修改 DAEMON SUPERVISORCTL 的路径
> 以及安装 start-stop-daemon

设置执行权限

```
$ chmod +x /etc/init.d/supervisord


```

#### CentOS 7

```
$ vim /usr/lib/systemd/system/supervisord.service


```

```
# supervisord service for sysstemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target


```

### 注册服务

```
$ chkconfig supervisord on
# 7.0
$ systemctl enable supervisord.service


```

### 启动

```
$ service supervisord start
# 7.0
$ systemctl start supervisord.service


```

### 停止

```
$ service supervisord stop
# 7.0
$ systemctl stop supervisord.service


```

### 重启

```
$ service supervisord restart
# 7.0
$ systemctl restart supervisord.service


```

### 查询状态

```
$ supervisorctl status


```

### 动态读取配置

如果添加了conf文件，可以使用以下命令启动

```
$ supervisorctl reread
$ supervisorctl update


```

针对指定任务，启动、重启

可以先使用`supervisorctl status`查看任务名称和序号

```
# 重启此任务的所有进程
$ supervisorctl restart laravel-worker:*
# 重启此任务的0号进程
$ supervisorctl restart laravel-worker:laravel-worker_00

# 启动/停止
$ supervisorctl start xxx:xxx
$ supervisorctl stop xxx:xxx

```

# 最后通过ip:端口访问web管理页面

```
http://127.0.0.1:9005/
```