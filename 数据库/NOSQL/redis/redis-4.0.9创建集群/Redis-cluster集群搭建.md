# Redis-cluster-4.0.9集群搭建.md
> 挺无奈的现实, redis官方没有提供redis-trib的镜像, 只能通过下载官方的源码包解压获取redis-trib.rb文件来制作镜像
>
> 在redis的镜像制作redis-trib, 让redis-trib镜像具有redis-cli的功能
> 
> 挺无奈的现实, 官方redis的Dockerfile和ruby的Dockerfile文件很复杂

##### 制作带redis-cli功能的redis-trib镜像的Dockerfile文件
```
FROM ruby:2.5-alpine3.7
RUN echo "http://mirrors.aliyun.com/alpine/v3.7/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.7/community" >> /etc/apk/repositories
RUN apk add --no-cache --virtual .build-deps curl tar coreutils gcc linux-headers make musl-dev; \
    gem install redis:4.0.1; \
    curl http://download.redis.io/releases/redis-4.0.9.tar.gz -s -o /redis-4.0.9.tar.gz; \
    tar -zxvf /redis-4.0.9.tar.gz -C / ; \
    make -C /redis-4.0.9 -j "$(nproc)"; \
    make -C /redis-4.0.9 install; \
    cp /redis-4.0.9/src/redis-trib.rb /usr/bin; \
    rm -rf /redis-4.0.9; \
    rm -rf /redis-4.0.9.tar.gz; \
    chmod +x /usr/bin/redis-trib.rb; \
    apk del --virtual .build-deps curl tar coreutils gcc linux-headers make musl-dev

CMD ["sh"]
```

```
docker build -t redis-trib:4.0.9 .
```

##### 制作redis-trib镜像的Dockerfile文件(该镜像只有redis-trib的功能)
```
FROM ruby:2.5-alpine3.7
RUN echo "http://mirrors.aliyun.com/alpine/v3.7/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.7/community" >> /etc/apk/repositories
# gem安装制定版本软件的命令  gem install redis:v 4.0.1
RUN gem install redis:4.0.1; \
    apk add --no-cache curl tar; \
    curl http://download.redis.io/releases/redis-4.0.9.tar.gz -s -o /redis-4.0.9.tar.gz; \
    tar -zxvf /redis-4.0.9.tar.gz -C / ; \
    cp /redis-4.0.9/src/redis-trib.rb /usr/bin; \
    chmod +x /usr/bin/redis-trib.rb; \
    rm -rf /redis-4.0.9; \
    rm -rf /redis-4.0.9.tar.gz; \
    apk del tar curl

CMD ["sh"]
```

### redis-trib不设置密码的redis-cluster集群启动方式
```
# 用docker加上配置参数跑redis服务时,不能加--daemonize yes, 该参数为yes时, 表示以后台形式运行redis, 但是容器检测不到后台运行的程序的状况, 以为redis挂掉了, 容器就会退出
docker stop redis1 redis2 redis3 redis4 redis5 redis6 redis-trib
docker rm redis1 redis2 redis3 redis4 redis5 redis6 redis-trib
docker network rm redis-network
docker network create --subnet=10.10.57.0/24 redis-network

docker run -itd --name redis1 --net redis-network --ip 10.10.57.101 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis1

docker run -itd --name redis2 --net redis-network --ip 10.10.57.102 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis2

docker run -itd --name redis3 --net redis-network --ip 10.10.57.103 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis3

docker run -itd --name redis4 --net redis-network --ip 10.10.57.104 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis4

docker run -itd --name redis5 --net redis-network --ip 10.10.57.105 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis5

docker run -itd --name redis6 --net redis-network --ip 10.10.57.106 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0
docker logs redis6

docker run -itd --name redis-trib --restart always --net redis-network --ip 10.10.57.90 hegp/redis-trib:4.0.9-alpine sh
docker exec -it redis-trib sh
redis-trib.rb create --replicas 1 10.10.57.101:6379 10.10.57.102:6379 10.10.57.103:6379 10.10.57.104:6379 10.10.57.105:6379 10.10.57.106:6379 
```

##### redis-trib创建带密码的redis-cluster集群，那么requirepass和masterauth都需要设置
```
docker stop redis1 redis2 redis3 redis4 redis5 redis6 redis-trib
docker rm redis1 redis2 redis3 redis4 redis5 redis6 redis-trib

docker network rm redis-network
docker network create --subnet=10.10.57.0/24 redis-network

docker run -itd --name redis1 --net redis-network --ip 10.10.57.101 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis1

docker run -itd --name redis2 --net redis-network --ip 10.10.57.102 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis2

docker run -itd --name redis3 --net redis-network --ip 10.10.57.103 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis3

docker run -itd --name redis4 --net redis-network --ip 10.10.57.104 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis4

docker stop redis5
docker rm redis5
docker run -itd --name redis5 --net redis-network --ip 10.10.57.105 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis5

docker run -itd --name redis6 --net redis-network --ip 10.10.57.106 --restart always redis:4.0.9-alpine redis-server \
--port 6379  \
--protected-mode no \
--pidfile redis.pid \
--appendonly yes \
--cluster-enabled yes \
--bind 0.0.0.0 \
--requirepass admin \
--masterauth admin \
--cluster-node-timeout 5000
docker logs redis6

docker run -itd --name redis-trib --restart always --net redis-network --ip 10.10.57.90 hegp/redis-trib:4.0.9-alpine sh
docker exec -it redis-trib sh
# gem提供的redis工具设置了redis连接密码为空，如果redis集群事先设置了密码，必须修改gem的redis的工具的连接密码
# 修改 /usr/local/bundle/gems/redis-4.0.1/lib/redis/client.rb  第15行的密码参数
sed -i 's|password => nil|password => "admin"|g' /usr/local/bundle/gems/redis-4.0.1/lib/redis/client.rb ; # sed替换密码，修改后就可以连接集群了
redis-trib.rb create --replicas 1 10.10.57.101:6379 10.10.57.102:6379 10.10.57.103:6379 10.10.57.104:6379 10.10.57.105:6379 10.10.57.106:6379 

# redis-trib的各种命令(博客:https://blog.csdn.net/huwei2003/article/details/50973967)
# 1) create命令:创建集群
# 2) check命令:检查集群状态的命令，没有其他参数，只需要选择一个集群中的一个节点即可
redis-trib.rb check 10.10.57.101:6379
# 3) info命令:用来查看集群的信息，没有其他参数，只需要选择一个集群中的一个节点即可
redis-trib.rb info 10.10.57.101:6379
```
