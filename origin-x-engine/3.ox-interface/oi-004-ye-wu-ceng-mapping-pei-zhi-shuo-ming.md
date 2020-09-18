# 业务层 Mapping

本文用于描述业务层的其中一个核心配置：映射配置（mapping）。

## 1. 配置字段

### 1.1. 配置表

> `mapping`的配置位于 `I_SERVICE`表中（zero-jet）

| 参数字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| mappingConfig | 字符串（JsonObject格式） | mapping 的核心配置信息 |
| mappingMode | 枚举值 | mapping 的模式 |
| mappingComponent | 字符串（Java类） | mapping 的自定义组件 |

## 2. 配置说明

映射配置中的基本原则：

* 左值不论在主动通道还是被动通道中，都是Ox端的字段。
* 右值用于对接第三方或者其他系统。

### 2.1. mappingConfig

Ox中的 mapping 配置主要配置字段信息的映射层，它的主要文件格式如下（mappingConfig字段内容）：

```json
{
    "ci.application.ba" : {
        "code" : "z_cicode",
        "name" : "z_ciname"
    },
    "approvalStatus": "z_approvalstatus",
    "category": "z_category"
}
```

mapping 的配置依靠三个核心的 `Zero` 类：

* `io.vertx.up.commune.config.DualMapping`类（映射管理器）
* `io.vertx.up.commune.config.DualItem`类（映射实例类）
* `io.vertx.up.commune.config.DualType`类（映射类型定义器）

整个映射配置会被解析成一个唯一的 `DualMapping`类，而一个`DualMapping`中会包含一个根`DualItem`和一个`ConcurrentMap<String, DualMapping>`的哈希表，表中存储了子结构，注意整个映射层只包含两层。

* 所有非`JsonObject`的属性（key = value），键值都是String，构成根`DualItem`。
* 每一个`JsonObject`的属性构成一个\( key = DualItem）的结构。

> DualItem是一个双通道结构，包含了 from 和 to 以及 fromType 和 toType 的相关属性。

### 2.2. mappingMode

Ox中的 mapping 模式主要分为四种，主要告诉Ox是否自动处理 mapping中的映射信息

| 模式 | 说明 |
| :--- | :--- |
| NONE | 不启用自动 mapping |
| BEFORE | 启用前置 mapping |
| AFTER | 启用后置 mapping |
| AROUND | 前后都启用 mapping |

### 2.3. mappingComponent（后期扩展）

自定义组件需要遵循一定的定义规则，目前的版本暂时不考虑`mappingComponent`的定义，在当前这个版本中，映射配置对应的自定义组件是一个保留值，用于后期扩展。

## 3. 执行原理

### 3.1. 映射处理器

映射处理类实现了 `io.vertx.up.uca.adminicle.Mapper`接口，目前系统中包含了一个内部实现类：`io.vertx.up.uca.adminicle.FieldMapper`，该类用于处理映射过程中的字段转换，该接口的定义如下：

```java
public interface Mapper {
    /*
     * Mapping
     * to -> from
     */
    JsonObject in(JsonObject in, DualItem mapping);

    /*
     * Mapping
     * from -> to
     */
    JsonObject out(JsonObject out, DualItem mapping);
}
```

上述方法为双通道方法，mapping 配置中的 `Json`格式左值为键（From端），右值为值（To端），上边方法中，in 负责从右往左边转换，而 out 负责从左往右边转换。

### 3.2. 值偏移

> 值偏移器是 Ox 引擎中定义的，不属于 Zero 中的定义，用于处理特殊的字段值偏移处理，属于：兼容性规则

值偏移器位于 `cn.originx.uca.change` 包中，所有的类都实现了：`cn.originx.uca.change.Adjuster`接口，值偏移器是根据数据的类型来选择的，目前支持的值偏移器表如下：

| 字段类型 | Adjuster 类 | 说明 |
| :--- | :--- | :--- |
| String | StringAj | 字符串处理 |
| Integer | IntegerAj | 整数处理 |
| Long | LongAj | 长整数处理 |
| Boolean | BooleanAj | 布尔值处理 |
| BigDecimal | BigDecimalAj | 带精度浮点数处理 |
| LocalTime | LocalTimeAj | 时间处理 |
| LocalDate | LocalDateAj | 日期处理 |
| LocalDateTime | LocalDateTimeAj | 日期/时间处理 |

### 3.2. 通道配合

#### 3.2.1. 通道分类

Ox的Engine中包含了四种通道

| 通道类型 | 主动/被动 | Database数据库 | Integration集成 | Mission任务配置 |
| :--- | :--- | :--- | :--- | :--- |
| ADAPTOR | 被动 | 支持 | 不支持 | 不支持 |
| CONNECTOR | 被动 | 支持 | 支持 | 不支持 |
| DIRECTOR | 主动 | 支持 | 不支持 | 支持 |
| ACTOR | 主动 | 支持 | 支持 | 支持 |

#### 3.2.2. 通道支持矩阵

|  | BEFORE | AFTER | AROUND |
| :--- | :--- | :--- | :--- |
| 被动 | to -&gt; from | from -&gt; to | 支持 |
| 主动 | to -&gt; from | from -&gt; to | 支持 |

> 映射配置（mapping）只针对 Channel 中的 Component 生效，在任务管理模式中，如果启用了 JobIncome/JobOutcome，这两个组件只能拿到映射管理器DualMapping ，不会触发 Ox Engine 中的自动映射功能。



