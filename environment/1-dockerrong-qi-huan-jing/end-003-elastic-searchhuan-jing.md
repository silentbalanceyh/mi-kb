# Elastic Search环境初始化

## 1. 安装

确认开发机器上已经安装了Docker环境，并保证Docker Engine已经运行起来。

```shell
# script/shell/ox-es.sh
# 必须进入到shell目录中执行，文件路径使用了相对路径
cd script/shell
./ox-es.sh
```

如果是「第一次」运行该脚本，需要等待Docker下载相关镜像信息：

```shell
./ox-es.sh                                                                                                                                                                                          lang@LangYus-MacBook-Pro
Error response from daemon: No such container: ox_es
Error: No such container: ox_es
Error: No such image: es:latest
Sending build context to Docker daemon  6.144kB
Step 1/1 : FROM elasticsearch:2.4.6
6.5.4: Pulling from elasticsearch/elasticsearch
a02a4930cb5d: Pull complete 
dd8a94cca3f9: Pull complete 
bd73f551dee4: Pull complete 
70de352c4efc: Pull complete 
0b5ae4c7310f: Pull complete 
489d9f8b18f1: Pull complete 
8ba96caf5951: Pull complete 
f1df04f27c5f: Pull complete 
Digest: sha256:5ca85697b6273f63196b44c32311c5a2d1135af9cfd919e5922e49c5045d04b8
Status: Downloaded newer image for elasticsearch:2.4.6
 ---> 93109ce1d590
Successfully built 93109ce1d590
Successfully tagged es:latest
```

## 2. 脚本内容

`ox-es.sh`的脚本内容如下

```shell
#!/usr/bin/env bash
img_name="es"
container_name=ox_${img_name}

docker stop ${container_name}
docker rm ${container_name}
docker rmi ${img_name}:latest

docker build -t ${img_name}:latest -f ox-es .
docker run \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  --name ${container_name} ${img_name}
```

`ox-es`的文件内容如下（Dockerfile）

```shell
FROM elasticsearch:2.4.6
```

由于jhipster的部分限制，es的版本最好和当前使用版本匹配：2.4.6，如果使用高版本或者其他版本有可能会运行不了，启动好ES过后，直接在浏览器中打开：[http://localhost:9200/](http://localhost:9200/) 地址，就可以看到下边的信息，证明ES已经启动好了：

```json
{
  "name" : "the Tomorrow Man Zarrko",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "ja-VLpfQR52BDcfmVW5VSg",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
```



