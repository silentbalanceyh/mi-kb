# Maven环境搭建

# 1. 安装

### 1.1. 下载

下载地址：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

![](/assets/images/env/006/1001-1.png)

Windows下载zip版，Linux/Mac则直接下载tar.gz版本

### 1.2. 设置

下载过后，将Maven解压到某个操作系统目录中

![](/assets/images/env/006/1001-2.png)

解压过后修改环境变量，Windows中可直接修改，Linux/Mac中修改文件`~/.bash_profile`

```
vim ~/.bash_profile
```

![](/assets/images/env/006/1001-3.png)

设置这几个环境变量

```
M2_HOME=/Users/lang/Tool/Maven
PATH=$M2_HOME/bin:$PATH            # 这个变量是追加，不要覆盖
```

保存该文件过后使用下边命令让环境变量生效

```
source ~/.bash_profile
```

### 1.3. 测试

安装好过后，可使用下边命令测试Maven是否安装成功

```
mvn -version
```

![](/assets/images/env/006/1001-4.png)

