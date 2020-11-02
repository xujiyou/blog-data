# Redis 编译部署

Redis 版本 5.0.10



## Redis 安装

采用源码包编译安装。

安装步骤：

```bash
$ sudo tar zxvf redis-5.0.10.tar.gz
$ make PREFIX=/usr/local/redis install
$ sudo mkdir /usr/local/redis/etc/
$ sudo cp redis.conf /usr/local/redis/etc/
```

修改配置文件 `/usr/local/redis/etc/redis.conf`

```
bind 0.0.0.0
```

添加 `/etc/init.d/redis` 文件：

```bash
#!/bin/bash
#chkconfig: 2345 80 90
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

PATH=/usr/local/bin:/sbin:/usr/bin:/bin
REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
REDIS_CLI=/usr/local/redis/bin/redis-cli
   
PIDFILE=/var/run/redis.pid
CONF="/usr/local/redis/etc/redis.conf"
   
case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        if [ "$?"="0" ] 
        then
              echo "Redis is running..."
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $REDIS_CLI -p $REDISPORT SHUTDOWN
                while [ -x ${PIDFILE} ]
               do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
   restart|force-reload)
        ${0} stop
        ${0} start
        ;;
  *)
    echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2
        exit 1
esac
```

Redis 开机自启动设置：

```bash
$ sudo chmod +x /etc/init.d/redis
$ sudo chkconfig --list
$ sudo chkconfig --add redis
$ sudo chkconfig --level 2345 redis on
```

启动 Redis：

```bash
$ sudo service redis start
```

停止 Redis：

```bash
$ sudo service redis stop
```

查看状态：

```bash
$ sudo ps -el|grep redis
$ sudo netstat -an|grep 6379
$ sudo lsof -i:6379
```

