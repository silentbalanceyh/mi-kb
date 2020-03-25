# 元数据仓库初始化

由于Origin X Engine本身使用了TiDB存储元数据，用于保存核心的引擎数据，元数据类型主要包含：

* 系统表中存储的数据：附件、类型、列表、应用、数据源、模块。
* 安全表中存储的数据：用户、角色、组、权限、资源、行为。
* 模型定义数据：实体/关系表、字段、键、索引、属性、模型。
* 接口定义数据：接口、业务组件。
* 图定义的数据：图、类型、节点、边。
* 界面定义数据：布局、页面、插槽、组件、字段、列。

由于Origin X Engine依赖 [http://www.vertxup.cn](http://www.vertxup.cn) ，所以您需要拉取最新的Zero代码在本地编译：

```shell
git clone https://github.com/silentbalanceyh/vertx-zero.git
cd vertx-zero
mvn clean package install
```

元数据仓库TiDB的搭建可以参考：[END-002 TiDB环境](/environment/1-dockerrong-qi-huan-jing/end-002-tidbhuan-jing-chu-shi-hua.html)。还有一点需要注意的是，您需要在您的hosts地址中添加如下：

```shell
127.0.0.1   ox.engine.cn
172.20.10.2 ox.engine.cn  # 根据你自己的环境来定前边的IP使用什么
```

这样处理的主要目的是方便所有开发人员在开发过程中统一使用一套配置文件。

## 1. 初始化


初始化数据库脚本可参考：`script/database/database-reinit.sh`内容（**记得修改mysql的路径**）：

```shell
#!/usr/bin/env bash
/usr/local/mysql/bin/mysql -u root -P 4000 -h 127.0.0.1 < database-reinit.sql
echo "[OX] 重建 TiDB 数据库成功!";
```

SQL内容如下：

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;
-- 删除原来的数据库
DROP DATABASE IF EXISTS DB_ORIGIN_X;
CREATE DATABASE IF NOT EXISTS DB_ORIGIN_X DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_bin;
```

创建好了过后，一个可连接的空库就初始化完成。

## 2. 业务数据库

业务数据库分为两种，这里不重述（MySQL和Oracle），MySQL基本信息如下：

| 项 | 备注 | 值 |
| :--- | :--- | :--- |
| hostname | 主机地址 | localhost |
| instance | 数据库实例名 | DB\_IOP |
| port | 数据库端口号 | 3306 |
| username | 用户名 | root |
| password | 你的MySQL密码 |  |

> 在执行脚本之前初始化。

## 3. 初始化环境

代码下载完成后，先执行不带测试用例的编译：

```shell
.../ox-engine >> mvn clean package install -DskipTests=true
```

编译成功过后，执行主项目下的liquibase工具创建表结构：

```shell
.../ox-engine/ox-driver/ix-atlantic >> ./run-init.sh
```

仓库初始化完成过后，可执行带测试用例的编译（第一次失败没关系，运行两次）：

```
.../ox-engine >> mvn clean package install
```

> 如果要一次成功可以先运行下边测试用例：
>
> **.../ox-engine/ox-driver/ix-atlantic/src/test/java/cn/originx/engine/service/InitTc**

## 4. 检查

全部执行完成过后，就可以启动了，保证最后多次运行带测试用例的编译通过。




