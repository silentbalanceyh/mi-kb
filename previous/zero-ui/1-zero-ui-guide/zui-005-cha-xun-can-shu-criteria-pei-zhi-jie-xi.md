# 查询参数criteria配置解析

## 1. criteria配置

在`Zero / Zero UI / Origin X`中有一种特殊的查询请求：POST查询，并且该查询主要包含了特殊参数：

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

那么不论是Zero UI还是Origin X，都需要将查询部分配置的`criteria`的数据解析出来才可以真正投入到实战中，关于查询引擎部分的语法可以参考：[KMZ-001 查询引擎语法](/uniform-documentation/zero-concept/kmz-001-cha-xun-yin-qing-yu-fa.html)。

## 2. 基本说明

比如现在有一个请求的URI如：`/app/:appId/menus`，那么这个时候的`appId`应该如何配置呢？你当然会说，直接在代码里写！——我承认这是一个不错的解决方案，但是别忘了Origin X是数据驱动的，Zero UI是配置驱动的，也就是说如果使用了硬编码，而不是配置/数据层面的方式，若是编码的方式来处理输入参数是没有问题的，但这个查询本意是：“按照应用查询。”——如何用数据和配置来表示这个语义，就是Zero UI/Origin X解决的事情，二者使用了同样的criteria配置解析流程。

比如下边这个片段：

```json
{
    "criteria":{
        "category,i","ENUM:PROD`STG`DEV"
    }
}
```

如何解释上边的表达式呢？

* 需要查询 category 字段，使用的操作符为`IN`操作符。
* 查询的category字段的Where条件右值是一个固定枚举集合。
* 这个集合的值为：`PROD、STG、DEV`三个。

也就是说，如果category对应的底层数据库字段为：`CATEGORY`，那么这个查询生成的SQL语句的Where子句为：

```sql
CATEGORY IN ('PROD','STG','DEV')
```



