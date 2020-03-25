# 题目

## Objective

1. 使用Java语言在Spring Boot中开发一个Restful Web Service
2. 使用JUnit测试该Restful Web Service

## Required

1. 这个Restful Web Service需要运行在Docker容器中（Docker容器自选），该Docker镜像在Maven打包过程中构建；
2. 数据库使用Postgre SQL的Docker模式（下载地址：[https://registry.hub.docker.com/\_/postgres/](https://registry.hub.docker.com/_/postgres/)），数据表可在启动过程中自动创建，也可手工创建；
3. 整个项目使用传统分层架构，包括：Resource -&gt; Service -&gt; Dao
4. 针对所有的类书写测试用例，基本要求：
   1. 所有依赖项使用Mock方式（Mockito）
   2. 使用Jococo检查测试覆盖率（&gt;65%）
5. 开发完成后使用Markdown书写相关文档，设计思路、架构、编码规范等；
6. （可选）将日志记录添加到整个请求中，并只是在开发模式中开启日志，可放到AOP层处理；
7. 最终发布到GitHub中，可直接下载代码并且根据文档部署；

## Interface

### 1. 创建记录

POST：[http://localhost:8080/api/v1/item](http://localhost:8080/api/v1/item)

**请求：**

```json
{
            "businessId": "业务主键（需要检查不重复）",
            "type": "只支持A，B，C三个值",
            "standardItemIdentification": "随意",
            "sellersItemIdentification": "随意",
            "itemName": "随意",
            "period": "时间格式，输入格式为201703类似",
            "unitCode": "EA",
            "currencyID": "CNY",
            "receiptID": "随意",
            "comment": "大文本",
            "department": "随意",
            "priceWithTax": "金钱格式",
            "quantity": "数量",
            "amount": "总金额"
}
```

* 在数据库中创建主键，主键使用UUID格式；
* type类型使用PostgreSQL中的enum，只能支持A，B，C输入；
* 时间格式参考：[https://en.wikipedia.org/?title=ISO\_8601](https://en.wikipedia.org/?title=ISO_8601)
* 创建过程中检查：单价 x 数量 = 总价，该公式是否匹配
* 业务主键不能重复；

**响应：**

```json
{
            "id":"UUID",
            "businessId": "业务主键（需要检查不重复）",
            "type": "只支持A，B，C三个值",
            "standardItemIdentification": "随意",
            "sellersItemIdentification": "随意",
            "itemName": "随意",
            "period": "时间格式，输入格式为201703类似",
            "unitCode": "EA",
            "currencyID": "CNY",
            "receiptID": "随意",
            "comment": "大文本",
            "department": "随意",
            "priceWithTax": "金钱格式",
            "quantity": "数量",
            "amount": "总金额",
            "createTime":"创建人"
}
```

### 2. 查询记录

POST：[http://localhost:8080/api/v1/item/query/:limit](http://localhost:8080/api/v1/item/query/:limit)

**请求：**

```json
{
            "type": "A，只选择一种",
            "itemName": "使用Like模式匹配",
            "period":{
                        "from":"从什么时候开始（包含）",
                        "to":"到什么时候截止（包含）"
            },
            "priceWithTax":{
                        "from":"从什么价格（包含）",
                        "to":"到什么价格（包含）"
            }
}
```

* 返回结果按照createTime创建时间逆序排列；
* limit可限制结果返回的条数，并写入到count中；

**响应：**

```json
{
    "count":11,
    "items":[
        {
            "comment":"此处数据格式参考接口：1.创建记录的对象响应格式"
        }
    ]
}
```

## Summary

考核标准：

1. 本身的代码风格、编程规范；
2. 题目的设计思路；
3. 整个程序的架构设计，以及相关完整度；
4. 完成度 &gt; 80%（最低要求可正常运行）



