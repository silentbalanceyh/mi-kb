# Ox RESTful设计规范

Ox RESTful设计规范是在URI路径设计规范基础之上扩展出来的指导规范信息，主要基于Ox的设计来定义的。

## 1. 路径分类

在整个Ox Engine中，基本URI的分类包括下边四种：

| 路径前缀 | 含义 |
| :--- | :--- |
| / | Zero中的公共URI |
| /ox/ | Origin X中的公共URI |
| /api/ | Zero中的安全访问URI |
| /api/ox/ | Origin X中的安全访问URI |

## 2. 关于参数

整个Origin X Engine中，都包含下列两个Http Header：

| Header键 | 使用场景 | 说明 |
| :--- | :--- | :--- |
| Ox-App-Id | Zero的API专用 | 当前应用的key信息 |
| Ox-App-Key | Ox的API专用 | 当前应用的appKey信息（由平台发放） |

## 3. 路径设计规则

### 3.1. 参考例子

路径规则的设计对整个应用都会有所影响，所以这里讲解一下整个Origin X中的路径设计基本规则。

| 模块 | 模型词 | 模型标识 |
| :--- | :--- | :--- |
| 部门 | dept | sec.department |
| 部门下的用户 | user | sec.user |

> 所有的实体在读取的时候，如果要读取所有的内容则不传Ox-App-Id或Ox-App-Key，如果仅读取某个应用内的，必须加上这个参数。

### 3.2. Zero中的基本接口

参考：[OXR-002 Ox系统类接口](/origin-x-engine/3-oxhou-duan-kuo-zhan-kai-fa/oxr-002-oxxi-tong-lei-jie-kou.html)

### 3.3. Ox中的基本接口

| 操作 | 路径（前缀 /api/ox） | 阶 | HTTP方法 |
| :--- | :--- | :--- | :--- |
| 添加一个部门 | /dept | 一阶路径 | POST |
| 读取所有部门 | /depts | 一阶路径 | GET |
| 批量更新部门 | /depts | 一阶路径 | PUT |
| 批量添加部门 | /depts | 一阶路径 | POST |
| 批量删除部门 | /depts | 一阶路径 | DELETE |
| 读取某个部门 | /dept/:key | 二阶路径 | GET |
| 更新某个部门 | /dept/:key | 二阶路径 | PUT |
| 删除某个部门 | /dept/:key | 二阶路径 | DELETE |
| 查询引擎搜索（固定路由） | /dept/search | 二阶路径 | POST |
| 初始化 | /dept/init | 二阶路径 | POST |
| 查询某一类部门（不带状态） | /depts/{type} | 二阶路径 | GET |
| 查询某一类部门（带状态） | /depts/type/{type} | 三阶路径 | GET |
| 查询某一状态部门（带状态） | /depts/etat/{etat} | 三阶路径 | GET |
| 按部门名称查询（单返回） | /dept/name/:name | 三阶路径 | GET |
| 查询某个部门下所有用户 | /dept/:key/users | 三阶路径 | GET |
| 批量更新某个部门所有用户 | /dept/:key/users | 三阶路径 | PUT |



