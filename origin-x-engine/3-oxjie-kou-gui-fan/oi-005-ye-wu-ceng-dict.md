# 业务层 Dict

本文用于描述业务层的其中另一个核心配置：字典配置（dict）。

## 1. 配置字段

### 1.1. 配置表

> dict 的配置位于 `I_SERVICE`表中（zero-jet）

| 参数字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| dictConfig | 字符串（JsonArray格式） | dict 字典定义配置 |
| dictComponent | 字符串（Java类） | dict 的自定义组件 |
| dictEpsilon | 字符串（JsonObject格式） | dict 字典的消费字段配置 |

> dictEpsilon 的消费字段配置仅使用左值（平台内的）字段名，不使用任何第三方的配置字段名。

## 2. 配置说明

### 2.1. dictConfig

#### 2.1.1. 配置示例

`dictConfig`中配置了当前通道组件需要使用的所有字典信息，目前不支持延迟加载模式，它的格式如下：

```json
[
    {
        "source": "TABULAR",
        "types": [
            "ci.environment",
            "ci.status",
            "ci.approval.status",
            "remove.mode",
            "add.mode",
            "level.info.system",
            "level.protect"
        ]
    },
    {
        "source": "CATEGORY",
        "types": [
            "ci.type"
        ]
    },
    {
        "source": "ASSIST",
        "key": "ci.relation.mapping",
        "component": "cn.originx.ucmdb.plugin.tunnel.DictRelationDef"
    }
]
```

dict 的配置依赖四个核心的 Zero 类：

* `io.vertx.up.commune.config.Dict`：（定义）对应该配置的核心字典定义。
* `io.vertx.up.commune.config.DictSource`：（定义）对应该配置中的每一个元素（JsonObject）的定义。
* `io.vertx.up.commune.config.DictEpsilon`：（定义）对应某一个字段的消费定义。
* `io.vertx.up.commune.config.DictFabric`：字典翻译器，根据字典翻译当前的数据专用执行器。

#### 2.1.2. 配置说明

dictConfig 是一个JsonArray的字符串，定义了所有当前通道需要使用的字典，主要分为：**标准字典** 和 **扩展字典**两种，而标准字典中又包含了平行列表和树形列表，整个字典定义最终会转换成数据结构：`ConcurrentMap<String,JsonArray>`来存储字典数据。每个字典的属性定义如下：

| 属性名 | 支持 | 说明 |
| :--- | :--- | :--- |
| source | 标准字典 / 扩展字典 | 字典类型 |
| types | 标准字典 | 标准字典中的types的每一个元素会转换成上述数据结构中的键 |
| key | 扩展字典 | 单个元素转换成上述数据结构中的键 |
| component | 扩展字典 | Java类名，所有扩展字典需要提供该类，该类实现接口：DictionaryPlugin |

#### 2.1.3. 分类说明

* TABULAR：标准字典中的平行列表，types 对应到 `X_TABULAR`中的 `type` 字段。
* CATEGORY：标准字典中的属性列表，types 对应到 `X_CATEGORY`中的 `type` 字段。
* ASSIST：由用户提供自定义字典插件实现，这种配置中 component 是必须的，并且是实现了字典插件接口的类。

### 2.2. dictComponent

> 字典组件规范

* 所有的 dictComponent 组件都实现 `io.vertx.tp.optic.component.Dictionary`接口。
* 框架中已经提供了默认的 dictComponent 组件，值为：`io.vertx.tp.optic.business.ExAmbientDictionary` 类。
* 每一个 `Dictionary` 的实现类对应到 dictConfig 的整体配置中，为一个 `JsonArray`格式，表示多个字典，而该类内部调用了：`io.vertx.tp.optic.component.DictionaryPlugin`插件，该插件用于处理某一个字典来源信息（只针对ASSIST）。

在字典扩展中，主要包含两层：

1. 如果没有特殊的字典需求，可以直接从 `DictionaryPlugin` 扩展，配置到 `ASSIST` 类型的字典中。
2. 如果有更加复杂的字典需求，则可替换掉原始的 `Dictionary`类，直接从该类扩展。

### 2.3. dictEpsilon

#### 2.3.1. 配置示例

该字段为字典消费配置，格式如：

```json
{
    "status": {
        "source": "ci.status",
        "in": "name",
        "out": "key"
    },
    "environment": {
        "source": "ci.environment",
        "in": "name",
        "out": "key"
    }
]
```

上述片段定义了模型中两个字段的消费信息：

* `status`字段消费名为`ci.status`的字典信息，传入数据为字典元素对象的`name`字段的值，当前对应字段为`key字段`。
* `environment`字段消费名为`ci.environment`的字典信息（出入配置同上）。

#### 2.3.2. 配置说明

上述配置中，`status`和`environment`均为当前模型的字段信息，它可以是

* 静态建模中的Pojo类的属性名。
* 动态建模中的Record类的属性名。

| 配置名 | 说明 |
| :--- | :--- |
| source | 字典名称，对应到 dictConfig 中生成的ConcurrentMap&lt;String,JsonArray&gt;数据结构中的键值。 |
| in | 第三方的传入值对应的字典字段名。 |
| out | Ox平台的最终翻译存储值。 |

## 3. 特殊说明

### 3.1. 翻译器使用

#### 3.1.1. 翻译器说明

字典配置中的终点是开发者需要理解如何使用：`DictFabric`类，它有两种模式：

* 单边模式：创建的时候不引入`DualItem`的映射配置，这种情况只能翻译平台内的属性。
* 双边模式：创建的时候引入 `DualItem`的映射配置，这种情况下可以同时翻译平台内和第三方的属性。

翻译器在翻译数据时，需要绑定两个核心数据结构：

* 字典数据（非定义）：`ConcurrentMap<String,JsonArray>`。
* 字典消费（定义）：`ConcurrentMap<String, DictEpsilon>`。

翻译器的初始化如下：

```java
// 单边字典翻译器
final DictFabric fabric = DictFabric.create()
    .dict(request.getDict())                    // 绑定 ConcurrentMap<String,JsonArray> 字典
    .epsilon(this.dict().getEpsilon());         // 绑定 ConcurrentMap<String,DictEpsilon> 定义

// 双边字典翻译器
final DictFabric fabric = DictFabric.create(mapping)
    .dict(dictMap)
    .epsilon(this.dict().getEpsilon());
```

#### 3.1.2. 翻译器核心API

|  | 单边翻译器 | 双边翻译器 |
| :--- | :--- | :--- |
| inTo | 支持 | 支持 |
| inFrom | 支持 | 支持 |
| outTo | 不支持 | 支持 |
| outFrom | 不支持 | 支持 |

> 字典的支持不会有副作用，也就是说，如果翻译失败，则直接不翻译传入的 JsonObject 或者 JsonArray，并不会在翻译失败的过程中报错，而且翻译出来的数据如果为 null 则会用 null 值填充，如果在 Epsilon 中的 `in / out`定义反向的话，那么对应的翻译器的API也会反向，请开发人员注意方向问题。

#### 3.1.3. 翻译示例

如 Mapping 的定义如下：

```json
{
    "name": "z_name"
}
```

字典数据如：

```json
{
    "ci.name":[
        {
            "value": "uuid1",
            "display": "display1"
        },
        {
            "value": "uuid2",
            "display": "display2"
        }
    ]
}
```

字典消费定义限定为：

```json
{
    "name": {
        "source": "ci.name",
        "in": "display",
        "out": "value"
    }
}
```

设置数据：

| 数据 | 编号 | 场景 |
| :--- | :--- | :--- |
| name = uuid1 | S11 | Ox中存储的值为 uuid1 |
| name = display1 | S12 | 从第三方传入的值为 display1 |
| z\_name = uuid1 | S21 | 拿到数据中为第三方字段 z\_name |
| z\_name = display1 | S22 | 拿到的数据为第三方字段 z\_name，存储的 display1 |

那么在上述流程中，看看调用不同的API的最终结果如：

| 调用API | 场景 | 翻译结果 |
| :--- | :--- | :--- |
| inTo | S11 | name = display1 |
| inFrom | S11 | 不翻译 |
| outTo | S11 | 不翻译 |
| outFrom | S11 | 不翻译 |
| inTo | S12 | 不翻译 |
| inFrom | S12 | name = uuid1 |
| outTo | S12 | 不翻译 |
| outFrom | S12 | 不翻译 |
| inTo | S21 | 不翻译 |
| inFrom | S21 | 不翻译 |
| outTo | S21 | z\_name = display1 |
| outFrom | S21 | 不翻译 |
| inTo | S22 | 不翻译 |
| inFrom | S22 | 不翻译 |
| outTo | S22 | 不翻译 |
| outFrom | S22 | z\_name = uuid1 |

#### 3.1.4. 最终支持表

| 场景描述 | 方向 | 调用Api |
| :--- | :--- | :--- |
| Ox端 | value -&gt; display | inTo |
| Ox端 | display -&gt; value | inFrom |
| 第三方 | value -&gt; display | outTo |
| 第三方 | display -&gt; value | outFrom |



