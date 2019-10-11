# Redis 安装+设置开机启动

### 1.安装redis
```SH
$ wget http://download.redis.io/releases/redis-5.0.3.tar.gz
$ tar xzf redis-5.0.3.tar.gz
$ cd redis-5.0.3
$ make

```
### 2.设置开机启动
**脚本路径：/opt/redis/redis-5.0.3/utils**
#### ==配置文件路径==
**==EXEC=/usr/local/bin/redis-server==** 

**==CLIEXEC=/usr/local/bin/redis-cli==**

**==CONF="/etc/redis/${REDISPORT}.conf"==** 

```
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.
# chkconfig: 2345 90 10
# description: Redis is a persistent key-value database

### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```
### 3. 配置redis.conf

==1.将 bind 127.0.0.1 使用#注释掉，改为# bind 127.0.0.1（bind配置的是允许连接的ip，默认只允许本机连接；若远程连接需注释掉，或改为0.0.0.0）==

==2.将 protected-mode yes 改为 protected-mode no（3.2之后加入的新特性，目的是禁止公网访问redis cache，增强redis的安全性）==

==3.将 requirepass foobared 注释去掉，foobared为密码，也可修改为别的值（可选，建议设置）==

### 4.永久关闭防火墙
```
systemctl stop firewalld.service #停止firewall

systemctl disable firewalld.service #禁止firewall开机启动
```
### 5.设置为开机自启动服务器 
```
chkconfig redis on 
```
### 6.打开服务
```
service redis start
```
### 7.关闭服务
```
service redis stop
```
