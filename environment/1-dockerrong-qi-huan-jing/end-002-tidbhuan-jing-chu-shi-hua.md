# TiDB环境

TiDB充当了Origin X Engine的元数据仓库的作用，里面主要存储了下列配置，由于它是一种MySQL兼容协议，所以最终搭建好了过后可以直接使用MySQL的客户端访问。

## 1. 安装

确保您的本机已经安装了Docker环境

```shell
# script/shell/ox-tidb.sh
# 必须进入到shell目录中执行，文件路径使用了相对路径
cd script/shell
./ox-tidb.sh
```

执行了该脚本过后，就可以启动一个新的`ox_tidb`的容器了，该脚本的内容如：

```shell
#!/usr/bin/env bash
export DATA_DIR="/Users/lang/Runtime/ox-engine/data"
img_name="tidb"
container_name=ox_${img_name}

docker stop ${container_name}
docker rm ${container_name}
docker rmi ${img_name}:latest

docker build -t ${img_name}:latest -f ox-tidb .
docker run \
  -p 4000:4000 \
  -v ${DATA_DIR}:/tmp/tidb \
  --name ${container_name} ${img_name}
```

注意环境变量`DATA_DIR`，该变量设置的路径会作为本机中TiDB存储数据文件的路径专用，该路径需要是自己本机存在的一个路径，否则会启动不了容器。搭建好了过后，默认的 root 是免密码登录，所以可直接使用 root 登录到环境中。

## 2. 连接

使用任何一个客户端或者mysql的命令可登录TiDB：

### 2.1. 命令登录

```shell
/usr/local/mysql/bin/mysql -u root -P 4000 -h 127.0.0.1
```

> 由于mysql命令对root密码有一定的要求，所以最好在最初将TiDB的密码修改掉，或者不使用root账号登录环境。

### 2.2. 界面连接

下图是使用Navicat工具连接TiDB的截图：

![](/assets/images/end/002/tidb-connect.png)

