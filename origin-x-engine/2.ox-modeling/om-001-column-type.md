# OM-001 数据库列类型

数据库列类型的元数据文件定义主要位于项目`ox-plugin/infix-db/infix-xxx`中，目前包含：

* infix-mysql：MySQL数据库层
* infix-oracle：Oracle数据库层

定义文件位于：`src/main/resources/engine/database/sql/XXX/schema.json` 中，其中**XXX**表示当前数据库的类型，类型定义位于源文件`io.vertx.up.eon.em.DatabaseType`中，目前的版本定义如下：

```java
public enum DatabaseType {
    /* MySQL */
    MYSQL5,     // MySQL 5.x
    MYSQL8,     // MySQL 8.x
    MYSQL,      // Old Version
    /* Oracle */
    ORACLE,     // Old Version
    ORACLE11,   // 11
    ORACLE12,   // 12
}
```

## 1. 文件格式

schema.json的文件格式为 JSON 对象格式（`{}`），主要包含下边几个节点：

| 节点名称 | 含义 |
| :--- | :--- |
| definitions | 数据库的类型定义，描述模型类型到数据库列类型。 |
| supports | 数据库可支持的列类型（生成SQL语句用）。 |
| mappings | 数据库列转换定义，定义了数据库列类型可支持的转换。 |

## 2. 文件片段

该文件的片段信息如下：

```json
{
    "definitions": {
        "BOOLEAN": "BOOL",
        "INT": "INTEGER",
        "....": "...."
    },
    "supports": [
        "CHAR",
        "VARCHAR",
        "....",
    ],
    "mappings": {
        "BIT": [
            "BIGINT",
            "BINARY",
            "....",
        ]
        "....": [
            "...."
        ]
    }
}
```

## 3. Java类型

Java类型和模型目前支持如下：

| Java数据类型 | 模型类型 |
| :--- | :--- |
| java.lang.Boolean | BOOLEAN |
| java.lang.Integer | INT |
| java.lang.Long | LONG |
| java.math.BigDecimal | DECIMAL |
| java.time.LocalDate | DATE1 |
| java.time.LocalDateTime | DATE2 |
| java.lang.Long | DATE3 |
| java.lang.LocalTime | DATE4 |
| java.lang.String | STRING1 |
| java.lang.String | STRING2 |
| java.lang.String | JSON |
| java.lang.String | XML |
| java.lang.String | SCRIPT |
| java.nio.Buffer | BINARY |

## 4. 模型/数据库

模型类型和数据库类型的清单如下：

* [OM-001-1 MySQL定义](/origin-x-engine/2.ox-modeling/om-001-database/om-001-1-mysql.html)
* [OM-001-2 Oracle定义](/origin-x-engine/2.ox-modeling/om-001-database/om-001-2-oracle.html)



