# golang supervisor

#### go准备工作

创建一个<demo>项目，里面main.go代码
代码摘自<Go Web 编程>

```
package main

import (
    "fmt"
    "log"
    "net/http"
    "strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()       //解析参数，默认是不会解析的
    fmt.Println(r.Form) //这些信息是输出到服务器端的打印信息
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}

func main() {
    http.HandleFunc("/", sayhelloName)       //设置访问的路由
    err := http.ListenAndServe(":9090", nil) //设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}


```

go build
go install
本地测试通过之后把运行的demo包传至服务器

#### 安装supervisor

官网地址
[http://supervisord.org/index.html](https://link.jianshu.com?t=http://supervisord.org/index.html)

```
sudo yum install python-setuptools
sudo easy_install supervisor

```

安装成功后 生成配置文件

```
sudo echo_supervisord_conf > /etc/supervisord.conf

```

这里很简单,如遇问题可以自己Google下

#### 添加自己的配置文件

我也学网上在/etc/下面新建一个专门放 .conf 的文件夹,感觉这样很好,比一味修改supervisord.conf文件要更方便以后管理
我这命名
"supervisorconffile"
在supervisorconffile中新建个.conf文件
我的
demo.conf

```
[program:demo]
user=root
command=/root/Applications/Go/bin/demo
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/root/Applications/LogFile/log/demo.log 
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/root/Applications/LogFile/err/demo.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
stopsignal=INT
[supervisord] 

```

说明

```
command：表示运行的命令，我这是填写的我demo安装包的原则路径。
autostart：表示是否跟随supervisor一起启动。
autorestart：如果该程序挂了，是否重新启动。
stdout_logfile：终端标准输出重定向文件。
stderr_logfile：终端错误输出重定向文件。

```

注意上面的两个log文件
/root/Applications/LogFile/log/demo.log
/root/Applications/LogFile/err/demo.log
都要在相应目录下面创建对应的log

#### 修改配置文件

好了开始
编辑/etc/supervisord.conf

在文件最下面
刚打开是这样的

```
;[include]
;files = relative/directory/*.ini

```

改成这样

```
[include]
files = /etc/supervisorconffile/*.conf

```

注意[include]前面的';'要去掉,我在这点上耽误了点时间

#### 启动

```
sudo /usr/bin/supervisord -c /etc/supervisord.conf

```

报错了
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.

网上查到执行下面两条命令可以解决

```
1.
find / -name supervisor.sock
2.
unlink /***/supervisor.sock

```

再次执行
sudo /usr/bin/supervisord -c /etc/supervisord.conf
启动成功
我这成功什么都没有打印输出反馈,没有反馈就是成功了没毛病了

可以输入命令查看下刚刚启动的demo服务是否是成功

```
sudo supervisorctl status demo

```

输出

```
demo                             STARTING 

```

查看状态 输入名录

```
supervisorctl

```

输出:

```
demo                             RUNNING   pid 4806, uptime 0:04:05

```

supervisor> help
用help查看帮助
输入exit退出: