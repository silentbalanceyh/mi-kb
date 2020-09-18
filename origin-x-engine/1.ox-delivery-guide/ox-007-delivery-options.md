# Options配置

>  xx包名为特殊的客户包名

在[核心数据结构](/origin-x-engine/1.ox-delivery-guide/ox-006-jie-kou-kai-fa.html)中，有一类比较特殊的配置就是`options`配置，该配置是一个`JsonObject`的数据格式，但是在`Origin X`中他存在一些特殊的配置信息实现配置扩展，这些配置并非标准，但可以作为`Origin X`中的一种约定，Options的配置主要对应于`I_SERVICE`表中的`serviceConfig`字段。

## 1. 基本配置

### 1.1. Todo基础配置

Todo配置在任务配置中，每天会有Daily的任务运行，从UCMDB拉取相关的配置项数据，并且生成确认单，并且将确认单和生命周期管理器关联（ITSM工作流？CMDB内部工作流），它的配置如下：

```json
{
    "plugin.todo": "cn.originx.xxx.extension.AspectTodo"
}
```

### 1.2. 变更历史配置

在系统的添加、删除、查询修改过程中，每一个操作都会在系统底层生成相关的变更历史，存储在：`X_ACTIVITY, X_ACTIVITY_CHANGE`表中，它记录了某个用户对于配置项的所有操作，它的核心配置如下：

```json
{
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

## 2. AOP专用配置

通道和插件的结构是整个`Origin X`的核心结构，这部分会在下一章节说明，先看看通道中的AOP插件配置，看看一个案例：

```json
{
    "configuration.operation": "ADD",
    "plugin.component.before": [
        "cn.originx.uca.plugin.semi.BeforeNumber",
        "cn.originx.uca.plugin.semi.BeforeLife"
    ],
    "plugin.component": "cn.originx.scaffold.plugin.AspectBatch",
    "plugin.component.after": [
        "cn.originx.uca.plugin.semi.AfterEs",
        "cn.originx.itsm.plugin.AfterItsm"
    ],
    "plugin.config": {
        "cn.originx.uca.plugin.semi.BeforeNumber": {
            "field": "code"
        }
    },
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

上述插件中，除开`plugin.activity`是第二章节提到的变更历史处理插件，其他的配置就是AOP插件的核心配置。

### 2.1. 操作类型

`configuration.operation`是操作类型的配置，它表示当前通道的核心操作，由于会在一些特殊场景处理**操作分流**的问题，所以它属于必须和当前通道绑定的`options`配置，这个配置包含了三个值：

* ADD：添加操作通道
* UPDATE：编辑操作通道
* DELETE：删除操作通道

### 2.2. 核心组件

核心组件（`plugin.component`）可以自己实现，也可以使用标准化的核心组件，目前标准化的核心组件如下：

| 插件名称 | 含义 |
| :--- | :--- |
| cn.originx.scaffold.plugin.AspectBatch | 批量操作插件 |
| cn.originx.scaffold.plugin.AspectRecord | 单记录操作插件 |

核心组件配置在options中的`plugin.component`节点中，该节点配置了核心组件的类名，如果您对AOP核心组件开发不熟悉，可以选择上述两种——不过需要分清楚当前通道处理的是单条数据还是批量数据。

### 2.3. 执行流程

节点：`plugin.component.before`是一个`JsonArray`的数组，它包含了当前通道要触发的所有前置插件链。节点：`plugin.component.after`同样是一个`JsonArray`的数组，它包含了当前通道要触发的所有后置插件链。整个**通道 + 插件**的执行流程如下：

1. Before阶段（序号生成器）：
     执行`cn.originx.uca.plugin.semi.BeforeNumber`插件
2. Before阶段（生命周期切换器）：选择ITSM或CMDB工作流
     执行`cn.originx.uca.plugin.semi.BeforeLife`插件
3. **待开发阶段**：执行开发人员开发的通道代码
4. After阶段（ES索引插件）：
     执行`cn.originx.uca.plugin.semi.AfterEs`插件
5. After阶段（ITSM推送插件）：
     执行`cn.originx.itsm.plugin.AfterItsm`插件

上述所有的插件中，每个插件执行完了过后，会把处理掉的数据：`JsonObject / JsonArray`直接传递给下一个插件，并且会把`Options`也传递给下一个插件，形成一个完整的**函数链**。这些插件都实现了接口：

```java
public interface AspectPlugin extends DataPlugin<AspectPlugin> {

    @Override
    default AspectPlugin bind(final DataAtom atom) {
        return this;
    }

    @Override
    default AspectPlugin bind(final DictFabric fabric) {
        return this;
    }

    /*
     * 前置函数
     */
    default Future<JsonObject> beforeAsync(final JsonObject record, final JsonObject config) {
        return Future.failedFuture(new _501NotSupportException(this.getClass()));
    }

    default Future<JsonArray> beforeAsync(final JsonArray records, final JsonObject config) {
        return Future.failedFuture(new _501NotSupportException(this.getClass()));
    }

    /*
     * 后置函数
     */
    default Future<JsonObject> afterAsync(final JsonObject record, final JsonObject config) {
        return Future.failedFuture(new _501NotSupportException(this.getClass()));
    }

    default Future<JsonArray> afterAsync(final JsonArray records, final JsonObject config) {
        return Future.failedFuture(new _501NotSupportException(this.getClass()));
    }
}
```

每个插件中都包含了绑定的`DataAtom`以及当前通道需要使用的字典翻译器`DictFabric`。

### 2.4. 插件配置

在整个配置中，还存在以下片段：

```json
    "plugin.config": {
        "cn.originx.uca.plugin.semi.BeforeNumber": {
            "field": "code"
        }
    }
```
这个片段中存储了每个配置的插件不同的配置信息，如这里可以看到的插件：`cn.originx.uca.plugin.semi.BeforeNumber`的配置中会包含`field = code`的配置信息，这种格式存在的限制：

1. 每个通道中某一类的插件只能使用一次。
2. 使用的时候配置是静态的，不过可以通过更新`I_SERVICE`的`serviceConfig`来动态切换。

## 3. Options的默认结构

### 3.1. 通道中的options

通道中的options配置在`I_SERVICE`的 `serviceConfig` 字段中，它的默认结构如下：

```json
{
    "name": "应用程序名称，X_APP中的NAME字段",
    "sigma": "多租户场景中的统一标识 sigma",
    "identifier": "模型标识符"
    ...
}
```

上述配置中，`...`省略的配置就是`serviceConfig`中的配置，它和默认配置合并到一起，也方便开发人员拿到合法的上下文环境。

### 3.2. 插件中的options

插件中的options在前者中会有稍许的变化，如：

```json
{
    "configuration.operation": "ADD",
    "name": "vie.app.ox",
    "sigma": "xxx",
    "identifier": "ci.server.pcs",
    ...
}
```

有几点需要说明：

1. `identifier`和`serviceConfig`中配置的`identifier`可能不相等，如果使用了标识规则。器，`identifier`有可能会存在变化。插件中的`identifier`是变化过后的值——简单说插件的options中的`identifier`是经过了**标识选择器**执行后的`identifier`。
2. `name`和`sigma`继续维持所需要的值。
3. 保留了操作标识：`configuration.operation`（三个值，ADD、UPDATE、DELETE）。
4. `...`中就是`plugin.config`定义的Json结构。
5. 所有`plugin`前缀的键值配置会被自动移除掉。

## 4. 通道 / 插件架构

同一个功能，可以使用**通道 / 插件**两种模式实现，前提在于你所编写的接口或者任务的存在位置，这里留个引用提供给读者参考。

### 4.1. UCMDB通道 + ITSM插件

* 接口开放项目：**infix-ucmdb**
* 通道代码：`cn.originx.optic.advanced.CreateComponent`
* 插件代码：`cn.originx.itsm.plugin.AfterItsm`
* 通道中的插件配置：

```json
{
    "configuration.operation": "ADD",
    "plugin.component.before": [
        "cn.originx.uca.plugin.semi.BeforeNumber",
        "cn.originx.uca.plugin.semi.BeforeLife"
    ],
    "plugin.component": "cn.originx.scaffold.plugin.AspectRecord",
    "plugin.component.after": [
        "cn.originx.uca.plugin.semi.AfterEs",
        "cn.originx.itsm.plugin.AfterItsm"
    ],
    "plugin.config": {
        "cn.originx.uca.plugin.semi.BeforeNumber": {
            "field": "code"
        }
    },
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

### 4.2. ITSM通道 + UCMDB插件

* 接口开放项目：**infix-itsm**
* 通道代码：`cn.originx.itsm.api.OnlineComponent`
* 插件代码：`cn.originx.ucmdb.plugin.semi.BeforeUcmdb`
* 插件配置：

```json
{
    "configuration.operation": "UPDATE",
    "plugin.component.before": [
        "cn.originx.ucmdb.plugin.semi.BeforeUcmdb",
        "cn.originx.xx.extension.BeforeOwner"
    ],
    "plugin.component": "cn.originx.scaffold.plugin.AspectBatch",
    "plugin.component.after": [
        "cn.originx.uca.plugin.semi.AfterEs"
    ],
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

### 4.3. 总结

综上所述：

* 通道组件（开发人员开发的核心组件）在哪个项目中，那么接口和任务的主入口就在那个项目里。
* 插件只能以功能的形式挂载到通道中，不可独立运行。
* 插件可以环绕于通道周围，配置成**Before，After，Around**三种。
