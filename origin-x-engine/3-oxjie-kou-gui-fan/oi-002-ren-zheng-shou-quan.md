# 认证授权接口

整个Ox平台默认采用 OAuth 的授权码认证方式，也可支持Basic认证（需单独配置），它对所有第三方集成拥有监控、管理的功能，每个第三方集成账号只能和接口集成，不可登录系统，权限的定制只有CMDB管理员可在平台内操作，实现细粒度的资源权限请求。

## 1. 流程

Ox平台的 OAuth 认证流程如下：

1. `账号 + 密码（MD5加密）`访问登录接口：`/oauth/login`获取账号信息。
2. 根据账号信息从授权码接口：`/oauth/authorize`获取临时授权码（30s过期）。
3. 使用临时授权码从令牌接口：`/oauth/token`获取令牌，令牌可本地存储，在一段时间内生效，由于登录限制，所以一个账号同一时间只会在平台内维持唯一的令牌。
4. 将令牌`access_token`放到请求头：`Authorization`中发送资源申请请求获取资源信息。

## 2. 接口详细

### 2.1. 登录：/oauth/login

地址：`http://<HOST>:<PORT>/oauth/login`

HTTP方法：`POST`

请求头：`Content-Type: application/json`

#### Request

```json
{
    "username":"（账号）",
    "password":"（MD5加密过的密码）"
}
```

| 参数名 | 说明 |
| :--- | :--- |
| username | 集成账号 |
| password | 集成账号密码（MD5加密并且转大写） |

#### Response

```json
{
    "data": {
        "key": "返回的用户主键",
        "scope": "应用名称",
        "clientSecret": "用户专用密钥",
        "grantType": "authorization_code"
    }
}
```

| 字段名 | 说明 |
| :--- | :--- |
| key | 当前账号的 clientId |
| scope | 当前应用的 范围 |
| clientSecret | 当前账号的 clientSecret |
| grantType | （保留）当前应用的 OAuth 认证方式 |

### 2.2. 授权码：/oauth/authorize

地址：`http://<HOST>:<PORT>/oauth/authorize`

HTTP方法：`POST`

请求头：`Content-Type: application/json`

#### Request

```json
{
    "client_id":"用户主键",
    "client_secret":"创建账号时生成的64位随机字符串，盐",
    "response_type":"...（保留）",
    "scope":"vie.app.ox"
}
```

| 参数名 | 说明 |
| :--- | :--- |
| scope | 当前应用的范围（必须，和第一步中的 scope 一致） |
| client\_id | /oauth/login 中返回的 key |
| client\_secret | /oauth/login 中返回的 clientSecret |
| response\_type | （保留，当前版本可不传）/oauth/login 中返回的 grantType 值 |

#### Response

```json
{
    "data": {
        "code": "Gr0RzNkZ"
    }
}
```

| 字段名 | 说明 |
| :--- | :--- |
| code | 临时授权码，随机8位数，并且只有30秒的使用时限。 |

### 2.3. 令牌：/oauth/token

地址：`http://<HOST>:<PORT>/oauth/token`

HTTP方法：`POST`

请求头：`Content-Type: application/json`

#### Request

```json
{
    "client_id":"9744ae98-eac1-4825-b558-3e53c78940da",
    "code":"yRRbnY3F"
}
```

| 参数名 | 说明 |
| :--- | :--- |
| client\_id | /oauth/login 中返回的 key |
| code | /oauth/authorize 中返回的临时授权码 |

#### Response

```json
{
    "data": {
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....",
        "refresh_token": "eyJ0.....",
        "iat": 1578237840003
    }
}
```

| 字段名 | 说明 |
| :--- | :--- |
| access\_token | 资源访问使用的JWT令牌 |
| refresh\_token | 资源访问令牌过期时候需重新生成令牌的刷新令牌 |
| iat | 令牌过期时间 |



## 3. 错误表

| Http状态 | 内部错误码 | 说明 |
| :--- | :--- | :--- |
| 500 | -60007 | 服务器内部错误 |
| 400 | -60004 | 请求格式不对，必须是 Json 对象格式 |
| 449 | -80203 | 账号不存在 |
| 401 | -80204 | 密码错误 |
| 401 | -80202 | scope参数丢失，clientId 和 clientSecret不匹配，无法生成 code |
| 401 | -80201 | 授权码 code 过期，无法生成 token |



