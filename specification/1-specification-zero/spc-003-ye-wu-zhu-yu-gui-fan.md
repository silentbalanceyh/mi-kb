# 业务术语规范

业务术语规范用语定义一些固定的业务层面的含义规范，方便开发人员统一理解。

## 1. API方法定义

API方法定义描述了在Agent/Api两种Java文件中方法的名字。

| 项 | 含义 |
| :--- | :--- |
| get | 读取单条记录的专用前缀，只用于描述按照 id = value的方式查询单条记录的结果。 |
| post | 创建记录专用方法 |
| put | 更新记录专用方法 |
| delete | 删除记录专用方法 |
| batchXxx | 所有批量操作使用batch作为方法前缀 |
| exist/miss | 辅助方法常用的是检查这个记录是否存在，直接使用exist/miss前缀，返回值统一 |
| getByXx | 按照某个字段查询单条记录返回JsonObject |
| queryXx | 按照某个条件查询多条记录返回JsonArray |
| search | 按照某个条件查询多条记录，返回JsonObject（list,count），带分页、排序、列过滤、查询树 |

* API中的标准方法定义和HTTP Method对应，使得开发人员不用去思考POST/PUT究竟哪个规则用于更新。
* 批量全部使用batch方法名前缀。
* 将query/search的返回值进行限定，query只返回JsonArray，有多少条返回多少条，search返回一个JsonObject，主要是带`list`和`count`两个属性，用于处理分页数据。

## 2. Worker方法

Worker方法名称定义了Api层的专用方法名称，并且可触发Aop层的Audit设置操作

| API | Worker中方法 |
| :--- | :--- |
| get | read |
| post | create |
| put | update |
| delete | remove |
| exist/miss | exist/miss |
| query/search | fetch（Jooq底层统一） |

* 上述方法直接可以通过`Ux.Jooq`的模式统一执行，所以不需要开`Stub/Service`的业务逻辑层来完成。
* 除了上边的属于对应之外，其他的想对自由。
* 由于`Ux.Jooq`已经生成了Dao层，所以不需要类似`insert/update/select/delete`等这种数据层的方法来描述行为。

## 3. 业务谓词

### 3.1. 基本业务谓词

| 词语 | 含义 | 协变规则 |
| :--- | :--- | :--- |
| get | 读取 | 【无】不可用于底层，只能使用于API接口定义中，禁止在底层Dao/Service中使用getBy，getX等方法，底层的Get表示Bean的“获取”方法。 |
| post | 创建 | 【无】只能用于API接口（Agent/Api） |
| put | 更新 | 【无】只能用于API接口（Agent/Api） |
| delete | 删除 | 可用于Dao层，不可用于业务逻辑层，Service业务逻辑层只能使用remove，不可用delete。 |
| exist/miss | 检查谓词 | 可用于Service业务逻辑层，也可用于Dao层，根据实际业务需求而定。 |
| query/search | 查询 | query只能用于返回JsonArray的情况，search只能用于返回JsonObject的情况。 |
| create | 创建 | 【无】用于Service业务逻辑层，不能用于Worker中 |
| read | 读取 | 【无】用于Service业务逻辑层，不能用于Worker中 |
| modify/update | 更新 | 【无】用于Service业务逻辑层，不能用于Worker中 |
| fetch | 查询 | 除开API层以外，全层通用。 |

* 【无】表示Zero中不推荐使用该谓词在其他地方使用。

### 3.2. 分层CRUD

|  | 添加 | 删除 | 查询 | 修改 |
| :--- | :--- | :--- | :--- | :--- |
| Api/Agent | post | delete | get / query / search | put |
| Worker | create | remove | read / fetch | modify |
| Service | create / add | remove | fetch | modify |
| Dao | insert | delete | select / fetch | update |

### 3.3. 含义性业务词

该部分内容在Zero/Origin中都是统一处理，所以参考：[SPC-000 业务术语统一](/specification/business-uniform/spc-000-ye-wu-zhu-yu-gui-fan.html)。

