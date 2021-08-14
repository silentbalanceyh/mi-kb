# Neo4j环境初始化

在准备环境之前，先看看Neo4j在百度百科的介绍：Neo4j是一个高性能的,NOSQL图形数据库，它将结构化数据存储在网络上而不是表中。它是一个嵌入式的、基于磁盘的、具备完全的事务特性的Java持久化引擎，但是它将结构化数据存储在网络\(从数学角度叫做图\)上而不是表中。Neo4j也可以被看作是一个高性能的图引擎，该引擎具有成熟数据库的所有特性。程序员工作在一个面向对象的、灵活的网络结构下而不是严格、静态的表中——但是他们可以享受到具备完全的事务特性、企业级的数据库的所有好处。

| 连接字符串 | 用户 | 口令 |
| :--- | :--- | :--- |
| bolt://localhost:6075 | neo4j | neo4j |

## 1. 安装

确认开发机器上已经安装了Docker环境，并保证Docker Engine已经运行起来。

```shell
# script/shell/ox-neo4j.sh
# 必须进入到shell目录中执行，文件路径使用了相对路径
cd script/shell
./ox-neo4j.sh
```

如果是「第一次」运行该脚本，需要等待Docker下载相关镜像信息：

```shell
» ./ox-neo4j.sh                                                                                                                                                                                      lang@LangYus-MacBook-Pro
Error response from daemon: No such container: ox_neo4j
Error: No such container: ox_neo4j
Error: No such image: neo4j:latest
Sending build context to Docker daemon  4.096kB
Step 1/1 : FROM neo4j:latest
latest: Pulling from library/neo4j
cd784148e348: Pull complete 
35920a071f91: Pull complete 
1a5149a464dd: Pull complete 
15bb04bfc35a: Pull complete 
2193028004db: Pull complete 
04035670872b: Pull complete 
255177ae817c: Pull complete 
Digest: sha256:5fb3cc45d92136b2d8bd6917a51efe5035bec0de49cdb27cfb0ddf477cd83069
Status: Downloaded newer image for neo4j:latest
 ---> 76b00a40e5f7
Successfully built 76b00a40e5f7
Successfully tagged neo4j:latest
74da44d584e55a8d5424dd39f5ee54b32532ef0cb878f54dd645a51cfcc72db5
```

## 2. 脚本内容

`ox-neo4j.sh`脚本的内容如下：

**「第一次初始化环境的时候执行，之后不用执行全部脚本，执行Docker运行部分**`docker run`**就可以了」**

```shell
#!/usr/bin/env bash
img_name="neo4j"
container_name=ox_${img_name}

docker stop ${container_name}
docker rm ${container_name}
docker rmi ${img_name}:latest

docker build -t ${img_name}:latest -f ox-neo4j .
docker run \
  -env=NEO4J_AUTH=neo4j/neo4j \
  -d -p 6074:7474 \
  -d -p 6073:7473 \
  -d -p 6075:7687 \
  --name ${container_name} ${img_name}
```

`ox-neo4j`的内容如下（Dockerfile）

```dockerfile
FROM neo4j:latest
```

上述脚本中的`NEO4J_AUTH`设置的就是下边登录过程中使用的账号和密码。

## 3. Web控制台

运行好这个Docker过后，可直接在浏览器中打开：[http://localhost:6074/](http://localhost:6074/)登录Web控制台，第一次登录时会提示您修改neo4j的密码，可以参考本文最开始把密码「口令」改掉，输入任意密码都可。

![](/assets/images/env/001/14-neo4j-first.png)在下一个界面将密码改成iop需要的密码：

![](/assets/images/env/001/14-neo4j-password.png)

