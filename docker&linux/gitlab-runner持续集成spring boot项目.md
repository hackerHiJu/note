### 1、安装gitlab-runner容器(包括安装maven环境以及java环境，因为是部署的java项目)
```
FROM gitlab/gitlab-runner:v11.0.2
MAINTAINER zhonghaijun <zhonghaijun1998@qq.com>

# 修改软件源
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get clean

# 安装 Docker （容器中也可以安装docker，但是需要将容器中的docker /var/run/docker.sock 关联到宿主机中的/var/run/docker.sock，这样就可以操作宿主机中的docker了）
RUN apt-get -y install apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update -y && \
    apt-get install -y docker-ce
COPY daemon.json /etc/docker/daemon.json

# 安装 Docker Compose
WORKDIR /usr/local/bin
RUN wget https://raw.githubusercontent.com/topsale/resources/master/docker/docker-compose
RUN chmod +x docker-compose

# 安装 Java（可以网上下载，也可以自己copy进容器安装）
RUN mkdir -p /usr/local/java
WORKDIR /usr/local/java
COPY jdk-8u152-linux-x64.tar.gz /usr/local/java
RUN tar -zxvf jdk-8u152-linux-x64.tar.gz && \
    rm -fr jdk-8u152-linux-x64.tar.gz

# 安装 Maven （可以网上下载，也可以自己copy进容器安装）
RUN mkdir -p /usr/local/maven
WORKDIR /usr/local/maven
RUN wget https://raw.githubusercontent.com/topsale/resources/master/maven/apache-maven-3.5.3-bin.tar.gz
# COPY apache-maven-3.5.3-bin.tar.gz /usr/local/maven
RUN tar -zxvf apache-maven-3.5.3-bin.tar.gz && \
    rm -fr apache-maven-3.5.3-bin.tar.gz
# COPY settings.xml /usr/local/maven/apache-maven-3.5.3/conf/settings.xml

# 配置环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_152
ENV MAVEN_HOME /usr/local/maven/apache-maven-3.5.3
ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin

WORKDIR /

```

### 2、将daemon.json 关联上容器中的docker，用于配置加速器和仓库地址

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    自己仓库的地址
  ]
}
```

### 3、使用docker-compose来启动gitlab-runner服务
```
version: '3.1'
services:
  gitlab-runner:
    build: environment # build会去找environment路径下的Dockerfile文件来进行构建进行（或者直接使用镜像名）
    restart: always 
    container_name: gitlab-runner 
    privileged: true  # 以管理员的身份进行
    volumes:
      - /usr/local/docker/runner/config:/etc/gitlab-runner  # gitlba-runner的配置
      - /var/run/docker.sock:/var/run/docker.sock  # 挂载的是容器中的docker跟宿主机docker的配置
```

### 4、注册runner

```
docker exec -it [容器id] gitlib-runner gitlab-ci-multi-runner register
```

### 5、配置项目中的.gitlab-ci.yml
```
stages:
  - build
  - run
  - clean

build:
  stage: build
  script:
    - /usr/local/maven/apache-maven-3.6.1/bin/mvn clean package  # 出现找不到mvn命令的情况，直接给绝对路径
    - cp target/*.jar src/docker/  # 将打包好的jar移动到Dockerfile的路径下，打包镜像，因为每个阶段都是独立的，jar包被打包好就会删除
    - cd src/docker/
    - docker build -t haijun1998/scientific:v1.0 .
run:
  stage: run
  script:
    - cd src/docker/       # 移动到docker-compose.yml文件夹下面
    - docker-compose down  # 先去关闭以up来运行的容器
    - docker-compose up -d # 启动容器

clean:
  stage: clean
  script:
    - docker rmi $(docker images -q -f dangling=true)   # 删除为none的虚悬镜像

```

### 6、使用Dockerfile 来将项目打包成镜像
```
FROM java:8

RUN mkdir -p /usr/local/app
WORKDIR /usr/local/app
COPY *.jar app.jar
EXPOSE 8011

ENTRYPOINT ["java","-jar","app.jar"]
```

### 7、使用docker-compose.yml来启动服务
```
version: '3.1'
services:
  scientific-server:  #服务名
    restart: always
    container_name: scientific #容器名称
    image: haijun1998/scientific:v1.0  #镜像名称
    ports:
      - 8011:8011
```

### 8、在此期间会遇到 gitlab调用容器执行docker没有权限的问题
```
因为gitlab会使用gitlab-runner用户来执行，所以没有权限
宿主机：vim /etc/group 查看组有没有docker 。 如果没有 groupadd -g 1212 docker 重启docker  systemctl restart docker   然后需要将docker.sock 加入到docker组中 

docker容器中：给容器中的docker.sock 加入到组中  chgrp /var/run/docker.sock docker 并且将gitlab-runner加入到docker组中 usermod -a -G docker gitlab-runner
```