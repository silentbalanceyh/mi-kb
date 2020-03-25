# 查询引擎语法

本文讲解Zero中使用的查询引擎语法。

## 1. 请求格式

查询引擎的请求使用固定格式：

* HTTP方法：使用POST查询而不是使用GET查询。
* Body的格式如下：

```json
{
    "criteria":{
    },
    "pager":{
        "page":1,
        "size":10
    },
    "sorter":[
        "name,DESC"
    ],
    "projection":[
    ]
}
```

* criteria：查询条件
* pager：分页参数，从第一页开始（page = 1）
* sorter：排序，数组格式，支持多字段优先级排序
* projection：列过滤。

## 2. 关于查询条件语法

查询条件的基本语法使用：

```json
{
    "field,op":"value",
    "related.field,op":"value"
}
```

criteria支持多级查询树的动作，也就是说可以支持多个查询条件构成查询语法树。

* 表名：USER
  * 字段：name，列名：NAME
  * 字段：email，列名：EMAIL
  * 字段：password，列名：PASSWORD
  * 字段：age，列名：AGE
  * 字段：active，列名：ACTIVE

接下来以上边为例讲解一些例子，先看看op列表。

### 2.1. op列表

| 操作符 | 示例格式（Json中键值） | 说明 | SQL 条件 |
| :--- | :--- | :--- | :--- |
| &lt; | "age,&lt;":20 | 小于某个值 | AGE &lt; 20 |
| &lt;= | "age,&lt;=":20 | 小于等于某个值 | AGE &lt; 20 |
| &gt; | "age,&gt;":16 | 大于某个值 | AGE &gt;= 16 |
| &gt;= | "age,&gt;=":16 | 大于等于某个值 | AGE &gt;= 16 |
| = | "age,=":12 或 "age":12 | 等于某个值 | AGE = 12 |
| &lt;&gt; | "name,&lt;&gt;":"LANG" | 不等于 | NAME &lt;&gt; 'LANG' |
| !n | "name,!n":"随意" | 不为空 | NAME IS NOT NULL |
| n | "name,n":"随意" | 为空 | NAME IS NULL |
| t | "active,t":"随意" | 等于TRUE | NAME = TRUE |
| f | "active,f":"随意" | 等于FALSE | NAME = FALSE |
| i | "name,i":\["A","B"\] | 在某些值内 | NAME IN \('A','B'\) |
| !i | "name,!i":\["C","D"\] | 不在某些值内 | NAME NOT IN\('A','B'\) |
| s | "name,s":"Lang" | 以某个值开始 | NAME LIKE 'Lang%' |
| e | "name,e":"Lang" | 以某个值结束 | NAME LIKE '%Lang' |
| c | "name,c":"Lang" | 模糊匹配 | NAME LIKE '%Lang%' |

### 2.2. 关于连接符

如果在查询引擎中出现了连接符，那么以同级树中的`""`键判断是 AND 还是 OR

| 键 | 值 | 连接符 |
| :--- | :--- | :--- |
| "" | true | AND（或者不写） |
| "" | false | OR（或者不写） |

## 3. 查询条件示例

### 3.1. 基本

查询系统中`name`的值为Lang，`email`的值为`lang.yu@hpe.com`的用户

```json
{
    "name":"Lang",
    "email":"lang.yu@hpe.com"
}
```

SQL：

```sql
NAME = 'LANG' AND EMAIL='lang.yu@hpe.com'
```

### 3.2. 嵌套

查询系统中`name`中包含了Lily，`email`后缀为`hpe.com`或`hp.com`的用户

```json
{
    "name,c":"Lily",
    "$0":{
        "":false,
        "email,e":"hpe.com",
        "email,e":"hp.com"
    }
}
```

> 注：上边的$0只是约定俗成，只要这个节点是Json的Object就可以了。

SQL：

```sql
NAME LIKE '%Lily%' AND (EMAIL LIKE '%hpe.com' OR EMAIL LIKE '%hp.com')
```

### 3.3. 其他操作符

查询年龄`age`大于16的未启用`active`的用户

```json
{
    "age,>":16,
    "active,f":"$0"
}
```

> 此时 $0 作为了占位符

```sql
AGE > 16 AND ACTIVE = FALSE
```

## 4. 作业

最终发一份Email给我，标上题号：silentbalanceyh@126.com

* 数据表名：SERVER

| 字段 | 列 | 含义 |
| :--- | :--- | :--- |
| name | NAME | 服务器名称 |
| ip | IP | 服务器IP地址 |
| os | OS | 服务器操作系统 |
| osVersion | OS\_VERSION | 操作系统版本 |
| type | TYPE | 服务器类型 |
| active | ACTIVE | 是否启用 |

### 4.1. 翻译

将下边查询条件翻译成SQL

```json
{
    "01-01":{
        "os":"Windows"
    },
    "01-02":{
        "name,e":"Linux",
        "osVersion":2.6
    },
    "01-03":{
        "active,f":"$0",
        "":false,
        "$0":{
            "name,s":"Duplicated",
            "active,t":"$1"
        }
    }
}
```

将下边SQL翻译成查询条件

```sql
-- 02-01
NAME LIKE '%X%' AND ( ACTIVE = TRUE OR TYPE = 'ONLINE' )
-- 02-02
IP LIKE '%.192.100' OR ( NAME = 'Linux' AND IP LIKE '%.100' )
-- 02-03
OS_VERSION > '10.0.1000' OR ( OS_VERSION < '9.0.1000' AND ACTIVE = TRUE )
```

描述（同时写Json和SQL）：

* 查询启用的操作系统为Windows并且版本不大于`8.0.1000`的记录。
* 查询所有NAME为空或操作系统版本号为空的记录。
* 查询所有TYPE为`OX`或TYPE不为`OX`但IP地址在`192.100.0.100`到`192.100.0.0`的值。



