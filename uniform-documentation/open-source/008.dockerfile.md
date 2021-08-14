# Dockerfile写法

## FROM

用法：

* `FROM <image>`
* `FROM <image>:<tag>`
* `FROM <image>@<digest>`

信息：

* **FROM**必须是没有注释的代码行
* **FROM**如果需要创建多个镜像，在单个Dockerfile文件中FROM可以出现多次——在每个FROM之前，简单使用注释备注一下当前这个镜像的ImageID
* `tag` 和`digest` 两个值是可选的，如果没提供则使用`latest` 的值体改替代，如果builder无法匹配`tag` 中的值则报错

## MAINTAINER

用法：

* `MAINTAINER <name>`

该命令允许您在文件中设置作者信息。

## RUN

用法：

* `RUN <command>` （shell格式，command命令将在shell中运行，Linux中默认使用`/bin/sh -c` ，在Windows中默认使用`cmd /S /C` ）
* `RUN ["<executable>","<param1>","<param2>"]`exec格式

信息：

* exec格式可防止shell中使用字符串，直接运行`RUN`命令会使用不包含特殊Shell可执行的基础镜像。
* 默认的shell可直接改成`SHELL`命令来完成。
* 如果直接使用`exec` 格式会触发普通的shell流程，如：

  ```
  RUN ["echo","$HOME"]
  ```

  将会打印环境变量

## CMD

用法：

* `CMD ["<executable>","<param1>","<param2>"]`（直接执行exec方式）
* `CMD ["<param1>","<param2>"]`（默认参数同`ENTRYPOINT`）
* `CMD <command> <param1> <param2>`（Shell模式）

信息：

* `CMD` 的主要意图是提供默认的执行容器，默认容器包含了一个执行器，若你使用了`ENTRYPOINT` 则可省略该执行器。
* 在一个Dockerfile中只能有一个`CMD`指令，若您提供了超过一个`CMD`，则只有最后一个`CMD`会生效。
* 如果`CMD`用来提供了`ENTRYPOINT`指令的默认参数，`CMD`和`ENTRYPOINT`指令应该使用JSON Array的格式。
* 如果为`docker run`制定了特殊参数，它将覆盖默认指定的`CMD`中的参数。
* 如果直接使用`exec`格式会触发普通的shell流程。

## LABEL

用法：

* `LABEL <key>=<value> [<key>=<value> ...]`

信息：

* `LABEL`指令可以在镜像中添加元数据。
* 如果要在LABEL中使用空白，则用引号或反斜杠，就像在命令行解析中一样。
* Labels本身是附加的，包括`FROM`镜像中的`LABEL`。
* 如果Docker遇到了已经存在键的label，新值将会替换掉旧值。
* 若要查看镜像中的labels，则使用`docker inspect`命令，所有信息将位于`"Labels"`的JSON属性中。

## EXPOSE

用法：

* `EXPOSE <port> [<port> ...]`

信息：

* 通知Docker容器运行时将要监听的网络端口。
* `EXPOSE`开放的端口不使用主机端口也可访问。

## ENV

用法：

* `ENV <key> <value>`
* `ENV <key>=<value> [<key>=<value> ...]`

信息：

* 使用`ENV`命令可设置`<key>=<value>`的环境变量。
* 这个值将在所有"descendant"的Dockerfile环境中生效，也可以内联替换。
* 环境变量在容器从镜像运行开始后一直有效。
* 第一个格式可设置但个环境变量，`<value>`中所有字符d吧生效包括空白以及引号。

## ADD

用法：

* `ADD <src> [<src> ...] <dest>`
* `ADD ["<src>",... "<dest>"]`（这种格式用于解析包含了空白字符参数的情况）

信息：

* 拷贝`<src>`指定的新文件、目录或者远程URLs，并且将它添加到镜像中`<dest>`指定的文件系统里。
* `<src>`中可以包含通配符，使用Go语言中的`filepath.Match`规则匹配。
* 如果`<src>`是一个文件或目录，则它们必须是和目录源相关（build的上下文环境）。
* `<dest>`是一个绝对路径，或相当于`WORKDIR`的相对路径。
* 如果`<dest>`不存在，则创建丢失的目录路径。

## COPY

用法：

* `COPY <src> [<src> ...] <dest>`
* `COPY ["<src>", ... "<dest"]`（这种格式用于解析包含了空白字符参数的情况）

信息：

* 拷贝`<src>`指定的新文件、目录或者远程URLs，并且将它添加到镜像中`<dest>`指定的文件系统里。
* `<src>`中可以包含通配符，使用Go语言中的`filepath.Match`规则匹配。
* 如果`<src>`是一个文件或目录，则它们必须是和目录源相关（build的上下文环境）。
* `<dest>`是一个绝对路径，或相当于`WORKDIR`的相对路径。
* 如果`<dest>`不存在，则创建丢失的目录路径。

## ENTRYPOINT

用法：

* `ENTRYPOINT ["<executable>","<param1>","<param2>"]`（前文提到的exec格式）
* `ENTRYPOINT <command> <param1> <param2>`（shell格式）

信息：

* 运行您配置可执行的容器。
* `docker run <image>`中的命令行参数将会追加到`ENTRYPOINT`中的`exec`格式后，并且会覆盖使用CMD时的元素。
* shell格式将会阻止任何`CMD`或者命令行使用的其他参数，但是`ENTRYPOINT`将会从shell启动。也就是说可执行容器不会是PID 1，也不会收到Unix信号，预处理`exec`解决了这个问题。
* Dockerfile中仅仅只有最后一个`ENTRYPOINT`会生效。

## VOLUME

用法：

* `VOLUME ["<path>",...]`
* `VOLUME <path> [<path> ...]`

使用特殊名字创建一个挂载点使得它可以从本机主机和其他容器访问该卷。

## USER

用法：

* `USER <username | UID>`

使用`USER`指令可设置运行镜像的用户名和UID，该信息会在`RUN`，`CMD`和`ENTRYPOINT`中使用。

## WORKDIR

用法：

* `WORKDIR </path/to/workdir>`

信息：

* 设置命令（如`RUN`，`CMD`，`ENTRYPOINT`，`COPY`，`ADD`）的工作目录。
* 在一个文件中可使用多次，若相对路径提供了，它将使使用前一次`WORKDIR`指令设置的工作目录。

## ARG

用法：

* `ARG <name> [=<default value>]`

信息：

* 定义用户可以传给`docker build`命令的build-time变量信息，直接使用`--build-arg <varname>=<value>`格式。
* 可使用`ARG`多次来设置多个变量的信息。
* 不推荐使用build-time变量传递一些密码如github中的keys, credentials等，build-time变量对任何镜像中的历史用户都是可见的。
* 如果同时使用了`ENV`指令设置了环境变量，则`ENV`中的变量会覆盖`ARG`指定的变量。
* Docker设置了预定义的`ARG`变量，您可以不适用ARG定义直接使用：
  * `HTTP_PROXY`
  * `HTTPS_PROXY`
  * `FTP_PROXY`
  * `NO_PROXY`

## ONBUILD

用法：

* `ONBUILD <Dockerfile INSTRUCTION>`

信息：

* 当一个镜像是基于另一个镜像在使用时，为镜像添加一个触发指令让它之后触发。如果在downstream镜像的Dockerfile的`FROM`指令后边添加了它，这个触发器将在downstream构建的上下文环境中执行。
* 任何build指令都可以注册成一个触发器。
* 触发器只能被"child"构建继承，换句话说，它不能被”grand-children"继承。
* `ONBUILD`指令不会触发`FROM`，`MAINTAINER`或`ONBUILD`指令。

## STOPSIGNAL

用法：

* `STOPSIGNAL <signal>`

`STOPSIGNAL`指令可设置系统调用的signal，它将在退出时发送给容器。这个signal可以是一个合法的无符号正整数，它的位置可以在内核syscall调用中，如9，或者使用signal名称（用`SIGNAME`指定），如`SIGKILL`。

## HEALTHCHECK

用法：

* `HEALTHCHECK [<options>] CMD <command>`（在容器内部执行命令检查容器运行状况）
* `HEALTHCHECK NONE`（关闭任何从base镜像中继承的healthcheck）

信息：

* 告诉Docker如何测试一个容器并知道它仍然在工作。
* 每当一个检查通过，容器变成`healthy`，经过一定数量的连续故障，容器变成`unhealthy`。
* 这里`<options>`可表示：
  * `--interval=<duration>`（默认：30s）
  * `--timeout=<duration>`（默认：30s）
  * `--retries=<number>`（默认：3）
* 容器启动过后Health check将首先运行`interval`秒，然后前一次检查结束后过了`interval`秒执行第二次检查。如果单次检查超过了`timeout`的时间（秒）则检查并定义为失败。若在`retries`的次数的连续故障后，则标记成`unhealthy`。
* 一个Dockerfile中只能包含一个`HEALTHCHECK`指令，若您列举了多个该指令，只有最后一个会生效。
* `<command>`可以是一个shell命令，也可以是一个JSON数组。
* 命令退出状态可标识容器状况：
  * `0`：success——容器健康，使用就绪
  * `1`：unhealthy——容器未正常工作
  * `2`：reserved——不要使用该退出代码
* `<command>`命令中若使用了stdout和stderr则前4096个字节会被保存，然后调用`docker inspect`可查看。
* 当一个容器的健康状况发生改变时，一个`health_status`事件将会产生。

## SHELL

使用：

* `SHELL ["<executable>","<param1>","<param2>"]`

信息：

* 允许使用默认的shell，并覆盖最初的shell。
* 每个`SHELL`指令将会覆盖之前所有`SHELL`指令，并影响后续指令。
* 允许使用其他shell如：`zsh`、`csh`、`tcsh`、`powershell`等。



