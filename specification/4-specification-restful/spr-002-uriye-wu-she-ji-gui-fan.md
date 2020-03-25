# URI业务设计规范

## 1. 基本Api说明

业务URI规范基础说明包括以下几个操作，用于解决一些特殊的URI地址规范相关问题。

### 1.1.路径参数冲突问题

如下：

```shell
/api/user/by/:name
/api/user/by/:type
```

该路径会被识别成同一个路径，即出现`Duplicated`的重复模式，且Restful需要灵活应用Http中的各种方法如：`PUT、POST、GET、DELETE`等。

### 1.2.满足HTTP幂等性

* 只允许在特殊的规范接口中使用`POST`查询，一般的接口提倡依旧使用`GET`查询优先；
* 如果是更新、审批、变更等使用`PUT`方法代替`POST`方法，不出现语义冲突，其中包括：
  * 如果是全资源更新，以及批量更新，使用`PUT`。
  * 如果是部分资源更新，且一般的HTTP不具有幂等性，则使用`POST`。
* 对于特殊的鉴权使用固定路径路由信息来完成。

### 1.3.API使用场景

该API主要用于规范化以下场景：

* 一个实体Entity的普通CRUD接口
* 根据单字段查询Unique对象，根据ID查询Unique对象，以及判断对象是否存在
* 复杂的条件查询：搜索、分页、排序、过滤
* 批量删除、批量添加、批量更新、读取所有数据
* 按条件删除和更新

### 1.4.参数类型

* Form：直接提交表单类型
* JsonObject：通过Body传输数据（POST、PUT、DELETE使用，GET禁用）
* JsonArray：通过Body传输数据
* Path：使用路径参数
* Query：使用查询参数
* Header：参数从Http Header头过来

## 2. 基础规范示例

### 2.1.动作基础主要分为

|  | 添加 | 更新 | 删除 | 查询 |
| :--- | :--- | :--- | :--- | :--- |
| （单） | 添加 | 更新 | 删除 | 查询 |
| （集合） | 批量添加 | 批量更新 | 批量删除 | 全查询 |
| （单条件） | X | 单条件更新 | 单条件删除 | 单条件查询 |
| （多条件AND） | X | 多条件更新 | 多条件删除 | 多条件查询 |
| （多条件OR） | X | 多条件更新 | 多条件删除 | 多条件查询 |
| （多值） | X | 多值更新 | 多值删除 | 多值查询 |
| （复杂搜索） | X | X | X | 分页、排序、筛选、过滤 |
| （存在判断） | X | X | X | 存在与否，类似count = 0 ? &lt;&gt; 0 |
| （运算处理） | X | X | X | 聚集计算，MR算法或其他 |

### 2.2.URI专用路径

基础规范示例以user为Entity，基本阅读规则如下：

* 标记为【间】表示根据实际情况进行设计，不直接使用
* 【E】表示当前集合中只支持JsonObject类型，不可使用其他类型
* 【ID】则表示当前集合中只允许ID集合，可能是长整型数组，也可能是字符串数组，【T】则表示当前数组中只能是某种类型，这两种标记都表示不可使用JsonObject作为数组元素
* `{field}`表示属性占位符，路径中填充上实际属性名
* 【单】表示单条件，【多】表示多条件
* 【O】表示返回值为JsonObject，【A】表示返回值为JsonArray，【B】表示返回值为true/false

| 路径前缀 | HTTP方法 | 属性标记 | 参数类型/格式 | API说明 |
| :--- | :--- | :--- | :--- | :--- |
| /api/user | POST | 【O】 | JsonObject | 添加一个用户 |
| /api/users | GET | 【A】 | 无 | 读取所有用户 |
| /api/users | POST | 【B】 | JsonArray【E】 | 批量添加用户 |
| /api/users | PUT | 【B】 | JsonArray【E】 | 批量更新用户 |
| /api/users | DELETE | 【B】 | JsonArray【ID】 | 批量删除用户 |
| /api/user/:id | PUT | 【O】 | Path + JsonObject | 根据ID更新一个用户 |
| /api/user/:id | DELETE | 【O】 | Path | 根据ID删除一个用户 |
| /api/user/:id | GET | 【O】 | Path | 根据ID读取一个用户 |
| /api/user/exist | POST | 【多、B】 | JsonObject，特殊查询条件 | 判断特殊条件用户是否存在 |
| /api/user/and | POST | 【多、O】 | JsonObject，特殊查询条件 | AND连接查询，所有条件之间使用AND |
| /api/users/and | GET | 【多、A】 | Query | 【3个条件或以下】AND连接查询，所有条件之间使用OR |
| /api/users/and | POST | 【多、A】 | JsonObject，特殊查询条件 | 【3个条件以上】AND连接查询，所有条件之间使用AND |
| /api/users/and | PUT | 【多、A】 | Query/JsonObject，特殊查询条件 | 所有条件之间使用AND，按条件批量更新 |
| /api/users/and | DELETE | 【多、B】 | JsonObject，特殊查询条件 | 所有条件之间使用AND，按条件批量删除 |
| /api/users/or | GET | 【多、A】 | Query | 【3个条件或以下】OR连接查询，所有条件之间使用OR |
| /api/users/or | POST | 【多、A】 | Query/JsonObject，特殊查询条件 | 【3个条件以上】OR连接查询，所有条件之间使用OR |
| /api/users/or | PUT | 【多、A】 | JsonObject，特殊查询条件 | 所有条件之间使用OR，按条件批量更新 |
| /api/users/or | DELETE | 【多、B】 | JsonObject，特殊查询条件 | 所有条件之间使用OR，按条件批量删除 |
| /api/users/search | GET | 【多、O\|A】 | Query | 【快速搜索】根据查询条件搜索满足条件的用户集合，分页参数page, size，支持单字段排序asc=field或desc=field |
| /api/users/search | POST | 【多、O\|A】 | JsonObject、特殊查询条件（标准条件） | 【高级搜索】分页、排序、筛选、过滤的搜索，返回值中带有count。 |
| /api/users/query | POST | 【多、O\|A】 | JsonObject、查询树高级语法（查询树） | 【高级查询】提供查询树语法用于高级查询，可分页，可排序 |
| /api/user/{field}/:value | GET | 【间、单、O】 | Path | 按某个字段查询用户，实例：/api/user/email/lang.yu@126.com，按email=lang.yu@126.com读取用户 |
| /api/user/{field}/:value | PUT | 【间、单、O】 | Path + JsonObject | 按某个字段更新用户，示例：/api/user/name/xxx，按name=xxx更新用户 |
| /api/user/{field}/:value | DELETE | 【间、单、B】 | Path | 按某个字段删除当前用户 |
| /api/users/{field}/:value | GET | 【间、单、A】 | Path | 按某个字段读取满足条件的所有用户 |
| /api/users/{field}/:value | PUT | 【间、单、A】 | Path + JsonObject | 按某个字段批量更新用户，示例：/api/users/type/single，批量更新type=single的用户 |
| /api/users/{field}/:value | DELETE | 【间、单、B】 | Path | 按某个字段删除满足条件的所有用户 |
| /api/users/in/{field} | POST | 【间、单、O\|A】 | JsonArray【T】 | 拉平返回满足条件的记录，构成JsonArray集，通过参数grouped判断是否执行分组，如果分则则返回值按传入的T执行GroupBy，返回JsonObject键数量和参数【T】的数量一致。 |
| /api/user/exist/{field}/:value | GET | 【间、单、B】 | Path | 判断数据库中是否包含该条件的记录，field=value为条件。 |
| /api/user/and/{field}/:value | POST | 【间、多、A】 | JsonObject，特殊查询条件 | AND连接查询，主条件使用field=value，符合条件放到JsonObject中 |
| /api/users/or/{field}/:value | POST | 【间、多、A】 | JsonObject、特殊查询条件 | OR连接查询，主条件使用field=value, 复合条件放到JsonObject中 |
| /api/users/and/{field}/:value | POST | 【间、多、A】 | JsonObject、特殊查询条件 | OR连接查询，主条件使用field=value，复合条件放到JsonObject。 |

### 2.3. URI设计基本规则

有几点需要说明：

* 单路径`/users`中的`PUT/DELETE`方法可提供Body参数支持【查询u树】语法。
* 双路径查询中，因为不存在“按条件批量添加”，所以`GET`用于字段少的场景，而`POST`用于字段多的场景，对应`and, or`。
* 双路径中，`search`不支持查询树，`query`仅支持查询树语法，二者互斥。
* 三路径中，`/users/{field}/:value`同样不存在“按条件添加”，所以`POST`被用于`in`语法的多值查询。
* 如果开发过程有自定义路径，则可参考双路径和四路径，不可扩展单路径和三路径。



