# Tradeshift 应用开发手册

Restful API基本说明

这是Tradeshift的开发人员API文档，这些API是基于REST的，它允许集成工程师处理很多相关资源任务，如管理业务文档、读取发送者/接受者信息、分发文档到接受者等。

## Tradeshift的基础

Tradeshift Apps支持在公司网络、个人用户之间沟通以及交换文档信息。账号（公司或用户）、网络、文档都属于Tradeshift App Platform的基础【Account、Network、Document】。

* **Account/Company**：【账号／公司】账号描述了一个可以和其他公司交换业务文档的公司，账号同样也称为公司或者公司账号，它不同于单独的用户，一个账号可通过其他账户信息或分支机构来描述、也可以有多个用户。
  * **Branches**：Tradeshift支持一个二级架构的账号（它可以包含子账号），当账号跟一些主账号（Master-Account）关联时，这些链接的账号称为分支。
  * **Master Branch/Master Account**：一个管理员账号可用来组织同属于一个组织下的子账号信息，ABC Global Enterprise主分支。只要它不是另一个账户的子分支，技术上任何一个账户都可以是主分支。
  * **Child Branch/Child Account**：若一个账号属于主分支，如：ABC Global Enterprise USA Division子分支。只要它不是另一个账户的子分支，技术上任何一个账号都可以是子分支。
  * **Account Info**：姓名、地址、国家、税务ID等。
  * **Users**：用户必须在公司账号内创建。当一个用户登录，这个用户都进入了公司账号，它可以访问公司账号中所有的文档和信息。如果是在一个主分支中创建，这个用户可以在这个主分支和属于它的子分支中进行切换。如果在一个子分支中创建，那这个账号不能切换到其他分支中。
  * **User Info**：姓名、邮件、消费限制、汇报关系。
* **Network**：用户用于连接其他用户的多对多的集合，所有的Tradeshift的账号都连接到了Tradeshift的网络，也就是说任何一个账号都可以邀请它的业务伙伴加入到自己的网络中（类似于社交网络）。
  * **Connections**：两个账号之间的链接，一个链接在两个账号之间交换文档时必须是激活的。要开启一个链接，一个账号必须对另外一个账号发送链接请求，而接收请求这个账号必须要接收该请求。
  * **External Connection**：（俗称“手动连接“）您为要连接的业务伙伴创建一个“占位符”账户，您填写了账户的个人资料、包括姓名、电子邮件、地址信息等。若业务伙伴尚未接收连接请求，或者改账户没有激活。您也许仍然会更新账号信息如姓名、电子邮件等。一般情况下，外部连接是暂时的，一旦业务伙伴激活了“占位符”账户，该连接将会转变成Tradeshift连接——您将无法再更改账户资料信息。
  * **Tradeshift Connection**：（俗称“主动连接”）公司被邀请连接并已经接受邀请，受邀请的公司准备发送和接收文件，受邀请公司拥有其名称和电子邮件账户信息——您不能更改账户信息。
  * **Connection Property**：用于表述连接的自定义属性。如：买方公司可以将其内部供应商ID存储为每个供应商连接上的连接数据，对称地，另一方的供应商可以将其内部客户ID存储为连接属性。
  * **Group**：一个账户可以根据自己创建的组来分类连接，命名和定义由该账户决定。如，买方公司可能会有两个供应商：供应商需要采购订单，供应商不需要采购订单。
* **Documents/Documentfiles**：在招标、报价、购买、运输、发票等业务交易中，公司之间交换业务的文件。

  * **UBL**：一个标准的电子XML商业文件库、如采购订单和发票。这是Tradeshift使用的首选标准文档格式，标准由OASIS（结构化信息提升标准组织）开发，有关信息信息，可访问：[https://www.oasis-open.org/committees/ubl](https://www.oasis-open.org/committees/ubl)
  * **TSUBL**：TSUBL包含UBL 2.0标准下的文档子集，从此链接下载TSUBL规范和样本文件到TSUBL规范。或：转到[http://integrate.tradeshift.com/](http://integrate.tradeshift.com/)：。然后在”支持的格式“部分中单击”UBL“。

  * **Tenant Isolation**：租户（账户）之间是不可直接共享数据的，有与每个账户关联的存储。发送文件时，将在接收者的账户中创建并呈现发件人的文档的副本。因此，租户之间没有直接的数据共享。根据上下文，文档的一个副本的操作可能会影响或不影响其他副本。例如：

    * Supplier（Sender）：供应商（发件人）删除发送的发票将使发票在供应商账户中消失。然而，买方（接收方）仍然可以看到买方账户中的发票，因为买方对该发票的副本不受影响——这种情况下，对发票人的发票副本的操作不会影响收件人副本。
    * Buyer（Receiver）：买方（收件人）将收到的发票的状态更新为”已接受“，这将触发向供应商（发件人）账户的状态消息，将发票的供应商副本更新为”已接受“。这种情况下，对发票的接收者副本的操作会自动影响发件人副本。

## 核心概念/术语

* **REST API**：Tradeshift的API是基于HTTPS、REST和OAuth的，它允许开发人员使用HTTP方法PUT/POST/GET/DELETE来访问”资源“如：分发文档、创建用户、和其他账户连接。它可以被Tradeshift和第三方开发人员使用，完整的API可参考：[https://api.tradeshift.com/tradeshift/rest/external/doc](https://api.tradeshift.com/tradeshift/rest/external/doc)
* **OAuth**：Tradeshift同时支持OAuth1和OAuth2，开发人员可基于此访问用户资源，更多OAuth信息参考：[http://oauth.net/](https://www.google.com/url?q=http://oauth.net/&sa=D&ust=1465606866900000&usg=AFQjCNFVoq3l3yHpvjwjcQPTcVjj98vcRg)
* **UUID**：全局唯一标示符用来表示Tradeshift中的实体，根据Wikipedia中的描述，UUID是一个软件标识标准。Tradeshift使用具有插入连字符的十六进制文本的规范格式。如：de305d54-75b4-431b-adb2-eb6b9e546014。这儿有一些JavaScript和免费的站点生成UUID。例如：[https://www.uuidgenerator.net/](https://www.google.com/url?q=https://www.uuidgenerator.net/&sa=D&ust=1465606866902000&usg=AFQjCNGhka_7OB4zo7oGxMHeZbNsAmM8Ig)和[https://github.com/broofa/node-uuid](https://www.google.com/url?q=https://github.com/broofa/node-uuid&sa=D&ust=1465606866903000&usg=AFQjCNGsFXLyJaAqGxqObLsQ7TkUX4y_YQ)
* **Standards**：Tradeshift遵循适用于国家代码、货币代码、税码等代码的标准。当使用Tradeshift文档或参数时，开发人员需要使用相同的代码。以下是Tradeshift遵循的标准清单（不详尽），有关完整的标准和值集，请参考：[TSUBL](https://www.google.com/url?q=http://jsi9mwa10eq3hertdc3j3ou.wpengine.netdna-cdn.com/wp-content/uploads/2015/11/TSUBL_20151118.zip&sa=D&ust=1465606866904000&usg=AFQjCNGLpWLinRZxap-EfuRHiWIyjadEIg)[ specifications](https://www.google.com/url?q=http://jsi9mwa10eq3hertdc3j3ou.wpengine.netdna-cdn.com/wp-content/uploads/2015/11/TSUBL_20151118.zip&sa=D&ust=1465606866905000&usg=AFQjCNGPpnYyfDlYm19BL76T4DX82BE6Qg)


## 1. Creating Embedded App

Tradeshift维护了两套环境：sandbox.tradeshift.com和go.tradeshift.com，两套环境都是生产级环境。Sandbox可让您用于开发，而开发完成过后，可将您的环境Deploy到Go中。

### 1.1. Creating an App

创建应用之前，您需要一个Tradeshift的账号，您可以在主页中注册；一旦登陆过后，您可以到AppStore中去找到Developer应用，激活该应用时，它会让您填写一个Vendor的ID。所有Tradeshift中的应用都是使用VendorId.AppName的格式进行标识，如Tradeshift.Developer是您安装的应用，那么您就可以使用这个名称在您的生产环境中作为应用标示符——所以VendorId本身不允许任何点`.`符号。

一旦设置了VendorId，则您就可以直接创建您的应用，应用参数会在您的Developer App界面中，主要需要填写的参数如下：

* **APP ID**：系统自动生成（不可带点`.`符号）
* **APP NAME**：应用名称，主要用于显示
* **APP DESCRIPTION**：应用描述信息，主要用于介绍该应用
* **APP ICON**：应用本身的图标信息
* **DEFAULT LOCALE**：默认的时区信息、语言信息
* **OAUTH2 CLIENT ID**：OAuth集成专用的Client ID参数信息
* **OAUTH2 CLIENT SECRET**：OAuth集成专用的客户端secret信息
* **OAUTH2 REDIRECT URI**：OAuth专用的Callback的URI地址（注意区分Main URL）
* **PERMISSIONS**：一些支持的Tradeshift服务端API的权限
* **WEBHOOKS**：Web应用的一些钩子程序
* **MAIN URL**：应用的主界面地址

### 1.2. Creating the Simplest App

最简单的App创建只需要提供一个URL即可——应用本身不需要Tradeshift API认证、同样不需要任何OAUTH2的参数，直接在Main URL中填写：

[https://www.google.com/maps/embed?pb=!1m10!1m8! 1m3!1d12613.0080182887!2d-122.39574375!3d37.78413355!3m2!1i1024!2i768!4f13.1!5e0!3m2!1sen!2sus!4v1433284539696](https://www.google.com/maps/embed?pb=!1m10!1m8! 1m3!1d12613.0080182887!2d-122.39574375!3d37.78413355!3m2!1i1024!2i768!4f13.1!5e0!3m2!1sen!2sus!4v1433284539696)

注意：Main URL中填写的应用实际上不是由Tradeshift托管，也就是说您需要将您的应用部署在自己的服务器上——这样既方便您的应用和Tradeshift做集成，同样也可以定义自己任意的应用结构。

### 1.3. Main URL Gotcha & X-Frame-Origin

一些开发框架会自动发送代码防止它们的页面被包含在一个IFrame中，它们将发送HTTP头`X-Frame-Origin: SAMEORIGIN`或`X-Frame-Origin: Deny`比如上边的Google Map应用，我们则是用了嵌入式代码。查看[RFC7034](https://tools.ietf.org/html/rfc7034)了解更多X-Frame-Origin头，这些信息可以直接从Developer Tool中看到。我们推荐您确保您的应用不会发送`X-Frame-Origin`头——它可以允许Tradeshift将您的应用嵌入到IFrame中，您可以指定`ALLOW-FROM`，但这个仅仅只被一些浏览器支持（Firefox/IE），如果是基于Chromium的浏览器则会报错。

_Releasing App Into Appstore_

当您创建您自己的应用时，实际上创建的是一个隐藏类型——这个应用不可以在AppStore中方法，但可以安装在自己的账号下进行测试。若要发布您的应用到AppStore中，您需要提交给Tradeshift进行Review，发邮件到[apps@tradeshift.com](mailto:apps@tradeshift.com)（带上VendorId和AppId），则我们可以Review并Unhide它，Tradeshift最终会将它分配到对应的类别中。

一旦Tradeshift审批了这个应用，则它将在AppStore中可用并且可以激活。如果您在Sandbox中（sandbox.tradeshift.com）测试该应用，则您可以在生产环境部署它，接下来让Tradeshift做最终的Review，为了保证一致性，尽可能在两个环境中保持VendorId的一致性。

### 1.4. Full App Examples

Tradeshift创建了许多应用的例子让您更容易去理解这个架构，包括如何针对Tradeshift服务器授权，其中一个例子使用了Spring Boot、Node.JS、PHP，您可以在[GitHub](https://github.com/Tradeshift/tradeshift-app-samples)中查看。这里有[12分钟](https://drive.google.com/file/d/0Bx2z3BvoWzgtU05QdFludEROZ2c/view)视频演示了如何发布Spring.Boot应用以及如何配置成和Tradeshift这边协同工作。我们强烈推荐您完成这些简单联系来熟悉它，简单应用——Java Spring Boot演示了Tradeshift的Platform中不同的方面。

**Development Tips**

在开发过程，您往往需要迭代您的应用，并且Tradeshift服务器需要传递authentication tokens，我们推荐您使用[ngrok](https://ngrok.com/)或[localtunnel](https://localtunnel.github.io/www/)，允许将您的本地应用开放到公网上。

## 2. App Authentication

大部分应用都需要和Tradeshift的API进行交互，若要启用它，Tradeshift实现了标准的OAUTH2协议（[RFC6749](https://tools.ietf.org/html/rfc6749)），对于APP，我们使用OAUTH中的3-legged协议，当激活这个应用时，用户将被要求当前账号进行认证和授权保证账号有权限做这些。当创建/编辑一个App时，您可以设置App的secret、client identifier，另外您需要选择您所访问的资源地址（OAUTH域中）。

### OAuth 2 Authentication Sequence

**Step 1 - 重定向到Authorization Server获取授权码（Auth Code）**

App Vendor会提供Main URL和Redirect URL，这样Tradeshift服务器就知道通过何种顺序和谁进行通信，Main URL将会重定向下边的URL（TS授权服务器）来获取授权码：

```
https://api-sandbox.tradeshift.com/tradeshift/auth/login?response_type=code&
client_id={vendor_id.app_id}&redirect_uri={redirect_uri}&scope=offline
```

Tradeshift授权服务器将会发送下边的Http响应：

```
HTTP/1.1 302 Found
Location: {redirect_uri}?state&code={auth_code}
Example：
vendor_id = Eltec; app_id = MapApp
Redirect URL = https://appvendor.com/tradeshift/redirect
```

当用户打开一个iFrame应用，App Vendor的Main URL将会重定向到下边地址：

```
https://api-sandbox.tradeshift.com/tradeshift/auth/login?response_type=code&
client_id=Eltec.MapApp&redirect_uri=https://appvendor.com/tradeshift/redirect&scope=offline
```

之后Tradeshift授权服务器将会发送下边Http响应：

```
HTTP/1.1 302 Found
Location: https://appvendor.com/redirect?state&code=sE7RFd48QkijbjZ
```

**Step 2 - 通过授权码（Auth Code）交换令牌（Access Token）**

App Vendor将会创建一个请求发送到Token Endpoint：

```
POST: https://api-sandbox.tradeshift.com/tradeshift/auth/token
```

* Authorization: 使用Basic认证，参数值通过Base64的方式加密`username:password`，username的格式如：`{vendor_id.app_id}`，密码则是OAUTH2中在Developer设置的；
* Body内容类型：`application/x-www-form-urlencoded`
* 请求中发送的POST正文的参数编码使用`UTF-8`
* 参数`grant_type`：值必须是`authorization_code`
* 参数`code`：先前从应用中获取的授权码

该接口响应信息包括下边内容：

* `access_token`：用于访问API资源接口的令牌（Token）
* `token_type`：值为`Bearer`（通用的一种OAuth）
* `expires_in`：值为30，表示30分钟
* `id_token`：一个JSON格式的Web Token（JWT）——它包含大多数断言、以及用户的ID、用户所属的账号，只是需要解码。对于Web解码器，参考：[http://jwt.calebb.net/](http://jwt.calebb.net/) ，关于JWT的更多细节参考[https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32)。
* `client_id`：默认的App Vendor ID和App ID
* `refresh_token`：在Access Token过期后，这个刷新令牌可以用来获取新的Token，新的Access Token将在10分钟后过期，而Refresh Token则永远不会过期，可以使用多次。

**Example Request**

* 应用ID全名：Eltec.MapApp
* 客户端Secret：test123（可直接在Developer中设置）
* 使用Base64方式编码：`Eltec.MapApp:test123`（使用Basic认证）

```
POST: https://api-sandbox.tradeshift.com/tradeshift/auth/token
Authorization=Basic RWx0ZWMuTWFwQXBwOnRlc3QxMjM=&grant_type=authorization_code&code=sE7RFd48QkijbjZE
```

**Example Response**（包含Access、Id、Refresh Token）

```json
{
   "access_token": "EQ90um+/ODi7Gf1E98CSXNIhIVUY8w96n+6vKf5JtWwr2awGQoACND0XCVOaeza+HNocg10QujGnw6VlxMng3z7eVe6RCFAlGayFD4p3wVvaWzQKECwRoVxFjwcX3XwOcwfE1tT1MTAHGKb435VUaIN7peD9zo6L5SbdTuX5jNZzz4GWiZjdDo7iWVZQ0HmB/HzrIi6goTIohazOUJepAEZWV8koHrMwpqJFaKAaFJDgecJMREm18eaXhZ55Un2L6wwPOqF3KPV0dj/7ycNVRPlWrUC6M1oVlH4zsrdEGVMvN6ccpnD3pcDskQwLNtmos8srCXvy7doMsKmm1tliB8hmrzh/P9Eywjw8xSKeiW0wWGpe/oYLEgL10loqVGUn1vGRBRR5GUjIs+ysBVAAWgIAAQ==",
   "token_type": "Bearer",
   "expires_in": 30,
   "id_token": "eyJhbGciOiJub25lIn0.eyJleHAiOjE0MzYyMjYyNzAsInN1YiI6InNtYStzYXJhaGJyb3duQHRyYWRlc2hpZnQuY29tIiwidXNlcklkIjoiMjFkMjVjOTItYzBmNy00NGZkLWJiMzgtMzhiZjZmYmE3NDBmIiwiYXVkIjpbIkFCQ0luYy5NYXBBcHAiXSwiaXNzIjoiaHR0cHM6XC9cL2FwaS1zYW5kYm94LnRyYWRlc2hpZnQuY29tXC90cmFkZXNoaWZ0XC8iLCJqdGkiOiJqMmFyQXhRNHJOMFVVQm1qIiwiY29tcGFueUlkIjoiMDZhY2Q5MmItNmNiNS00OWZlLWFmZWUtOWY3YTBmZjMxODU1IiwiaWF0IjoxNDM2MjI2MjQwfQ.",
   "client_id": "ABCInc.MapApp",
   "refresh_token": "EQ90um+/ODi7Gf1E98CSXNIhIVUY8w96n+6vKf5JtWwr2awGQoACS8+RJkINGl1T50JQSVbLfxNhMPYb50Wv/t1ULeNdiPHrVqU1r/1wkjw56M1xhYSkkrXJsS25KCfHecV8lbrCt9d80ZASR6QrHd1O5/gWQ3Hzg09xCefVLm2Apq1ZRihWUIx2CEQU6SR+0U6cNbbtY7JdW/iwhD2ygPW40deguOrqYHtwZKqG/vSR1InWMBEzRNC2EZmSKkAfx+qoQa8ZDGcLRMvn3d4Jqc2W57YNzpOSTu/z8+Yeiob1Eeg3Ocse44yivDEkv9N82AgLSxpkQgWhZglkh+OgSnFU0Lt5dvQl/KuRb8+CgdFAg6usaUU+NYf/31pp68kZb5G4+7bxcUjAjPG7BVADWgIAAQ=="
}
```

## 3. Passing Tradeshift User Identity to Apps

`id_token`：描述了标准的JWT实现，而JWT是在[RFC7519](https://www.rfc-editor.org/rfc/rfc7519.txt)中实现的，JWT的Token中包含了所有App在运行上下文环境中可理解的一切。如果这个App和系统做集成则它将拥有自己的用户控件，也鼓励用户在Tradeshift用户ID和内部用户ID之间定义一个映射，这种方式中App自己可以为当前用户渲染相关属性是[JWT](https://www.rfc-editor.org/rfc/rfc7519.txt)中实现的，JWT的Token中包含了所有App在运行上下文环境中可理解的一切。如果这个App和系统做集成则它将拥有自己的用户控件，也鼓励用户在Tradeshift用户ID和内部用户ID之间定义一个映射，这种方式中App自己可以为当前用户渲染相关属性。JWT\) Token是未签名的，也就是说 alg = none，不包含任何数字签名信息；尽管如此，因为它是从指定请求的响应中获取的，所以不需要针对当前token进行签名验证。

这是解码过后Tradeshift传入到客户端的数据：

```json
{
  "alg": "none"
}
```
```json
{
  "exp": 1436226270,
  "sub": "sma+sarahbrown@tradeshift.com",
  "userId": "21d25c92-c0f7-44fd-bb38-38bf6fba740f",
  "aud": [
    "ABCInc.MapApp"
  ],
  "iss": "https:\/\/api-sandbox.tradeshift.com\/tradeshift\/",
  "jti": "j2arAxQ4rN0UUBmj",
  "companyId": "xxxxxxxx-6cb5-49fe-afee-9f7a0ff31855",
  "iat": 1436226240
}
.....[signature]
```

上边的信息含义包括（大部分JWT，最后两个是Tradeshift指定）：

* **exp**：超时时间Expiration Time
* **sub**：标题（Subject）或者Tradeshift的用户Email
* **aud**：一般这里使用App Name，表示接收人（Audience）
* **iss**：发放者，或Server的URL用来发放JWT令牌
* **jti：**JWT唯一标识符
* **iat**：最后一次发放时间
* **companyId**：激活Tradeshift中应用的组织的ID
* **userId**：Tradeshift用户的UUID

### SSO

用户若登陆到Tradeshift，那么SSO需要在用户账号中配置，当应用调用时，该用户已经是登陆状态。初始化请求期间，应用程序本身可以使用常规的OAuth2流程找出运行用户，并获取Tradeshift中通过`id_token`解码过的JWT令牌，然后应用程序可适当设置当前会话和当前用户，包括是否需要连接到其他API。

## 4. Permissions

你的App若调用Tradeshift Api，则当激活该应用时，用户需要设置相关权限。在Developer应用中，您可以控制应用所需权限。当激活该应用时，用户或管理员将检查所需的权限，尽可能保证您仅仅启用您所需要的权限，下边加锁的权限则没有办法修改。——权限信息是映射到OAuth2的域中。

## 5. Webhooks

通常，嵌入的应用需要和Tradeshift进行交互，如：当一个发票【invoice】或采购订单【purchase order】被一个公司激活的应用接收过后，这个app也许需要回应——要么存储一个文档、要么执行其他行为。

Webhooks允许您和Tradeshift中的一些特殊的事件【Event】交互，当其中某一个事件发生时，我们叫发送一个HTTP POST请求到Webhook中配置的URL，而Webhook是在应用级别配置的，当它触发时，它们将根据公司安装时的一些参数发送信息，这个事件的正文包含了额外的信息：如文档已经收到等。您可以在Developer的App中配置Webhook，滑动到Webhook章节，点击Add，则需要填写Webhook的名称以及提供POST请求将发送的地址。

如（`AFTER_DOCUMENT_RECEIVE`）

```
POST?event=AFTER_DOCUMENT_RECEIVE&id={event uuid}&tsUserId={user uuid}&tsCompanyAccountId={account uuid}&tsUserLanguage={user language}
```

**Parameters**

* **event**：事件名称，如：`AFTER_DOCUMENT_RECEIVE`
* **id**：事件ID
* **tsUserLanguage**：用户使用语言，如：`en_UK`
* **tsUserId**：用户关联的ID——您将在创建用户账号时得到用户的ID
* **tsCompanyAccountId**：激活App的公司

**Tips**：如果您想要查看请求内容，则您可以直接将webhook配置到[https://requestb.in/或者https://hookbin.com/。](https://requestb.in/或者https://hookbin.com/。)

### Webhook Events

当配置一个webhook时，您可选择哪个事件您将接收数据，文档相关【Document-related】的webhook当前仅仅支持的文档类型为：Invoice、Order、OrderChange。

文档发送事件

* `DOCUMENT_VALIDATING`：第一步，当您发送文档之前，验证文档中是否包含了所有信息。
* `DOCUMENT_SENDING`：开始发送Document，等待接收的信息。
* `DOCUMENT_SENT`：最后一步，发送文档完成。

文档接收事件

* `BEFORE_DOCUMENT_RECEIVE`：当文档被接收者接收之前触发。
* `AFTER_DOCUMENT_RECEIVE`：当文档全部被接收过后触发。

网络连接事件

* `NETWORK_REQUEST_RECEIVED`：公司接收到一个新的Network连接请求时触发。
* `NETWORK_REQUEST_ACCEPTED`：其他的Party接收当前网络连接时触发。
* `NETWORK_RESPONSE_RECEIVED`：接收到网络请求响应时触发。

## 6. API Usage最佳实践

Tradeshift REST API可以有很多用途，这儿有一些开发过程中的最佳实践。

### 6.1.客户端考虑

* 在所有请求中发送User-Agent请求头，该请求头可用于标识您的客户端、版本，如：BananaCorpClient/3.4；
* 在所有请求中发送Accept请求头如application/json、text/xml来标识客户端偏好；
* 所有的请求都可能因为某些原因而失败，对于应用错误，通常使用相同的错误结构，所以处理错误的方式也统一。

### 6.2.文档同步

* 常用的API主要用于文档同步，有可能直接同步改变的文档资源部分。这里有两种策略：
  * 从上一次Poll到这一次Poll的改变，这种情况仅仅做一个Poll，保存时间戳，包括下一次Poll中的`since`查询参数。
  * 针对不可知的文档执行Poll，当Poll不可知的文档时，直接执行一次Poll，迭代所有收到的文档，使用地址`external/documents/{documentId}/tags/{tag}`资源来PUT一个自定义的Tag到文档中，如`POLLED_BY_BANANACORP`，所有针对external/documents的资源应该不包括`tag=POLLED_BY_BANANACORP`查询参数，仅仅接收没有对应tag的文档。
* There的API是没有限制的——也就是说您不需要对所有的请求都过于小心，因为文档Transfer过程在Tradeshift中是异步调用，这儿不需要Poll太多因为每隔5分钟可以同步一次。
* **Sending Documents**：当发送文档时，使用了异步调用方式处理，即使分发请求返回了200 OK代码，或者这个分发【dispatch】之后会失败，如Email边界、接收方的自定义验证规则等。若您想要确认文档被发送，则可访问：`external/documents/{documentId}/dispatches/latest`资源地址直到状态变成`COMPLETED`，若状态变成了`FAILED`，则dispatch过程失败，响应信息中还会包含其他错误信息。如果它是`FAILED_TRANSIENT`，则dispatch失败，所以它可以制动进行。

