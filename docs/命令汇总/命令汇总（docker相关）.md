### 1.操作系统权限
`chmod 777 folder_name`

+ **第一个数字 (7)**：用户（文件或文件夹的所有者）的权限。
+ **第二个数字 (7)**：与用户同组的其他用户（同组成员）的权限。
+ **第三个数字 (7)**：所有其他用户（其他用户）的权限。
+ `4` 代表 **读取**（read）。
+ `2` 代表 **写入**（write）。
+ `1` 代表 **执行**（execute）。

### 2.安装docker
+ 下载docker-ce的yum源

```plain
sudo wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

+ 安装Docker

```plain
sudo yum -y install docker-ce
```

+ 检查Docker是否安装成功

```plain
sudo docker -v
```

+ 执行以下命令，启动Docker服务，并设置开机自启动。

```plain
sudo systemctl start docker
sudo systemctl enable docker
```

+ <font style="color:rgb(24, 24, 24);">执行以下命令，查看Docker是否启动。</font>

```plain
sudo systemctl status docker
```

### 3.安装dockercompose
+ <font style="color:rgb(24, 24, 24);">运行以下命令，安装setuptools。</font>

```plain
sudo pip3 install -U pip setuptools
```

+ <font style="color:rgb(24, 24, 24);">运行以下命令，安装docker-compose。</font>

```plain
sudo pip3 install docker-compose
```

+ <font style="color:rgb(24, 24, 24);">运行以下命令，验证docker-compose是否安装成功。</font>

```plain
docker-compose --version
```

### 4.配置docker国内源
```plain
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://dockerhub.icu"]
}
EOF
```

+ tee：将标准输入的内容输出到文件，同时也将内容显示到终端
+ <<EOF：这是一个 heredoc，表示将接下来的内容写入到 `tee` 命令中，直到遇到 `EOF`。
+ {
+     "registry-mirrors": ["https://dockerhub.icu"]
+ }：用于配置 Docker 使用一个指定的镜像仓库镜像加速器 `https://dockerhub.icu`，该加速器用于加速 Docker 镜像的拉取速度
+ sudo systemctl daemon-reload：systemctl是用来控制 systemd 服务的命令工具，加载配置
+ sudo systemctl restart docker：重启

### 5.dockercompose配置文件示例
```plain
services:
  mysql:
    image: mysql:8.4.1
    container_name: mysql-8
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - TZ=Asia/Shanghai
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
    volumes:
      - ${PWD}/mysql/config:/etc/mysql/conf.d
      - ${PWD}/mysql/data:/var/lib/mysql
      - ${PWD}/mysql/logs:/var/log/mysql
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 3306:3306
    restart: always
    healthcheck:
      test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]
      interval: 5s
      timeout: 10s
      retries: 10
  redis:
    image: redis:latest
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - ${PWD}/redis/config/redis.conf:/usr/local/etc/redis/redis.conf:rw
      - ${PWD}/redis/data:/data:rw
    command:
      /bin/bash -c "redis-server /usr/local/etc/redis/redis.conf"
  nacos:
    image: nacos/nacos-server:v2.3.2
    container_name: nacos
    env_file:
      - ${PWD}/nacos/env/nacos-standlone-mysql.env
    volumes:
      - ${PWD}/nacos/logs/:/home/nacos/logs
    ports:
      - "8848:8848"
      - "9848:9848"
    restart: always
    depends_on:
      mysql:
        condition: service_healthy
  namesrv:
    image: apache/rocketmq:5.2.0
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ${PWD}/rocketMQ/data/namesrv/logs:/home/rocketmq/logs
    command: sh mqnamesrv -n 172.16.0.104:9876
  broker:
    image: apache/rocketmq:5.2.0
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    volumes:
      - ${PWD}/rocketMQ/data/broker/logs:/home/rocketmq/logs
      - ${PWD}/rocketMQ/data/broker/store:/home/rocketmq/store
      - ${PWD}/rocketMQ/data/broker/conf/broker.conf:/home/rocketmq/rocketmq-5.2.0/conf/broker.conf
    command: sh mqbroker -n 172.16.0.104:9876  -c /home/rocketmq/rocketmq-5.2.0/conf/broker.conf autoCreateTopicEnable=true
    depends_on:
      - namesrv
  rocketmq-dashboard:
    image: apacherocketmq/rocketmq-dashboard:latest
    container_name: rocketmq-dashboard
    environment:
      - JAVA_OPTS=-Drocketmq.namesrv.addr=172.16.0.104:9876
    ports:
      - "8080:8080"
    restart: always
    depends_on:
      - namesrv
      - broker
  xxl-job:
    image: xuxueli/xxl-job-admin:2.4.1
    container_name: xxl-job
    volumes:
      - ${PWD}/xxl-job/config/application.properties:/application.properties
    ports:
      - "8081:8080"
    restart: always
    depends_on:
      - mysql
  es:
    container_name: es
    image: elasticsearch:8.13.0
    environment:
      - discovery.type=single-node
      - ELASTIC_PASSWORD=123456
      - TZ=Asia/Shanghai
    ports:
      - "9200:9200"
      - "9300:9300"
    mem_limit: 2g
    volumes:
      - ${PWD}/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ${PWD}/es/data:/usr/share/elasticsearch/data
      - ${PWD}/es/plugins:/usr/share/elasticsearch/plugins
    restart: always
  kibana:
    image: kibana:8.13.0
    container_name: kibana
    restart: always
    ports:
      - 5601:5601
    volumes:
      - ${PWD}/kibana/conf/kibana.yml:/usr/share/kibana/config/kibana.yml
    depends_on:
      - es
  seata-server:
    image: seataio/seata-server:2.0.0
    container_name: seata-server
    environment:
      - STORE_MODE=db
      - SEATA_IP=172.16.0.104
      - SEATA_PORT=8091
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${PWD}/seata-server/resources/application.yml:/seata-server/resources/application.yml
      - ${PWD}/seata-server/logs:/seata-server/logs
    ports:
      - "7091:7091"
      - "8091:8091"
    restart: always
  sentinel-dashboard:
    image: javajianghu/sentinel-dashboard:v1.8.8
    container_name: sentinel-dashboard
    environment:
      - dashboard.server=172.16.0.104:8888
    ports:
      - "8888:8888"
    restart: always
```

### 6.查看ip
```plain
ip a
```

### 7.docker命令
```plain
docker-compose up  #启动所有容器
docker-compose up -d  #后台启动并运行
docker-compose stop  #停止容器
docker-compose start  #启动容器
docker-compose down #停止并销毁容器
docker-compose -f standalone-mysql-8.yaml up
```

```plain
docker-compose ps # 查看运行中服务的状态
```

```plain
sudo systemctl start docker && sudo systemctl enable docker # 更新配置并且启动docker
```

```plain
sudo systemctl status docker  # 查看Docker是否启动
```

```plain
sudo docker exec -it mysql-8 /bin/bash # 进入mysql镜像
show variables like '%port%';
mysql -uroot -p123456 --default-character-set=utf8
SELECT user, host FROM mysql.user;

```

目前好用的源

```plain
https://github.com/dongyubin/DockerHub?tab=readme-ov-file
"https://hub.fast360.xyz"
"https://docker.m.daocloud.io"
```

```plain
ss -tuln # 查看哪些端口被用up -d
```

查看docker容器网络ip

```plain
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql-container
```

创建网络

```plain

```

### 8.创建文件命令
```plain
touch filename
```

### 9.修改ip的脚本
```plain
#!/bin/bash
if [ ! -n "$1" ] ;then
    echo "要修改的ip为空"
else
    echo "ip为：$1"
    sed -i "s/172.16.0.104/$1/g" docker-compose.yaml
    sed -i "s/172.16.0.104/$1/g" nacos/env/nacos-standlone-mysql.env
    sed -i "s/172.16.0.104/$1/g" rocketMQ/data/broker/conf/broker.conf
    sed -i "s/172.16.0.104/$1/g" xxl-job/config/application.properties
    sed -i "s/172.16.0.104/$1/g" kibana/conf/kibana.yml
    sed -i "s/172.16.0.104/$1/g" seata-server/resources/application.yml
fi
```

### 10.redis
docker进入redis命令行

```plain
docker exec -it redis redis-cli
```

