# 资源请求接口

资源请求接口主要在认证授权接口之后执行，它需要经历一个初始化的流程，初始化主要包含：

* 应用名称在本地配置，一般为固定值，用于连接应用专用，如例子中的：`vie.app.ox`。
* 非安全类型的应用接口，可返回：`X-Sigma`和`X-AppId`，满足大部分资源消费需求。
* 安全类型（认证授权后）的应用接口，除了上述两个值以外，还包含：`X-AppKey`，用于访问应用敏感资源信息。

## 1. 初始化

地址：`http://<HOST>:<PORT>/app/name/:name`

HTTP方法：`GET`

| 参数名 | 说明 |
| :--- | :--- |
| name | 访问的应用资源的名称，单独访问时固定，平台有接口可读取所有的应用列表。 |

### Response

```json
{
    "data": {
        "key": "239975a7-f164-43b3-a9d1-4516f98c1948",
        "name": "vie.app.ox",
        "code": "ox",
        "title": "测试应用",
        "domain": "ox.engine.cn",
        "appPort": 5000,
        "urlEntry": "/login/index",
        "urlMain": "/home/dashboard",
        "path": "ox",
        "route": "/ox",
        "active": true,
        "sigma": "ENhwBAJPZuSgIAE5EDakR6yrIQbOoOPq",
        "language": "cn"
    }
}
```

上述响应信息中，核心信息如下

| 字段名 | 说明 |
| :--- | :--- |
| key | 资源请求专用自定义请求头：X-AppId |
| sigma | 资源请求专用自定义请求头：X-Sigma |
| language | （保留）多语言环境专用请求头：X-Lang |

## 2. 资源请求

资源请求只需要提供核心请求头的基础信息即可立即访问 ox 中的接口，需要消费的请求头可参考具体接口的基础说明。

| 请求头 | 说明 |
| :--- | :--- |
| X-Sigma | 多租户环境标识符 |
| X-AppId | 多应用环境标识符 |
| X-AppKey | 敏感资源标识符 |
| X-Lang | 多语言环境标识符 |
| Authorization | 格式如：Bearer &lt;access\_token&gt;，安全标识符 |

安全标识符是通过认证授权接口拿到的令牌组装完成，有几点需要说明：

* 前缀为：`Bearer`（大小写敏感）
* `Bearer`和`access_token`中间带有空白



