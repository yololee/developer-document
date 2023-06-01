## 一、服务器准备

服务器发行版本

```shell
root@manager:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.3 LTS
Release:        18.04
Codename:       bionic
```

Docker 版本

```shell
root@manager:~# docker -v
Docker version 24.0.1, build 6802122
```

修改主机名

```shell
# 永久修改主机名
hostnamectl set-hostname 新主机名

# 查看主机名
hostname
```

| 服务器ip        | 主机名  |
| --------------- | ------- |
| 116.211.105.103 | manager |
| 116.211.105.107 | worker1 |
| 116.211.105.112 | worker2 |
| 116.211.105.117 | worker3 |

## 二、swarm集群搭建

### 1、开放2375端口

[docker开发2375端口](https://gitee.com/huanglei1111/docker-compose/blob/master/%E5%9F%BA%E7%A1%80/09-Docker%E9%85%8D%E7%BD%AE%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A52375%E7%AB%AF%E5%8F%A3.md)

### 2、swarm集群搭建

[swarm 集群搭建](https://gitee.com/huanglei1111/docker-compose/blob/master/%E5%9F%BA%E7%A1%80/22-docker%20swarm.md)

## 三、可视化界面-portainer

### 1、在每一个服务器上创建目录

```shell
mkdir /root/portainer
cd /root/portainer

mkdir -p portainer/data/volumes
mkdir -p portainer/data/portainer_data
```

### 2、创建docker-compose-portainer.yml文件

```yml
version: '3.3'

services:
  agent:
    image: portainer/agent
    labels:
      service: portainer_agent
    logging:
      options:
        labels: "service"
        max-size: "100m"
        max-file: "4"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer/data/volumes:/var/lib/docker/volumes
    environment:                               
      AGENT_CLUSTER_ADDR: tasks.agent
    networks:
      - portainer_net
    deploy:
      mode: global
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer
    labels:
      service: portainer
    logging:
      options:
        labels: "service"
        max-size: "100m"
        max-file: "4"
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    depends_on:
      - agent   
    ports:
      - "19000:9000"
    volumes:
      - ./portainer/data/portainer_data:/data
    networks:
      - portainer_net
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager] 

networks:
  portainer_net:
    driver: overlay
    attachable: true  
```

### 3、发布应用

```shell
docker stack deploy docker-compose-portainer.yml portainer
```

访问`ip(集群任一ip):19000`

第一次访问需要你设置密码

## 四、部署mysql主从集群

### 1、在每一个服务器上创建目录

```shell
mkdir /root/mysql
cd /root/mysql

mkdir -p data/mysql-n1/db
mkdir -p data/mysql-n2/db
mkdir -p data/mysql-n3/db
mkdir -p data/mysql-n1/logs
mkdir -p data/mysql-n2/logs
mkdir -p data/mysql-n3/logs
mkdir  config
```

把`mysqlId.conf`文件放入`config`文件夹中。`mysqlId.conf`内容如下

```conf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
#log-error      = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
#sql-mode=NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
sql-mode=NO_ENGINE_SUBSTITUTION
#user=mysql
#default-storage-engine=INNODB
character-set-server=utf8mb4
default-time_zone = '+8:00'
max_connections=100000

server_id=1
log_bin=mysql-bin
## 需要主从复制的数据库 根据自己的需求定义
binlog-do-db=nacos_config
binlog-do-db=digital

## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql

## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M

## 主从复制的格式（mixed,statement,row，默认格式是statement。建议是设置为row，主从复制时数据更加能够统一）
binlog_format=row

## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062

skip-name-resolve

[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```

### 2、创建一个swarm的全局网络

```shell
docker network create -d overlay mysqlnet
```

### 3、创建`docker-compose-mysql.yml`文件

```yml
version: '3.7'
services:
  mysql-n1:
    image: mysql:5.7
    hostname: mysql-n1
    ports:
      - 3306:3306
    user: root
    environment:
      - MYSQL_ROOT_HOST=%
      - MYSQL_ROOT_PASSWORD=a123456
      - TZ=Asia/Shanghai
    volumes:
      # 和docker-compose 不一样的是要提前创建，否则会报错
      - ./data/mysql-n1/db:/var/lib/mysql
      - ./data/mysql-n1/logs:/var/log/mysql
      - ./config:/etc/mysql/conf.d
    command:
      - "--server-id=1"
      - "--default-authentication-plugin=mysql_native_password"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_general_ci"
      - "--explicit_defaults_for_timestamp=true"
      - "--lower_case_table_names=1"
      - "--binlog-ignore-db=mysql"
      - "--binlog_format=row"
      - "--log-bin=mysql-bin"
      - "--sync_binlog=1"
      - "--expire_logs_days=7"
      - "--slave_skip_errors=1062"
    networks:
      - mysqlnet
    deploy:
      placement:
        constraints:
          - node.hostname == worker1
          - node.role == worker
  mysql-n2:
    image: mysql:5.7
    hostname: mysql-n2
    user: root
    environment:
      - MYSQL_ROOT_HOST=%
      - MYSQL_ROOT_PASSWORD=a123456
      - TZ=Asia/Shanghai
    volumes:
      - ./data/mysql-n2/db:/var/lib/mysql
      - ./data/mysql-n2/logs:/var/log/mysql
      - ./config:/etc/mysql/conf.d
    command:
      - "--server-id=2"
      - "--default-authentication-plugin=mysql_native_password"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_general_ci"
      - "--explicit_defaults_for_timestamp=true"
      - "--lower_case_table_names=1"
      - "--binlog-ignore-db=mysql"
      - "--binlog_format=row"
      - "--log-bin=mysql-bin"
      - "--sync_binlog=1"
      - "--expire_logs_days=7"
      - "--slave_skip_errors=1062"
    depends_on:
      - mysql-n1
    networks:
      - mysqlnet
    deploy:
      placement:
        constraints:
          - node.role == worker
          - node.hostname == worker2
  mysql-n3:
    image: mysql:5.7
    hostname: mysql-n3
    user: root
    environment:
      - MYSQL_ROOT_HOST=%
      - MYSQL_ROOT_PASSWORD=a123456
      - TZ=Asia/Shanghai
    volumes:
      - ./data/mysql-n3/db:/var/lib/mysql
      - ./data/mysql-n3/logs:/var/log/mysql
      - ./config:/etc/mysql/conf.d
    command:
      - "--server-id=3"
      - "--default-authentication-plugin=mysql_native_password"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_general_ci"
      - "--explicit_defaults_for_timestamp=true"
      - "--lower_case_table_names=1"
      - "--binlog-ignore-db=mysql"
      - "--binlog_format=row"
      - "--log-bin=mysql-bin"
      - "--sync_binlog=1"
      - "--expire_logs_days=7"
      - "--slave_skip_errors=1062"
    depends_on:
      - mysql-n1
      - mysql-n2
    networks:
      - mysqlnet
    deploy:
      placement:
        constraints:
          - node.role == worker
          - node.hostname == worker3

networks:
  mysqlnet:
    #false-统自动创建网桥名,格式为: 目录名_网桥名，默认为false; true-使用外部创建的网桥，需需要自己手动创建
    external: true
```

发布应用

```shell
docker stack deploy docker-compose-mysql.yml mysql
```

## 五、部署Mysql/Mariadb高可用主从复制集群

### 1、创建swarm全局网络

```shell
# 创建网络
docker network create --driver overlay --attachable mydbnet
```

### 2、创建一个mysql cluster集群

设置副本为1，–replicas=1，当副本为1时 mariadb-cluster镜像为这个实例自动变成引导节点

```shell
docker service create --name dbcluster --network mydbnet --replicas=1  --env DB_SERVICE_NAME=dbcluster --env MYSQL_ROOT_PASSWORD=123456 --env MYSQL_DATABASE=mydb --env MYSQL_USER=mydbuser --env MYSQL_PASSWORD=123456 toughiq/mariadb-cluster
```

扩展dbcluster服务的任务数量，即为数据库集群增加2个节点

```shell
docker service scale dbcluster=3
```

### 3、maxscale 部署

创建MaxScale代理服务并连接到dbcluster

由于Swarm提供了一个负载平衡器，因此使用该Docker Swarm启用的数据库集群不需要MaxScale Proxy服务。因此，可以使用负载均衡器DNS名称连接到集群,上面运行的例子就是mysqldbcluster。它在同一个名字，由启动时提供–name。
 但是MaxScale提供了一些关于负载平衡数据库流量的附加功能。它是获取有关群集状态的信息的简单方法

```shell
docker service create --name maxscale --network mydbnet --env DB_SERVICE_NAME=dbcluster --env ENABLE_ROOT_USER=1 --publish 3306:3306 toughiq/maxscale
```

> 要通过MaxScale禁用root对数据库的访问，只需设置--env ENABLE_ROOT_USER=0或删除该行即可。
> 默认情况下禁用根访问

成功之后，可以到maxscale运行的主机上查看mysql的集群情况

```shell
docker exec -it maxscale名称/容器id  maxadmin -pmariadb list servers
```

![image-20230601152755293](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230601152755293.png)

## 六、部署redis哨兵模式

### 1、在每一个服务器上创建目录

```shell
mkdir -p /root/redis-sentinel
cd /root/redis-sentinel

mkdir -p redis-master-slave-sentinel/redis/master/data
mkdir -p redis-master-slave-sentinel/redis/slave-1/data
mkdir -p redis-master-slave-sentinel/redis/slave-2/data

mkdir -p redis-master-slave-sentinel/redis/master/config
mkdir -p redis-master-slave-sentinel/redis/slave-1/config
mkdir -p redis-master-slave-sentinel/redis/slave-2/config

mkdir -p redis-master-slave-sentinel/sentinel
```

### 2、redis配置文件

`redis.conf`文件

```conf
# 注释允许外部访问redis
bind 0.0.0.0
# 开启保护模式后，需要 bind ip 或 设置密码
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
logfile ""
databases 16
always-show-logo yes
# 900秒内，如果超过1个key被修改，则发起快照保存
save 900 1
# 300秒内，如果超过10个key被修改，则发起快照保存
save 300 10
# 60秒内，如果1万个key被修改，则发起快照保存
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb
rdb-del-sync-files no
dir ./
replica-serve-stale-data yes

replica-read-only yes

repl-diskless-sync no
repl-diskless-sync-delay 5
repl-diskless-load disabled
repl-disable-tcp-nodelay no
replica-priority 100
acllog-max-len 128

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
oom-score-adj no
oom-score-adj-values 0 200 800
appendonly no
appendfilename "appendonly.aof"
# 每次操作都会立即写入aof文件中
# appendfsync always
# 每秒持久化一次(默认配置)
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
jemalloc-bg-thread yes
```

### 3、哨兵配置文件

`redis-sentinel-1.conf`

```conf
# 配置可参考 http://download.redis.io/redis-stable/sentinel.conf
# 配置说明 https://redis.io/topics/sentinel

# 哨兵sentinel实例运行的端口 默认26379
port 26379
protected-mode no

# 哨兵sentinel的工作目录 
dir /tmp

# 哨兵sentinel监控的redis主节点的 ip port 
# master-name 可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。 
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了 
# sentinel monitor <master-name> <ip> <redis-port> <quorum> sentinel monitor mymaster 127.0.0.1 6379 2 
sentinel monitor mymaster 116.211.105.107 6379 2


# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒 
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000


# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步
#这个数字越小，完成failover所需的时间就越长，
# 但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。 
#可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。 
sentinel parallel-syncs mymaster 1

# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供 密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码 
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456

# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。 
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那 里同步数据时。 
#3.当想要取消一个正在进行的failover所需要的时间。 
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时， slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了 
# 默认三分钟 # sentinel failover-timeout <master-name> 
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

`redis-sentinel-2.conf`

```conf
port 26380
protected-mode no
dir /tmp
sentinel monitor mymaster  116.211.105.107  6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster 123456
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

`redis-sentinel-3.conf`

```conf
port 26381
protected-mode no
dir /tmp
sentinel monitor mymaster 116.211.105.107  6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster 123456
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

### 4、创建docker-compose-redis-sentinel.yml文件

```yml
version: '3'

networks:
  redis:
    external: true

services:
  # ============================ ↓↓↓↓↓↓ redis ↓↓↓↓↓↓ ============================

  # 主
  redis-master:
    image: redis:6.0.8
    command: redis-server /etc/redis/redis.conf --port 6379 --requirepass 123456 --masterauth 123456 --appendonly no
    environment:                       
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    volumes:                           
      - "./redis-master-slave-sentinel/redis/master/data:/data"
      - "./redis-master-slave-sentinel/redis/master/config/redis.conf:/etc/redis/redis.conf"
    ports:
      - "6379:6379"
    networks:
      - redis
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker1     
  # 从1
  redis-slave-1:
    image: redis:6.0.8                                      
    command: redis-server /etc/redis/redis.conf --port 6380 --requirepass 123456 --appendonly no --slaveof redis-master 6379 --masterauth 123456
    environment:             
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    volumes:                          
      - "./redis-master-slave-sentinel/redis/slave-1/data:/data"
      - "./redis-master-slave-sentinel/redis/slave-1/config/redis.conf:/etc/redis/redis.conf" 
    ports:    
      - "6380:6380"
    networks:
      - redis
    depends_on:
      - redis-master
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker2 
  # 从2
  redis-slave-2:
    image: redis:6.0.8                                                         
    command: redis-server /etc/redis/redis.conf --port 6381 --requirepass 123456 --appendonly no --slaveof redis-master 6379 --masterauth 123456
    environment:                      
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    volumes:                          
      - "./redis-master-slave-sentinel/redis/slave-2/data:/data"
      - "./redis-master-slave-sentinel/redis/slave-2/config/redis.conf:/etc/redis/redis.conf" 
    ports:                           
      - "6381:6381"
    networks:
      - redis
    depends_on:
      - redis-master
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker3   

  # ============================ ↓↓↓↓↓↓ sentinel ↓↓↓↓↓↓ ============================

  redis-sentinel-1:
    image: redis:6.0.8                  
    command: redis-sentinel /etc/redis/sentinel.conf
    environment:
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    ports:
      - "26379:26379"
    volumes:
      - "./redis-master-slave-sentinel/sentinel/redis-sentinel-1.conf:/etc/redis/sentinel.conf"
    networks:
      - redis
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker1   
  redis-sentinel-2:
    image: redis:6.0.8                    
    command: redis-sentinel /etc/redis/sentinel.conf
    environment:         
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    ports:
      - "26380:26380"
    volumes:
      - "./redis-master-slave-sentinel/sentinel/redis-sentinel-2.conf:/etc/redis/sentinel.conf"
    networks:
      - redis
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
      - redis-sentinel-1
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker2   
  redis-sentinel-3:
    image: redis:6.0.8
    command: redis-sentinel /etc/redis/sentinel.conf
    environment:
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
    ports:
      - "26381:26381"
    volumes:
      - "./redis-master-slave-sentinel/sentinel/redis-sentinel-3.conf:/etc/redis/sentinel.conf"
    networks:
      - redis
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
      - redis-sentinel-1
      - redis-sentinel-2
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker3   
```

### 5、发布

```shell
docker stack deploy -c docker-compose-redis-sentinel.yml redis
```

## 七、部署rabbitmq集群

### 1、在管理节点创建目录

```shell
mkdir -p /root/rabbitmq-cluster
cd /root/rabbitmq-cluster
```

### 2、管理节点的服务器上拉取文件

```shell
git clone https://github.com/liangxiaobo/RabbitMQ-Docker-cluster.git
```

或者在浏览器查看gitee中的部署文件https://gitee.com/huanglei1111/docker-compose/tree/master/Linux/rabbitmq-cluster

### 3、创建网络

```shell
docker network create --driver overlay --attachable rabbitmq-cluster
```

### 4、编写docker-compose

```shell
cd RabbitMQ-Docker-cluster/

vim docker-compose-rabbitmq.yml
```

```yml
version: "3.3"
services:
  rabbit1:
    image: rabbitmq:3-management
    hostname: rabbit1
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret string"
      RABBITMQ_NODENAME: rabbit1
    depends_on:
      - rabbit3
    ports:
      - "4369:4369"
      - "5671:5671"
      - "5672:5672"
      - "15671:15671"
      - "15672:15672"
      - "25672:25672"
    configs:
      - source: rabbitmq_config
        target: /etc/rabbitmq/rabbitmq.config
      - source: definitons_json
        target: /etc/rabbitmq/definitions.json
    networks:
      - rabbitmq-cluster
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker1   
  rabbit2:
    image: rabbitmq:3-management
    hostname: rabbit2
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret string"
      RABBITMQ_NODENAME: rabbit2
    depends_on:
      - rabbit3
    configs:
      - source: rabbitmq_config
        target: /etc/rabbitmq/rabbitmq.config
      - source: definitons_json
        target: /etc/rabbitmq/definitions.json
    networks:
      - rabbitmq-cluster
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker
          - node.hostname == worker2  
  rabbit3:
    image: rabbitmq:3-management
    hostname: rabbit3
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret string"
      RABBITMQ_NODENAME: rabbit3
    configs:
      - source: rabbitmq_config
        target: /etc/rabbitmq/rabbitmq.config
      - source: definitons_json
        target: /etc/rabbitmq/definitions.json
    networks:
      - rabbitmq-cluster
    deploy:
      replicas: 1
      placement:
        constraints: 
          - node.role == worker 
          - node.hostname == worker3 

configs:
  rabbitmq_config:
    file: ./rabbitmq.config
  definitons_json:
    file: ./definitions.json

networks:
  rabbitmq-cluster:
    external: true
```

使用了docker-compose编排3.3版本的`docker config`就不需要在每台mq的机器上放置两个配置文件

### 5、发布

```shell
docker stack deploy -c docker-compose-rabbitmq.yml rabbitmq
```

成功后访问 [http://集群的任一:15672](https://gitee.com/link?target=http%3A%2F%2F%E9%9B%86%E7%BE%A4%E7%9A%84%E4%BB%BB%E4%B8%80%3A15672) 默认用户名和密码都是guest

## 八、部署digital

### 1、创建网络，用于前后端通信

```shell
docker network create --driver overlay --attachable digital_net
```

### 2、部署后端

因为前端通过nginx转发，转发路径不支持下划线`_`，而docker stack 发布的时候服务名称自动拼接位下划线连接，所以前后端分开部署。

这里后端需要跟mysql，redis，前端通信，所以加上相应的网络。

--mount 相当于挂载的意思，前面是宿主机路径，后面是容器路径，这里需要我们在各个服务器创建宿主机路径

```shell
docker service create --name digital-be  --network mysqlnet --network digital_net --network redis --replicas=3 --constraint 'node.role==worker' --mount type=bind,src=./digital-be,dst=/config img.chinamye.com/tianma-blocks/digital-be:dev-0.0.2
```

### 3、部署前端

```shell
version: '3'
services:
  fe:
    image: img.chinamye.com/tianma-blocks/mye-site:sandbox-1.0.1
    environment:
      TZ: Asia/Shanghai
      NODE_ENV: production
    volumes:
      - "./digital-fe/config:/etc/nginx/http.d"
    networks:
      - "digital_net"
    ports:
      - "3000:3000"
    deploy:
      replicas: 3 #可以指定该服务运行的容器数量
      resources:
        limits:
          cpus: "1"
          memory: 1024M
      restart_policy:
        condition: on-failure
      placement: #指定约束和偏好设置，这里指定该服务在manager节点启动
        constraints: [node.role == worker]
networks:
 digital_net:
  external: true
```

发布

```shell
docker stack deploy -c docker-compose-digital-fe.yml digital
```

## 九、swarm常见命令

[swarm常见命令参考](https://gitee.com/huanglei1111/docker-compose/blob/master/%E5%9F%BA%E7%A1%80/22-docker%20swarm.md)

