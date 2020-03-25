# 第一步：任务和接口配置

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


## 2. 接口配置

本文只介绍接口本身的配置，不介绍接口中的权限配置，接口中的权限配置参考：[OS-001 - 接口权限配置](/origin-x-engine/quan-xian-jiao-cheng/os-001-jie-kou-quan-xian-pei-zhi.html)



