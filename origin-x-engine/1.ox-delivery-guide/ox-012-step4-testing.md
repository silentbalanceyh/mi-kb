# 第四步：最终测试

在`Origin X`开发中，它提供了专用的单元测试平台，方便开发人员模拟测试书写的API接口，整个测试结构如下：

| 基类 | 测试类 | 说明 |
| ---: | ---: | ---: |
| AsyncBase | QzPlatform | 平台测试专用 |
| QzPlatform | QzMigrate | 自动升级程序专用测试 |
| QzPlatform | QzDao | 动态数据库访问专用测试 |
| QzPlatform | QzGraphic | 图引擎Neo4j专用测试 |
| QzPlatform | QzMetadata | 元数据库建模引擎专用测试 |
| QzPlatform | QzChannel | 任务/接口专用测试 |

>  本文只枚举常用测试，主要集中在介绍`QzChannel`的测试中。

## 1. 常用测试

### 1.1. Migration插件测试

假设你书写了一个Migration插件如下：

```java
public class BackupUser extends AbstractStatic {

    public BackupUser(final Environment environment) {
        super(environment);
    }
    // 主代码
}
```

>  Migration 插件不能带测试输入，只设置环境 Environment = Development 或 Production 值。

测试步骤如下，直接书写测试类，然后直接运行该测试类：

```java
package cn.originx.migration;

import cn.originx.migration.backup.BackupHistory;
import cn.originx.quiz.QzMigrate;
import io.vertx.ext.unit.TestContext;
import org.junit.Test;

public class UpgradeTc extends QzMigrate {

    @Test
    public void testBackUpStep2(final TestContext context) {
        this.async(context, this.stepAsync(BackupHistory.class), actual -> {
            System.out.println("第二步：补充备份");
        });
    }
}
```

### 1.2. 通道测试：Api/Job

测试通道之前保证您已经完成了环境配置：[任务和接口配置](ox-009-di-yi-bu-ff1a-ren-wu-he-jie-kou-pei-zhi.html)，并且开发已经完成，然后书写测试类。

#### 1.2.1. 测试类

```java
package cn.originx.plugin.bastion.component;

import cn.originx.quiz.QzChannel;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.unit.TestContext;
import org.junit.Test;

public class ActorJobTc extends QzChannel {

    @Test
    public void testJob(final TestContext context) {
        this.async(context, this.startJob("bastion-sync.insert.once"), actual -> {
            final JsonObject data = actual.data();
            System.err.println(data.encodePrettily());
        });
    }

    @Test
    public void testApi(final TestContext context) {
        this.async(context, this.startApi("fetch.ci.json"), actual -> {
            final JsonObject data = actual.data();
            System.err.println(data.encodePrettily());
        });
    }
}
```

#### 1.2.2. 前提说明

Job一般没有输入，所以不需要任何前置条件就可以测试，直接从`QzChannel`继承，然后调用父类的`startJob`方法，传入`I_JOB`中配置的`CODE`字段数据信息。

Api需要额外的输入，借用Zero中的测试框架，在测试资源目录`test/<package>/`中放置测试文件，文件格式如：

```json
{
    "request": "GET /api/ox/ci.device/:key",
    "data": {
    },
    "headers":{
    }
}
```

数据格式中的属性说明如：

| 项 | 值说明 |
| ---: | ---: |
| request | 请求键，数据库中存储的信息，包括方法和路径。 |
| data | 请求数据。 |
| headers | 请求头数据。|

## 2. 环境数据获取

所有的测试类都是从顶层基类`QzPlatform`中继承而来，接下来介绍如何读取环境数据相关信息。

### 2.1. 应用、数据库、集成

应用程序信息可以直接调用下边的方法：

>  当前版本中，应用、数据库、集成三个环境只能使用 runtime/external/once 目录下的三个文件，属于不可配的情况。

```java
    // 读取配置：runtime/external/once/application.json
    protected JtApp app() {
        return this.connector.configApp();
    }
    protected String sigma() {
        return this.app().getSigma();
    }
    // 读取配置：runtime/external/once/database.json
    protected Database database() {
        return this.connector.configDatabase();
    }
    // 读取配置：runtime/external/once/integration.json
    protected Integration integration() {
        return this.connector.configIntegration();
    }

    /**
     * 动态集成环境，可读取额外的集成信息，用于底层操作
     * -- 可读取特殊的集成配置信息，默认集成到 UCMDB 中
     * -- 默认的调用 this.integration()
     **/
    protected Integration integration(final String filename) {
        final Integration integration = new Integration();
        integration.fromJson(this.ioJObject(filename));
        return integration;
    }
```

### 2.2. 直接环境

　　直接环境包括：字典配置、服务配置、映射配置和字典消费配置，这几个配置位于：

```shell
# 测试资源目录中
# src/resources/test/channel/cmdb-v2/ 
dict-config: 字典配置
dict-epsilon：字典消费配置
mapping：映射配置
options：服务选项配置
```

　　这些配置根据不同的文件名会有所区别，您可以调用下边的方法对当前配置进行切换。

#### 2.2.1. 构造时处理（单通道）

>  整个项目默认使用 ucmdb.json 文件

```java
// 构造过后绑定的文件路径
// src/resources/test/channel/cmdb-v2/dict-config/ucmdb.json
// src/resources/test/channel/cmdb-v2/dict-epsilon/ucmdb.json
// src/resources/test/channel/cmdb-v2/mapping/ucmdb.json
// src/resources/test/channel/cmdb-v2/options/ucmdb.json
public class QzDao extends QzPlatform {
    public QzDao() {
        super("ucmdb");
    }
}
```

#### 2.2.2. 动态切换（多通道）

```java
    // 在你执行代码过程中调用（切换到另外一个通道中执行）
    // src/resources/test/channel/cmdb-v2/dict-config/ci.json
    // src/resources/test/channel/cmdb-v2/dict-epsilon/ci.json
    // src/resources/test/channel/cmdb-v2/mapping/ci.json
    // src/resources/test/channel/cmdb-v2/options/ci.json
    this.connect("ci");
```

## 3. 测试模型

测试模型主要用于创建特殊的数据结构。

### 3.1. QuizInput

```java
    final QuizInput input = this.input("filename.json");
```

数据结构如：

```json
{
    "identifier": "模型ID",
    "data": "可以是JsonObject，也可以是JsonArray"
}
```

### 3.2. QuizRequest

```java
    final QuizRequest request = new QuizRequest(this.ioJObject("filename.json"));
```

数据结构如：

```json
{
    "request": "GET /api/ox/ci.device/:key",
    "data": {
        "key": "00031bb6-649d-417e-bf73-24941c77f17a"
    }
}
```
