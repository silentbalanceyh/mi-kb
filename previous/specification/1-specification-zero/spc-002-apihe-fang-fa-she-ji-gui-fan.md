# API和方法设计规范

Api类中的方法定义一般有两套，这个开发者可以根据实际情况来，这个文件接口中一般包含普通CRUD的操作接口，对于更复杂的RESTful的接口规范可以参考：[SPR-002 URI业务设计规范](/specification/4-specification-restful/spr-002-uriye-wu-she-ji-gui-fan.html)。

实体名如：user（用户）

## 1. 基本增删查改

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| get | GET | /user/{id} | 按照id查询某个实体 |
| post | POST | /user | 创建某个实体 |
| put | PUT | /user/{id} | 按照id更新某个实体 |
| delete | DELETE | /user/{id} | 按照id删除某个实体 |

## 2. 批量操作

* 可带上查询树`criteria`语法。
* 返回值为JsonArray。

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| batchGet | GET | /users | 读取所有用户，JsonArray |
| batchPut | PUT | /users | 批量更新用户 |
| batchPost | POST | /users | 批量创建用户（ID集） |
| batchDelete | DELETE | /users | 批量删除用户（ID集） |

## 3. 辅助功能

* 返回值为固定字符串：`EXISTED，MISSED`，不论哪种接口，EXISTED表示存在，MISSED表示不存在，替换原始的true/false。
* 这二者都为POST查询，可提供`criteria`语法来执行查询树。
* `updated`标记表示添加过程中检查还是更新过程：更新过程中检查是否存在需要去掉本条记录。

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| existUser | POST | /user/exist | 检查用户是否存在，存在 = true，不存在 = false |
| missUser | POST | /user/miss | 检查用户是否不存在，不存在 = true，存在 = false |
| existWhenUser | POST | /user/exist-updating | 更新过程检查 |
| missWhenUser | POST | /user/miss-updating | 更新过程检查 |

## 4. 单记录查询

* 单记录查询中的 field = value 模式只可以输入单个条件信息。
* 在POST查询中固定使用 `/<entity>/query`模式，支持`criteria`语法。
* 返回值为JsonObject。

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| getByField | GET | /user/{field}/{value} | 按照 field = value 查询用户信息 |
| getByFields | POST | /user/fetch | 按照criteria模式查询用户信息 |

## 5. 列表查询

* 列表记录查询中 field = value 模式只可以传入单个条件信息。
* 在POST查询中固定使用 `/<entity>/query`模式，支持`criteria`语法。
* 返回值为List&lt;T&gt;，最终转换成JsonArray。

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| queryByField | GET | /users/{field}/{value} | 按照 field = value 查询用户信息 |
| queryByFields | POST | /users/query | 按照criteria模式查询用户信息 |

## 6. 复杂搜索

* 复杂搜索只能使用`criteria`语法执行查询。
* 返回值为Pagination转换出来的JsonObject，带`list, count`两个节点。
* 如果是Map结果，则直接使用 `value = JsonObject`的格式（暂定）

| 方法名 | HTTP方法 | URI | 含义 |
| :--- | :--- | :--- | :--- |
| search | POST | /users/search | 复杂搜索 |
| searchMapField | POST | /users/search/{field} | 复杂搜索，返回值一个Map结果 |

## 7. 子查询规则

对于牵涉到的子查询，直接使用单个实体带上id的根路径，后边跟上 1 - 6的基本规则即可。

举例：

* app：应用模型
* menu：菜单模型

按照应用查询菜单的全套API如下：

| API | 返回值 | 含义 |
| :--- | :--- | :--- |
| /app/{key}/menus | JsonArray | 查询当前应用下所有菜单信息 |
| /app/{key}/menus/search | JsonObject（分页处理） | 查询当前应用下满足条件的所有菜单 |
| /app/{key}/menus/query | JsonArray | 查询当前应用下满足条件的所有菜单 |
| /app/{key}/menus/search/{field} | JsonObject | 底层调用 searchMapField |

## 8. 业务含义常用

业务术语表参考：[SPC-003 业务术语规范](/specification/1-specification-zero/spc-003-ye-wu-zhu-yu-gui-fan.html)。

