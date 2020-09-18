# 环境搭建

>  xx 在文中代表客户名称，infix-hy 项目中的内容因客户不同而有所差异
## 0. 修改 host

> 开发过程中需要使用该步骤，减少配置文件的修改

在您的 host 文件中增加如下记录（推荐把您所有的合法本机IP加入，主要映射到域名：`ox.engine.cn` ）：

```shell
# Ox Engine
127.0.0.1 ox.engine.cn
# 172.20.10.2 ox.engine.cn
# 10.0.0.6 ox.engine.cn
192.168.1.8 ox.engine.cn
```

## 1. 前端环境

前端环境的基本要求，保证您的电脑上有基础环境：`node 13.x`（下载地址：[https://nodejs.org/en/](https://nodejs.org/en/)），在您的环境中使用一个空目录，下载代码（**如果需要做前端开发请走Fork流程**，参考 2.3 章节）：

> 该代码属于闭源代码，所以下载需要申请权限。

```shell
git clone https://github.com/silentbalanceyh/ox-ui.git
```

代码下载过后，进入到代码目录：

```shell
cd ox-ui
```

安装依赖包（这个步骤需要等待一段时间）：

```shell
npm install
```

然后执行脚本运行前端：

```shell
./run-zero.sh      # Linux 专用
./run-zero.bat     # Windows 专用
```

看到如下输出证明启动完成：

```shell
You can now view vertx-zero in the browser.

  Local:            http://localhost:5000/
  On Your Network:  http://192.168.1.3:5000/

Note that the development build is not optimized.
To create a production build, use yarn build.

ℹ ｢atl｣: Time: 2344ms
```

在浏览器中输入：[http://ox.engine.cn:5000/ox/login/index](http://ox.engine.cn:5000/ox/login/index)，后端如果没有启动则可以看到如下界面：

![](/assets/images/ox/002-1.png)

## 2. 后端环境

基本环境要求，保证你电脑上有以下基础环境

| 基础环境 | 版本要求 |
| :--- | :--- |
| Java（JDK） | &gt;= 8.x（最低8） |
| Maven | &gt;= 3.6.x |
| MySQL | &gt;= 5.7 |
| ElasticSearch | &gt;= 6.x（最新版即可） |
| Git | &gt;= 2.x |

### 2.1. 数据库基本配置

数据库**暂时**统一使用账号和密码，统一访问MySQL数据库，需要注意的是 `DB_IOP`数据库是业务数据库，并且是动态的，所以不存在于`vertx-jooq.yml`的配置中，而元数据库和历史库直接配置在该文件中，所以在准备数据库的时候创建好对应的三个数据库。

#### 2.1.1. 核心参数

| 参数 | 值 |
| :--- | :--- |
| Character Set | utf8mb4 |
| Collation | utf8mb4\_bin |

#### 2.1.2. 启动参数

启动参数的配置文件可以在网上查，我本机配置地址为：`/usr/local/etc/my.cnf`，在启动参数中追加核心配置：

```properties
[mysqld]
# 解决 Many Connections 的专用配置
max_connections=2048

# 解决 Communications link failure 专用配置
wait_timeout=1814400

# 性能基础参数（可选）
innodb_buffer_pool_size=4G
innodb_log_file_size=256M
innodb_flush_log_at_trx_commit=2
innodb_flush_method=O_DIRECT

# 批量处理专用参数
max_allowed_packet = 256M

#（Windows中的默认值是1，只有Windows需要该设置，Linux默认0，Mac默认就是2）
lower_case_table_names = 2
```

### 2.2. zero 框架编译

1. 在本机找一个空目录，输入：

    ```shell
    git clone https://github.com/silentbalanceyh/vertx-zero.git
    ```

2. 代码下载完成后，进入到项目目录，并且执行 Maven 编译（不跳过测试）

    ```shell
    cd vertx-zero
    
    # 不跳过测试编译
    mvn clean package install
    ```
3. 看到如下截图证明编译通过

    ```shell
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  03:51 min
    [INFO] Finished at: 2020-03-24T11:34:06+08:00
    [INFO] ------------------------------------------------------------------------
    ```


> 保证自己机器中的 zero 版本是最新的，由于 zero 是开源框架，并且项目中一直使用最新版，如果有更新我会提示，若想要对 zero 进行修改，那么请使用标准的 Fork - PR 流程。

### 2.3. ox-engine 代码下载

> 该代码属于闭源代码，所以下载需要申请权限。

开发人员在更改 ox-engine 代码时必须走标准的 Fork - PR 流程：

![](/assets/images/ox/002-3.png)

代码下载流程：

>  关键点：分清楚远程核心库，自己的远程仓库，自己的本地仓库！

1. 登录自己的github账户，打开地址：[https://github.com/silentbalanceyh/ox-engine](https://github.com/silentbalanceyh/ox-engine)，在右上角点击`Fork`按钮，将仓库代码Fork到自己的仓库中，完成第一步操作，如果你的账号是 `lang.yu`，Fork过后会生成一个新库（在你自己的github账号中）：[https://github.com/lang.yu/ox-engine](https://github.com/lang.yu/ox-engine)（这是你自己的库）
2. 运行下边命令在自己本地下载代码（从自己的库拉取）
    
    ```shell
     git clone https://github.com/lang.yu/ox-engine
    ```
3. 进入项目目录

    ```shell
     cd ox-engine
    ```
4. 在项目目录添加远程库引用（注意这里使用我的远程库，核心库）

    ```shell
     git remote add upstream https://github.com/silentbalanceyh/ox-engine/
     # 这里的 upstream 可自已定义，一般推荐使用 upstream，它表示：中心核心库是你个人仓库的上游库
    ```

经过了上边四个步骤，代码就下载到本机，并且搭建好 Fork 环境了，在 Fork 环境中需要注意几点：

1. 更新代码需要使用：`git pull upstream master`，它表示从你的`upstream`引用的`master`分支下更新最新的代码，实际上是从远程核心库拉代码。
2. 提交代码时使用：`git push`，实际上这份代码并没有合并到远程核心库中，只是提交到你个人的远程库中了。
3. 当你的代码通过了测试，需要将代码合并的时候，可以以 `PR` 的形式提交 `Pull Request`，如何提交PR请自行百度。

## 3. IDE环境搭建

### 3.1. 导入代码到IDEA

>  如果没有 License 可以用社区办的 IDEA 和 WebStorm，足够用了。

1. 打开主界面，选择**导入项目（Import Project）**：

    ![](/assets/images/ox/002-4.png)
2. 选择 Maven 的项目类型
    ![](/assets/images/ox/002-5.png)
3. 第一次导入的时候会很慢，导入成功过后可以在项目窗口看到如下：
    ![](/assets/images/ox/002-6.png)
4. 并且在 Maven 项目配置窗口可以看到目前所有的项目表：
    ![](/assets/images/ox/002-7.png)

### 3.2. 项目编译

由于项目本身使用了`Oracle / UCMDB`的依赖库，所以在编译项目之前需要做相关的准备工作。

1. 直接进入脚本目录（从 ox-engine 项目根目录进入）：

    ```shell
    cd script/maven/
    ```
2. （Oracle本地库）Linux环境可以直接运行命令，Windows运行完整命令：

    ```shell
    ./install-oracle-jar.sh
    # 如果是 Windows 可直接出来运行（Maven）
    mvn install:install-file  \
        -Dfile=ojdbc7-12.1.0.2.jar  \
        -DgroupId=com.oracle  \
        -DartifactId=ojdbc7 \
        -Dversion=12.1.0.2 \
        -Dpackaging=jar
    ```
3. （Ucmdb本地库）：

    ```shell
    ./install-ucmdb-2019-jar.sh
    # 如果是 Windows 则直接运行：
    mvn install:install-file \
        -Dfile=ucmdb-api-2019.2.jar \
        -DgroupId=com.microfocus \
        -DartifactId=ucmdb-api \
        -Dpackaging=jar \
        -Dversion=2019.2
    ```

4. 回退到根目录中，执行下边命令编译（记得跳过测试用例）：

    ```shell
    mvn clean package install -DskipTests=true
    ```
5. **重要步骤**：检查环境目录：`ox-driver/ix-atlantic/src/main/resources`，看是否存在如下目录，如果不存在的话手动创建。：
    ![](/assets/images/ox/002-8.png)

### 3.3. 运行配置设置

1. 整个项目有三个主要入口：
    ![](/assets/images/ox/002-9.png)
2. 三个入口的含义如下：
    1. OxAgent：（开发测试专用）容器启动入口，启动主容器
    2. OxDevelop：（开发测试专用）导入特殊配置时使用（可更改）
    3. OxEntry：（生产环境专用）开发人员可忽略，程序主入口
3. 运行`OxAgent`生成配置，运行过一次过后就会生成配置，生成配置过后，先停掉容器，点击：**Edit Configurations...** 菜单：
    ![](/assets/images/ox/002-10.png)
4. 编辑`OxAgent`的配置程序，注意修改 Working directory：
    ![](/assets/images/ox/002-11.png)
5. 重新生成 `OxEntry` 的运行配置，并且编辑配置，注意比 `OxAgent` 多一个配置（Program arguments = "dev" ）：
    ![](/assets/images/ox/002-12.png)

### 3.4. 数据初始化

1. 进入到 `ox-engine` 的根目录，进入数据库初始化目录：
    
    ```shell
        cd script/database
        ./database-reinit.sh          # Linux 专用
        ./database-reinit.bat         # Windows 专用
    ``` 
2. 进入元数据仓库初始化目录初始化`DB_ORIGIN_X`元数据库：

    ```shell
         cd ox-driver/ix-atlantic
         ./run-init.sh                 # Linux 专用
         ./run-init.bat                # Windows 专用
    ```
3. 进入xx项目目录初始化`DB_IOP`数据库：
    
    ```shell
        cd ox-plugin/infix-hy/hy-xx-up
        ./run-init.sh                 # Linux 专用
        ./run-init.bat                # Windows 专用
    ```
4. 初始化完成后，直接运行 3.3. 步骤中配置的 `OxEntry`，可以看到如下界面，输入`h`的帮助文档：
    ![](/assets/images/ox/002-13.png)
1. 执行命令：
    1. `d`执行OxLoader导入元数据。
    2. `j`生成Json建模文件：执行Excel到Json的转换，第一次可能runtime中没有json目录，那么需要手工创建两个目录，`runtime/json/model`和`runtime/json/schema`。
2. 使用`q`命令退出控制台，然后回到`ox-engine`主目录，重新编译一次`ox-engine`（该操作只会在建模发生变化或者第一次初始化项目的时候做，后续操作中不需要执行该操作），编译之前查看下边两个目录中是否包含了生成好的`Json建模文件`

    ```shell
     # ox-driver/ix-atlantic/src/main/resources 目录下
     runtime/json/model                # 不为空
     runtime/json/schema               # 不为空
    ```
4. 编译好了过后之下下边命令`i`：初始化`DB_IOP`的初始化仓库（**初始化之前必须保证生成的Json建模文件已经编译过一次**，步骤`6`）。
5. 在`7`中的步骤结束过后，需要检查数据表是否生成完成，主要检查`DB_IOP`这张表中的`CI_*`的表是否生成，如果生成完成，则初始化任务执行完成。

### 3.5. 启动容器

上述所有步骤完成后，就可以直接启动 `OxAgent` 来启动容器，容器启动过后，可以看到详细的日志输出，看到如下输出就证明容器启动完成了（IP地址根据每个人的环境有所区别）：

```shell
[ μηδέν ] ( Http Server ) ZeroHttpAgent Http Server has been \
    started successfully. Endpoint: http://198.18.58.127:6083/.
```

启动完成过后，在前端刷新界面可以看到主登录界面，然后使用账号和密码登录（联系作者），登录过后就可以看到主界面。

>  在浏览器中安装 Resize 插件，使用客户的标准分辨率打开（1280 x 1024）。

## 4. 测试环境

`ox-engine`项目中提供了 `Post-man` 的测试环境，配置文件位于下边路径中：

```shell
document/configuration/post-man/
# Collection资源接口配置文件
-Ἀτλαντὶς νῆσος测试.postman_collection.json 
# 环境变量专用文件
OX专用最新测试环境.postman_environment.json 
# 安全认证专用前置脚本
pre-script.js
```

### 4.1. 账号修改

将上述文件分别导入到 `Post-man`中，如果需要使用特殊账号，则在环境中更改（右上角）。

![](/assets/images/ox/002-14.png)

在环境中修改 Client 的 ID即可：
![](/assets/images/ox/002-15.png)

### 4.2. 特殊账号统一

点击发送接口的 `pre-script` 标签确认接口和账号保持一致

![](/assets/images/ox/002-16.png)




