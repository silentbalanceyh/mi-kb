---
title: 02.新测试框架的使用
---

&ensp;&ensp;&ensp;&ensp;新版测试框架位于`zero-vie`的**脚手架**项目中，原始项目祛除了底层`ox-platform`项目，将通用型不带业务的内容全部移动到了`zero-vie`中，整个测试框架教程如下：

> 包名：`cn.originx.quiz`

# 1. 内容概览

## 1.1. 测试基类

&ensp;&ensp;&ensp;&ensp;根据测试目的的不同，整个环境中包含如下测试基类（包`cn.originx.quiz`）。

|类名|用途|
|---|:---|
|AbstractPlatform|顶层测试基类，底层数据访问测试以及标准模块测试专用。|
|AbstractCommand|Shell命令工具测试专用。|
|AbstractMigration|升级工具测试专用，测试Migrate和MigrateStep。|
|AbstractChannel|通道测试专用基类，测试API和Job。|

## 1.2. 环境综述

> `io.vertx.up.eon.em.Environment`常量，表格中o表示支持，*表示默认值

&ensp;&ensp;&ensp;&ensp;各类环境以及基类支持表格如下：

||Development|Production|Mockito|
|---|---|---|---|
|Platform|o|o|o *|
|Channel|o|o|o *|
|Command|o *|o||
|Migration|o|o|o *|

&ensp;&ensp;&ensp;&ensp;如果子类在测试过程中想要切换环境可书写如下：

```java
// 该类为测试类
public class DemoMigrateTc extends AbstractMigration {
    public DemoMigrateTc() {
        super(Environment.Development);     // 提供环境信息
    }
    @Test
    public void tcX(){
        // 测试类
    }
}
```

## 1.3. 核心数据结构

#### 1.3.1. CMDB动态模型

* 类名：`cn.originx.quiz.atom.QModel`

&ensp;&ensp;&ensp;&ensp;该结构对应的Json数据格式如：

```json
{
    "identifier": "模型统一标识符",
    "key": "数据记录主键",
    "data": {
    }
}
```

&ensp;&ensp;&ensp;&ensp;`data`为模型中的数据部分。

### 1.3.2. Web请求模型

* 类名：`cn.originx.quiz.atom.QRequest`

&ensp;&ensp;&ensp;&ensp;该结构对应的Json数据格式如：

```json
{
    "request": "POST /api/xxx",
    "data": {}
    "headers": {

    }
}
```

&ensp;&ensp;&ensp;&ensp;`data`为请求中的数据部分，如果是JsonObject则使用`{}`，如果是JsonArray则使用`[]`。

### 1.3.3. 环境模型

&ensp;&ensp;&ensp;&ensp;环境模型使用了新的**甲方乙方**模型，**甲方**就是我们面对的直接甲方，**乙方**就是第三方，如DDI、堡垒机等集成方，模型结构如下：

![](/assets/images/ox/2021-10-15-14-35-44.png)

* 测试用例以**甲方**为中心，且只包含一个甲方，如CMDB应用
* 可同时测试多个**乙方**（第三方），如堡垒机和UCMDB同时测试

### 1.3.4. 基本测试规范

&ensp;&ensp;&ensp;&ensp;基本测试规范整体如下：

1. 所有的测试类名以`Tc`结尾，全称为`TestCase`（不如此做无法运行概不负责）。
2. 测试的方法名以`tc`为前缀，如`tcEnvironment`,`tcExtension`等。
3. 测试的数据文件路径位于`src/test/resources/test/`目录中，您的测试类的包是什么，那么就在该目录中创建对应的子目录。
4. 具体详细内容可参考`qz-atlantic`项目。

# 2. 获取对象的API

&ensp;&ensp;&ensp;&ensp;获取对象的API分两类：

* 测试类：以tc开头，可直接运行代码逻辑以及测试结果。
* 工具类：获取所有相关数据对象。

> 不解释测试参数，基类方法子类可调用

## 2.1. 测试类方法

|方法名|类|含义|
|---|:---|:---|
|tcDao|AbstractPlatform|测试AoDao执行动态建模数据库访问。|
|tcRun|AbstractCommand|执行Shell中的子系统命令。|
|tcApi|AbstractChannel|API接口组件专用测试。|
|tcJob|--|JOB任务组件专用测试。|
|tcStep|AbstractMigration|升级步骤专用测试。|
|tcAop|--|升级过程中的AOP组件专用测试。|
|tcBackup|--|备份测试。|
|tcRestore|--|还原测试。|

## 2.2. 工具类方法

> 甲乙方配置自己参考对应的代码，都是获取数据专用。

|方法名|类|含义|
|---|:---|:---|
|app()|AbstractPlatform|读取JtApp应用配置。|
|database()|--|读取数据库配置。|
|atom(String)|--|根据传入identifier构造DataAtom模型定义。|
|dbBuilder()|--|「数据库」元数据构造器。|
|dbConnection()|--|「数据库」原生数据连接。|
|dbDao(String)|--|「数据库」根据identifier构造数据访问器。|
|es()|--|「Es」构造Es原生访问器。|
|es(ChangeFlag,String)|--|「Es」构造EsIndex引用。|
|neo4j()|--|「Neo4j」构造Neo4j原生访问器。|
|neo4P()|--|「Neo4j」构造Neo4j绘图仪。|
|ioOut(String,JsonObject)|--|输出数据写文件专用。|
|inData|--|读取数据文件构造QModal。|
|inWeb|--|读取数据文件构造QRequest。|

> 下边的方法不属于测试框架，是Zero Framework原生的。

## 2.3. Zero原生方法

|方法名|含义|文件格式|
|---|:---|:---|
|tcAsync|异步测试。|x|
|logger|记录日志专用，打印输出也用该方法。|x|
|ioJArray|读取数据文件，转换成JsonArray。|JsonArray|
|ioString|读取数据文件，转换成String。|任意|
|ioBuffer|读取数据文件，转换成Buffer。|二进制|
|ioJObject|读取数据文件，转换成JsonObject。|JsonObject|
|ioCriteria|读取数据文件，转换成Criteria。|JsonObject|
|ioDatabase|读取数据文件，转换成Database。|JsonObject|
|ioIntegration|读取数据文件，转换成Integration.|JsonObject|

# 3. 测试用例示例

## 3.1. 测试AoDao

```java
package cn.originx.cmdb.component.dao;

import cn.originx.quiz.AbstractPlatform;
import io.vertx.ext.unit.TestContext;
import org.junit.Test;

public class DaoTc extends AbstractPlatform {

    @Test
    public void tcDao(final TestContext context) {
        this.tcDao(context,
            (model, dao) -> dao.fetchOneAsync(model.dataJ()),   // 执行代码
            actual -> {
                // 测试结果回调
                System.out.println(actual.toJson());
            // test/cn.originx.cmdb.component.dao/data.json
            }).accept("data.json");
    }
}
```

## 3.2. 升级工具测试

```java
package cn.originx.cmdb.migrate;

import cn.originx.migration.backup.EnvPath;
import cn.originx.quiz.AbstractMigration;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.unit.TestContext;
import io.vertx.up.eon.em.Environment;
import org.junit.Test;

public class DemoMigrateTc extends AbstractMigration {
    public DemoMigrateTc() {
        super(Environment.Development);     // 必须提供环境信息
    }

    @Test
    public void tcPath(final TestContext context) {
        // 执行 MigrateStep ( EnvPath )
        final JsonObject params = this.ioJObject("input.json");
        this.tcAsync(context, this.tcStep(EnvPath.class, params), actual -> {
            System.out.println("Environment!");
        });
    }
}
```

## 3.3. 命令测试

```java
package cn.originx.cmdb.shell;

import cn.originx.quiz.AbstractCommand;
import io.vertx.ext.unit.TestContext;
import org.junit.Test;

public class DemoCommandTc extends AbstractCommand {
    @Test
    public void tcCommand(final TestContext context) {
        this.tcRun(context, "d,p");
    }
}
```

&ensp;&ensp;&ensp;&ensp;参数语法：

```shell
<subsystem>,<command>
<subsystem>为子系统名
<command>为命令名
```

## 3.4. Api/Job组件测试

```java
package cn.originx.ddi;

import cn.originx.quiz.AbstractChannel;
import io.vertx.ext.unit.TestContext;
import org.junit.Test;

public class JobTc extends AbstractChannel {
    @Test
    public void testTc(final TestContext context) {
        this.tcAsync(context, this.tcJob("ddi-sync.rng.once"), actual -> {
            System.out.println(actual);
        });
    }
}
```

&ensp;&ensp;&ensp;&ensp;参数语法：


* 对任务而言，参数为任务数据库中存储的任务的`code`。
* 对API而言，为数据库中存储的`method + uri`（近似于HTTP请求格式）。
