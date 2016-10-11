## Dockerfile Note

Dockerfile 是用于制作Docker Image（镜像）的文件。

FROM MAINTAINER
RUN CMD ENV ENTRYPOINT VOLUME
USER WORKDIR
ADD COPY
LABEL EXPOSE
ARG ONBUILD STOPSIGNAL HEALTHCHECK SHELL

### 理解 Docker 容器运行的模型
Image: 一系列堆叠的Read-Only-Layer (docker history)   
Container: 在Image的基础上加上一层Read-Write-Layer (docker diff)  
Tips:
  * 上层Layer会覆盖下层Layer
  * 每层之间的变化会提现到镜像到体积里面
  * 区分两种命令，一种是BuildTime，一种是RunTime（CMD／ENTRYPOINT）

### 基本的 docker run flag
docker run IMAGE [command]
docker run --entrypoint="/app/run/start.sh" IMAGE
docker run -p 8080:80 IMAGE
docker run -e FOO=bar IMAGE
docker run -u foo IMAGE whoami
docker run -w="/app/run"


### 用 docker run 命令来理解 Dockerfile

ENV FOO=bar
```
cid = $(docker run -e Foo=bar <Image>)
docker commit $cid
```

VOLUME /mydata
```
cid = $(docker run -v /mydata <Image>)
docker commit $cid
```
在VOLUME /somedata 之后   
所有RUN命令中对 /somedata及其目录内的操作都不会生效  


### 最佳实践

#### 最小化 Context


#### 安装依赖的正确姿势
```
FROM debian:wheezy
RUN apt-get update && apt-get install -y wget
```

```
FROM debian:wheezy  
RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*
```

```
COPY apache-maven-3.0.5-bin.tar.gz /tmp/
RUN tar -xzvf apache-maven-3.0.5-bin.tar.gz /usr/local/ && \
    mv /usr/local/apache-maven-3.0.5 /usr/local/maven
```

```
ADD apache-maven-3.0.5-bin.tar.gz /usr/local/
RUN ln -s /usr/local/apache-maven-3.0.5 /usr/local/maven
```

#### 利用 Build Cache

```
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY. /tmp/
```

```
COPY. /tmp/
RUN pip install --requirement /tmp/requirements.txt
```

#### 配合使用 CMD 和 ENTRYPOINT
| Image ENTRYPOINT        | Image CMD           | Container CMD  | Final CMD
| ------------- |:-------------:| -----:|-----:|
| ／cmd      | [foo bar] | NULL | [cmd foo bar]|
| ／cmd      | [foo bar]      |   other-cmd |[cmd other-cmd]|

ENTRYPOINT [/myapp.sh]  
CMD [--help]

#### 为 Dockerfile 写测试
