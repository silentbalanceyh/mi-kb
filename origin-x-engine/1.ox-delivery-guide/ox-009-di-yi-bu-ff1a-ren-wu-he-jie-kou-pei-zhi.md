# 第一步：任务和接口配置

>  所有表结构解析中不包含 zero extension 扩展模块中的系统型字段，主要是：`sigma, active, language, metadata, createdAt, createdBy, updatedAt, updatedBy`八个系统型字段。
> 所有的配置导入都在`ox-driver/ix-atlantic`项目中执行（主程序）

本文介绍接口和任务的相关配置，二者的配置组合如下：

| 组合 | 备注 |
| :--- | :--- |
| 接口 + 权限 | 接口配置、权限配置（`I_API, I_SERVICE`） |
| 任务 | 任务配置（`I_JOB, I_SERVICE`） |

## 1. 表结构解析

### 1.1. S_API 

`S_API`是接口表，用于定义将要部署的 RESTful 接口信息。

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 接口主键 | 新生成UUID |
| name | NAME | 接口名称 | 中文说明 |
| uri | URI | HTTP请求路径 | 不带`/api/`安全前缀和动态路由前缀`/ox` |
| method | METHOD | HTTP方法 | 四个值：GET,POST,PUT,DELETE |
| consumes | CONSUMES | 客户端MIME | JsonArray数组格式，MIME集合 |
| produces | PRODUCES | 服务端MIME | JsonArray数组格式，MIME集合 |
| secure | SECURE | 是否安全接口 | 安全接口自带`/api`前缀，非安全接口不带 |
| comment | COMMENT | 接口备注 | 接口详细描述 |
| type | TYPE | 接口类型 | 三种值：ONE-WAY, REQUEST-RESPONSE, PUBLISH-SUBSCRIBE |
| paramMode | PARAM_MODE | 传参类型 | 四个值：PATH, QUERY, BODY, DEFINE |
| paramRequired | PARAM_REQUIRED | 是否必须 | 400基本验证Query/Path |
| paramContained | PARAM_CONTAINED | 是否必须 | 400基本验证Body |
| inRule | IN_RULE | 验证、转换规则 | **保留** |
| inMapping | IN_MAPPING | 映射规则 | **保留** |
| inPlug | IN_PLUG | 插件规则 | **保留**，Java类名 |
| inScript | IN_SCRIPT | 脚本规则 | **保留** |
| outWriter | OUT_WRITER | 响应处理器 | 目前默认使用Json格式组件，Java类名 |
| workerType | WORKER_TYPE | Worker类型 | 三个值：JS, PLUG, STD |
| workerAddress | WORKER_ADDRESS | Worker消费地址 | 字符串格式 |
| workerConsumer | WORKER_CONSUMER | Worker消费组件 | Java类名 |
| workerClass | WORKER_CLASS | Worker类名 | Java类名 |
| workerJs | WORKER_JS | Js类型的worker文件 | JavaScript文件地址 |
| serviceId | SERVICE_ID | 服务层ID | 关联服务层记录主键 |

### 1.2. S_JOB

`S_JOB`是任务表，用于定义一个后端任务的基本信息

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 任务主键 | 新生成UUID |
| namespace | NAMESPACE | 任务所属名空间 | 使用`X_APP`生成 |
| name | NAME | 任务名称 | 中文格式 |
| code | CODE | 任务编码 | 通常以`task`前缀为主，不重复即可 |
| type | TYPE | 任务类型 | 三个值：ONCE, PLAN, FIXED |
| comment | COMMENT | 任务备注 | 配置任务的描述信息 |
| additional | ADDITIONAL | 附加配置 | 任务需要的额外配置信息 |
| runAt | RUN_AT | 定义任务中的JOB执行时间 | 时间格式，每天定时 |
| duration | DURATION | 任务间隔时间 | 两次任务执行的时差 |
| proxy | PROXY | 任务代理类 | Java类名 |
| threshold | THRESHOLD | 超时设置 | 默认300s，秒为单位 |
| incomeComponent | INCOME_COMPONENT | JobIncome组件 | Java类名 |
| incomeAddress | INCOME_ADDRESS | 输入数据地址 | 字符串 |
| outcomeComponent | OUTCOME_COMPONENT | JobOutcome组件 | Java类名 |
| outcomeAddress | OUTCOME_ADDRESS | 输出数据地址 | 字符串 |
| serviceId | SERVICE_ID | 服务层ID | 关联服务层记录主键 |

### 1.3. S_SERVICE

`S_SERVICE`是服务表，用于定义一个任务复杂的业务服务信息

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 服务主键 | 新生成UUID |
| namespace | NAMESPACE | 任务所属名空间 | 使用`X_APP`生成 |
| name | NAME | 服务名称 | 中文格式 |
| comment | COMMENT | 服务备注 | 服务的描述信息 |
| isWorkflow | IS_WORKFLOW | 是否驱动工作流 | 逻辑值 |
| isGraphic | IS_GRAPHIC | 是否驱动图引擎 | 逻辑值 |
| inScript | IN_SCRIPT | JS脚本内容 | **保留** |
| outScript | OUT_SCRIPT | JS脚本内容 | **保留** |
| channelType | CHANNEL_TYPE | 通道类型 | 四个值：ADAPTOR, CONNECTOR, ACTOR, DIRECTOR |
| channelComponent | CHANNEL_COMPONENT | 通道组件 | Java类名 |
| channelConfig | CHANNEL_CONFIG | 通道配置 | Json配置格式 |
| configIntegration | CONFIG_INTEGRATION | 集成配置 | 对应集成结构Integration |
| configDatabase | CONFIG_DATABASE | 数据库配置 | 对应数据库结构Database |
| dictConfig | DICT_CONFIG | 字典配置 | Json配置格式 |
| dictComponent | DICT_COMPONENT | 字典组件 | Java类名 |
| dictEpsilon | DICT_EPSILON | 字典消费定义 | Json配置格式 |
| mappingConfig | MAPPING_CONFIG | 映射配置 | Json配置格式 |
| mappingMode | MAPPING_MODE | 映射模式 | 四个值：BEFORE, AFTER, AROUND, NONE |
| mappingComponent | MAPPING_COMPONENT | 映射组件 | Java类名 |
| serviceRecord | SERVICE_RECORD | 服务记录类型 | Java类名，Record接口 |
| serviceComponent | SERVICE_COMPONENT | 服务组件定义 | Java类名，**开发重点** |
| serviceConfig | SERVICE_CONFIG | 服务配置 | Json配置格式，对应 options |
| identifier | IDENTIFIER | 模型标识 | 配置的建模中的模型标识 |
| identifierComponent | IDENTIFIER_COMPONENT | 标识选择器 | Java类名 |


## 2. 配置步骤

本文只介绍接口和任务本身的配置，不介绍接口中的权限配置，接口中的权限配置参考：[OS-001 - 接口权限配置](/origin-x-engine/4.ox-authorization/os-001-jie-kou-quan-xian-pei-zhi.html)

```shell
# 安装UUID生成工具，主键生成使用该配置
npm install -g uuid
```

>  # Excel文件配置技巧，在单元格中填写：JSON:plugin/bastion/cmdb-v2/options/job-config.json，加入了`JSON:`前缀就可以直接从 `resources` 目录中读取Json文件作为Json字段的配置数据内容。

### 2.1. 准备流程

在`src/main/resources/init/oob/`中创建API所需`Excel`文件（可参考目前的`xxx-api.xlsx`类型的）。

### 2.2. 填充规则

#### 2.2.1. 接口配置规则

1. 接口类型`type`目前必须配置`REQUEST_RESPONSE`（请求/响应模式）。
2. 注意`secure`安全配置，`secure = true`则路径前缀会加上`/api/ox`，`secure = false`则只加`/ox`前缀，这里`/ox`前缀在`vertx-jet.yml`中配置，因客户而有所区别。
3. `uri`和`method`不能是系统中已经存在的，遵循RESTful基本指导法则，**不可重复**！
4. 注意参数来源`paramMode`配置，尽可能采用最小需求法则：
    1. **PATH**：仅读取路径上的参数，如`/api/:name`中的`name`。
    2. **QUERY**：读取路径和请求参数，`/xxx?name=yyy`中的`name`，同时包括**PATH**。
    3. **BODY**：读取HTTP请求体，包含**QUERY**和**PATH**两种。
5. 注意导入/导出接口需要设置`consumes`和`produces`。
6. 注意所有表中的`sigma, active, language`都不要填错，参考已有配置。
7. `serivceId`中填写新生成的服务主键（`I_SERVICE`）。

#### 2.2.2. 任务配置规则

1. 首先注意`namespace`，目前必须是`cn.originx.vie.app.ox`。
2. 任务名称`name`就是任务的编码`code`，该名称**不可重复**！
3. 注意任务类型`type`的配置：
    1. ONCE：一次性任务。
    2. PLAN：计划任务，启动时间是容器启动时间，间隔执行。
    3. FIXED：定时任务，启动时间可固定，间隔执行。
4. 如果配置的是 **ONCE**（一次性任务），`runAt`和`duration`都不需配置。
5. 如果配置的是 **PLAN**（计划任务），`duration`必须配置。
6. 如果配置的是 **FIXED**（定时任务），`runAt`和`duration`都必须配置。
7. 如果任务时间执行很长，必须配置`threshold`。
8. 所有时间单位都采用了`秒（s）`。
9. 根据自己的需要配置`incomeAddress / incomeComponent`。
10. 根据自己的需要配置`outcomeAddress / outcomeComponent`。
11. `proxy`的默认值是：`io.vertx.tp.jet.uca.micro.JtThanatos`，可定义自己的任务代理类，一旦使用了自己的任务代理类，那么很多内容会面临重写。
12. 注意所有表中的`sigma, active, language`都不要填错，参考已有配置。

#### 2.2.3. 服务配置规则


2. 注意`namespace`，目前必须是`cn.originx.vie.app.ox`。
3. 服务名称`name`必须唯一，**不可重复**！
4. 注意选择通道类型`channelType`，带集成的选择后者:
    2. 接口类型只能使用`ADAPTOR / CONNECTOR`
    3. 任务类型只能使用`ACTOR / DIRECTOR`
5. `isWorkflow / isGraphic`是**保留配置**，后期会使用。
6. 服务配置：
    2. 服务组件类`serviceComponent`就是配置开发人员开发的通道类。
    3. 服务配置`serviceConfig`对应`Options`。
    4. 服务记录类`serviceRecord`配置的值必须是数据记录值：
        2. 旧版配置：`cn.originx.modeling.atom.data.DataRecord`。
        3. 新版配置：`io.vertx.tp.atom.modeling.data.DataRecord`。
7. 标识配置：
    2. `identifier`为固定的模型标识符，需要静态绑定，并且需要静态绑定到合法定义的模型中。
    3. `identifierComponent`用来定义标识选择器，选择不同的`identifier`执行服务。
8. 映射器配置：
    2. `mappingConfig`配置基础映射文件，静态Json结构。
    3. `mappingMode`配置映射模式：`BEFORE/NONE/AFTER/AROUND`。
    4. `mappingComponent`配置映射组件，**保留配置**。
9. 字典翻译器：
    2. `dictConfig`字典配置文件，静态Json结构。
    3. `dictEpsilon`字典消费配置，定义字典的消费相关信息。
    4. `dictComponent`字典组件
        2. 默认值：`io.vertx.tp.optic.business.ExAmbientDictionary`
10. 数据库/集成配置：
    2. `configDatabase`：数据库配置
    3. `configIntegration`：集成配置

### 2.3. 导入配置

1. 停止容器
2. 修改 `OxDevelop`代码如下，首参不变，第二参使用您创建的文件前缀：

    ```java
    package cn.vertxup;
    
    import io.vertx.tp.ke.booter.Bt;
    
    public class OxDevelop {
    
        public static void main(final String[] args) {
            Bt.doImports("init/oob/", "第二参");
        }
    }
    ```
1. 设置`OxDevelop`的执行配置，参考环境配置，然后执行该程序，将数据重新导入
2. 检查您的权限是否已经进入到系统，查询几张表中的相关信息
3. 重启容器，使用 `Postman` 测试

## 3. 总结

这样就配置好完整的接口和任务了。


