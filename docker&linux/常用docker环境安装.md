# 常用docker环境安装

## 1. 安装docker

```cmd
# 卸载原来老版本的docker
sudo apt-get remove docker docker-engine docker.io containerd runc
# 更新源
sudo apt-get update
# 下载依赖
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 添加docker GPG钥匙
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 添加源
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# 更新源
sudo apt-get update
# 查看历史版本
apt-cache madison docker-ce
# 下载对应版本
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io 或者 sudo apt-get -y install docker-ce
```

配置镜像加速器

```json
vim /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```

```cmd
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### 常用启动资源

#### 1. 启动Spug

```dockerfile
FROM registry.aliyuncs.com/openspug/spug
#给spug配置maven和jdk的环境变量
ENV MAVEN_HOME=/usr/local/maven/apache-maven-3.8.4
ENV JAVA_HOME=/usr/local/jdk8
ENV PATH $PATH:$MAVEN_HOME/bin:$JAVA_HOME/bin
#创建maven和jdk路径
RUN mkdir -p /usr/local/maven/apache-maven-3.8.4 && mkdir -p /usr/local/jdk8

CMD ["java", "-version"]
CMD ["mvn", "-v"]
```

```bash
#!/bin/bash
eval "docker run 
		-d 
		--restart=always 
		--name=spug 
		-p 80:80 
		-v /spug/:/data 
		-v /var/run/docker.sock:/var/run/docker.sock # 使用宿主机中的docker
		-v /usr/bin/docker:/usr/bin/docker 
		-v /usr/local/maven/apache-maven-3.8.4:/usr/local/maven/apache-maven-3.8.4 #挂载mavne的路径到宿主机当中，以及jdk
		-v /usr/local/jdk8:/usr/local/jdk8 spug"
```



## 2. 安装docker-compose

```cmd
# 下载文件
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.6.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 移动到目录下
chmod +x /usr/local/bin/docker-compose
```

- [国内镜像源](http://get.daocloud.io/#install-compose)

- [命令文档](https://www.runoob.com/docker/docker-dockerfile.html)
- [docker-compose命令详解](https://blog.51cto.com/u_4925054/2342021)

### 常用启动资源

#### 1. 启动redis

```yml
version: '3'
services:
  redis:
    image: redis:6.2
    container_name: redis
    command: redis-server --requirepass root #连接容器时需要设置密码
    ports:
      - "6379:6379"  #设置端口
    volumes: #映射配置文件路径
      - "/usr/local/soft/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf"
    restart: always #设置始终重启
```

配置文件

```properties
# 绑定到指定的 IP 地址。如果需要监听所有网络接口，请使用 0.0.0.0
bind 127.0.0.1

# 端口号
port 6379

# 设置 Redis 的工作目录，数据文件和日志文件将存储在这里
dir /data

# 设置数据库的数量，默认是 16 个数据库
databases 16

# 启用保护模式，仅允许本地连接，如果需要远程连接，请禁用此选项
protected-mode yes

# 设置密码，用于连接到 Redis 服务器
requirepass 123456

# 启用 RDB 持久化，默认是 yes
save 900 1
save 300 10
save 60 10000

# 禁用 RDB 持久化（仅用于测试或特殊情况）
# save ""

# 指定 RDB 持久化文件名
dbfilename dump.rdb

# 指定日志文件
logfile "/var/log/redis/redis-server.log"

# 设置服务器的最大客户端连接数
maxclients 10000

# 启用 AOF 持久化
appendonly yes

# 指定 AOF 持久化文件名
appendfilename "appendonly.aof"

# AOF 持久化每秒 fsync 选项，默认为 "everysec"
# appendfsync everysec

# 设置 AOF 持久化策略为每个写命令都进行 fsync
# appendfsync always

# AOF 文件重写触发条件，默认为 "auto"
# auto-aof-rewrite-min-size 64mb
# auto-aof-rewrite-percentage 100

# 最大 AOF 文件尺寸，默认为 512mb
# auto-aof-rewrite-min-size 64mb
```



#### 2. 启动redis哨兵

创建两个文件夹 **redis** 和 **sentinel** 文件夹用于存放docker-compose.yml文件

```yaml
version: "3"
services:
  master:
    image: redis:latest
    container_name: my_redis_master
    command: redis-server --requirepass root  # 在连接容器时需要密码
    ports:
      - 6379:6379
  slave1:
    image: redis:latest
    depends_on:  # 这里目的是需要先启动master，随后再启动slave节点
      - master
    container_name: my_redis_slave1
    command: redis-server --slaveof my_redis_master 6379 --requirepass root --masterauth root # 再容器启动后，通过这里命令来指定主节点ip地址
    ports:
      - 6380:6379
  slave2:
    image: redis:latest
    depends_on:
      - master
    container_name: my_redis_slave2
    ports:
     - 6381:6379
    command: redis-server --slaveof my_redis_master 6379 --requirepass root --masterauth root
networks:   # 这里是配置网络环境，目的是让容器之间能够相互连接，如果不配置，哨兵将获取不到从节点的信息，并且无法转换master节点
  default:
    external:
      name: redis_net
```



```yaml
version: '3'
services:
  sentinel1:
    image: redis
    container_name: redis-sentinel-1
    networks:
      - redis_net
    ports:
      - 26379:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf  # 启动哨兵，并且指定配置文件
    volumes:
      - ./sentinel1.conf:/usr/local/etc/redis/sentinel.conf

  sentinel2:
    image: redis
    container_name: redis-sentinel-2
    networks:
      - redis_net
    ports:
      - 26380:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./sentinel2.conf:/usr/local/etc/redis/sentinel.conf

  sentinel3:
    image: redis
    container_name: redis-sentinel-3
    networks:
      - redis_net
    ports:
      - 26381:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./sentinel3.conf:/usr/local/etc/redis/sentinel.conf
networks:  # 这里指定网络跟redis在同一个网络环境下
  redis_net:
    external:
      name: redis_net
```

```properties
sentinel配置文件，将下面的配置文件 cp成三分就行
port 26379
dir /tmp

#指定集群的名字以及集群中master的ip地址，这里是容器地址，如果是用的虚拟机地址我这边会导致master转换不了。2就是指哨兵有多少个哨兵认为失效master就失效，保证超过50%就行
sentinel monitor mymaster 192.168.16.2 6379 2 

# 重点：我就是在这里忘了配置master需要密码登陆 导致一直获取不到master节点下的从信息
sentinel auth-pass mymaster root

#这里是指定master在失效之后多久就认为失效
sentinel down-after-milliseconds mymaster 30000

# failover进行主备切换时最多可以有多少个slave对新的master进行同步，值越小完成failover的时间就越长，设置为1就是每次只有一个slave处于不能处理命令的状态
sentinel parallel-syncs mymaster 1 
sentinel failover-timeout mymaster 180000 
sentinel deny-scripts-reconfig yes
```



#### 3. 启动kafka

```yaml
version: '3'
services:
  zookeeper:
    image: docker.io/bitnami/zookeeper:3.8
    restart: always
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    networks:
      - zk_kafka
  kafka: #创建kafka
    image: docker.io/bitnami/kafka:3.2
    container_name: kafka
    hostname: kafka
    ports:
      - "9092:9092"
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
    networks:
      - zk_kafka
# 创建kafka和zk之间在同一个网络中
networks:
  zk_kafka:
```

