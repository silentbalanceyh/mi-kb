# Windows上的Docker环境搭建

## 0. Tools

* Docker Machine——它将改变看待Docker的角度：
  * 最传统的方式为Linux Distribution加上Docker，这里的Docker在使用者看来，其实和其他Linux上的软件概念没什么不同，仅是一个应用程序。
  * 而CoreOS将Docker进一步提升，在CoreOS中，Docker是用户要运行其他软件的核心平台，否则别无他法。
  * Docker Machine的出现将Docker地位再次提升，用户创建Docker Machine时就只关心Docker，不关心其他组件。
* Docker Swarm——（A Docker-Native Clustering System），CoreOS主战场是集群环境，可对比Docker Swarm和CoreOS Fleet对比，二者都用于构建集群工具。
* Docker Compose——这个特性中规中矩，是Docker应用场景的自然衍生，就Docker设计准则来说，他的目标是足够轻量、只运行一个应用，做一个应用容器，而不是变成一个OS容器。所以Compose引入了group概念，将写作容器组成一个Group，然后使用YAML文件来定义Group，同时对Docker各个标准API进行扩展以支持group、build、pull、run、start、stop等，还引入了`docker up`

## 1. Versions

安装完成过后，打开命令提示符（`cmd.exe`）或者PowerShell，通过调用`docker`和`docker-compose`来检查版本信息：

```
C:\Users\Lang>docker --version
Docker version 17.03.1-ce, build c6d412e

C:\Users\Lang>docker-compose --version
docker-compose version 1.11.2, build f963d76f

C:\Users\Lang>docker-machine --version
docker-machine version 0.10.0, build 76ed2a6
```

## 2. Explore the application & Run examples

### 2.1. Basic Operation

确认您安装了最新的docker，且命令可用，使用`docker ps` 、`docker version` 、`docker info`命令查看Docker信息：

```
PS C:\Users\Lang> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             \n
        STATUS              PORTS               NAMES

PS C:\Users\Lang> docker version
Client:
 Version:      17.03.1-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Tue Mar 28 00:40:02 2017
 OS/Arch:      windows/amd64

Server:
 Version:      17.03.1-ce
 API version:  1.27 (minimum version 1.24)
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Tue Mar 28 00:40:02 2017
 OS/Arch:      windows/amd64
 Experimental: true

PS C:\Users\Lang> docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.03.1-ce
Storage Driver: windowsfilter
 Windows:
Logging Driver: json-file
Plugins:
 Volume: local
 Network: l2bridge l2tunnel nat null overlay transparent
Swarm: inactive
Default Isolation: hyperv
Kernel Version: 10.0 16170 (16170.1000.amd64fre.rs_prerelease.170331-1532)
Operating System: Windows 10 Enterprise
OSType: windows
Architecture: x86_64
CPUs: 8
Total Memory: 15.95 GiB
Name: LANGW520
ID: T2ER:WVIS:7BHW:CYH2:5IY6:4FN4:QPGV:PHQA:JSM6:HM6Q:P7X3:P4PQ
Docker Root Dir: C:\ProgramData\Docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: -1
 Goroutines: 17
 System Time: 2017-04-10T12:43:14.0966205+08:00
 EventsListeners: 0
Registry: https://index.docker.io/v1/
Experimental: true
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```

### 2.2. Hello World

运行实例程序`docker run hello-world`来测试从Docker Hub中下载镜像并启动该容器（在Windows操作系统中，因为远程镜像默认使用了`linux` 所以一定要确认Container是Linux的）：

Windows Container运行结果：

```
C:\Users\Lang>docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
docker: image operating system "linux" cannot be used on this platform.
See 'docker run --help'.
```

Linux Container运行结果（期望结果）：

```
C:\Users\Lang>docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

### 2.3.Import

从快照文件中导入为镜像，例如：

```
docker import - test/xxx
docker images                    // 查看镜像
```

直接导入镜像，使用URL地址或直接使用相对路径，注意load和import二者的区别

#### docker import

```
E:\KTS\Docker>docker import postgres
sha256:37d2f46bf0aae375d85d0df235712579929a31a48c9d094871c2968ca9cd7796

E:\KTS\Docker>docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
<none>              <none>              37d2f46bf0aa        About a minute ago   276 MB
hello-world         latest              48b5124b2768        2 months ago         1.84
```

#### docker load

```
E:\KTS\Docker>docker load < postgres
b6ca02dfe5e6: Loading layer [==================================================>] 128.9 MB/128.9 MB
cac8f844071e: Loading layer [==================================================>] 344.6 kB/344.6 kB
3fdb504c3c49: Loading layer [==================================================>]  4.33 MB/4.33 MB
4cd6ccab8f91: Loading layer [==================================================>] 20.05 MB/20.05 MB
f369c7196f3e: Loading layer [==================================================>] 1.536 kB/1.536 kB
0e60069a5cd9: Loading layer [==================================================>]  7.68 kB/7.68 kB
b640579f03cf: Loading layer [==================================================>] 3.584 kB/3.584 kB
abf767d33e61: Loading layer [==================================================>] 122.1 MB/122.1 MB
e7a81d666e3e: Loading layer [==================================================>] 26.62 kB/26.62 kB
88c151001ae0: Loading layer [==================================================>] 2.048 kB/2.048 kB
3bfe66c7842d: Loading layer [==================================================>]  5.12 kB/5.12 kB
Loaded image: postgres:latest
```

\*注：用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息

### 2.4.Remove

* **docker rmi**：删除镜像images，参数：ImageID
* **docker rm**：删除容器containers，参数：ContainerID

删除镜像之前，需要先删除容器，否则会报类似下边的错误信息：

```
Error response from daemon: conflict: unable to delete 0e24dd8079dc (must be forced)
   - image is being used by stopped container f7b07cca1e77
```





