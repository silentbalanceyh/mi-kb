# Zero基本开发规范

## 0. 术语

* &lt;app&gt;：应用程序缩写，表示项目本身的名字。
* &lt;module&gt;：模块缩写，表示项目中某个服务的名字。

## 1. 包名规范

Zero中由于使用了微服务的整体结构，所以对于使用Zero开发的某一个项目而言，基本的包规范如下：

根包名直接使用：`com.<app>`的格式。

| 包名 | 职责 | 含义 |
| :--- | :--- | :--- |
| aop | AOP切面专用包 |  |
| cv | 全局常量文件包 | Constant Value的缩写 |
| domain | 领域模型包 | Jooq生成，包含了Domain部分的领域模型，也包含生成的Dao文件。 |
| epic | 可共享Monad专用工具类 | 可用于调用远程Rpc，也可做服务通信客户端，或者初始化、响应处理某一类的基础工作，属于原来业务逻辑层中类似Helper的功能。——这里的静态方法可直接作为Future的compose方法的参数来处理。 |
| exception | 全局异常专用包 | 用于定义一些业务层面的全局异常信息。 |
| ipc | 服务内部通信Rpc服务端 | 当微服务相互通信的时候，这个包中放的就是内部通讯时开放给其他服务的Rpc接口。 |
| micro.&lt;module&gt; | 某个服务所有组件 | 定义某个服务中的所有组件信息，包括RESTful的Api、Agent/Worker、业务逻辑层接口和实现（Stub/Service）。 |
| wall | 当前服务的安全组件 | 墙，不论安全与否，墙这个词语对于中国人而言，一定会联想到安全相关。 |

## 2. 类名规范

| 包 | 示例类名（I-接口/C-类） | 规则/示例 | 含义 |
| :--- | :--- | :--- | :--- |
| aop | AuditAop \[C\] | 直接使用Aop后缀 | 直接告诉开发人员这个类就是Aop部分的代码。 |
| cv | Addr \[I\] | 地址常量 | Address的简称，连接Agent和Worker的EventBus中的地址表。 |
| cv | Pojo \[I\] | 领域模型映射常量 Pojo | 使用Pojo直接会让开发人员联想到和Java中的Domain相关。 |
| epic | OrderAider \[C\] | 使用Aider后缀 | aider的含义在英文翻译中有：辅助、后援的意思，实际上是业务逻辑层的辅助工具类。 |
| exception | HotelMissingException \[C\] | 所有异常类直接使用Exception后缀 |  |
| ipc | MenuIvy \[C\] | 统一使用Ivy的后缀命名 | Ivy有常春藤的含义，由于这个包中包含的是开放给内部服务的Rpc接口，Rpc又是长连接，一旦调用后会保持，加上Ivy的名字比较短，所以用了该命名。 |
| micro.&lt;module&gt; | RoomApi \[I\] | 所有RESTful接口类统一使用Api后缀 |  |
|  | RoomIrApi \[I\] | 查询和基础接口分离的时候，查询类的使用IrApi后缀 | Ir全称：Information Retrieval，信息检索。 |
|  | RoomStub \[I\] | 业务逻辑接口类统一使用Stub的后缀 | Stub在Java语言的RMI中有存根的意思，实际上是不带实现的远程调用接口。 |
|  | RoomAgent \[C\] | 非接口模式的Zero RESTful接口类统一使用Agent后缀 |  |
|  | RoomWorker \[C\] | 后台Worker线程专用，统一使用Worker后缀 |  |
|  | RoomService \[C\] | 实现了Stub的业务逻辑实现类 | Service就是服务，不是接口，直接使用该名字描述服务的内容。 |

## 3. 其他规范

### 3.1. Addr中的变量名和字符串值

本文中的规范是Zero中我们实战常用规范，不一定是最优化，但开发人员遵循这个规范来开发项目，可以将Zero用得更加得心应手。

| 规则 | 示例 |
| :--- | :--- |
| Addr的地址名称统一使用 EVENT://ADDR/ 前缀，全大写 | EVENT://ADDR/PAYBILL/PUT |
| Addr中的变量名和字符串中的文字一致 | PAYBILL\_PUT |

示例代码

```java
String PAYBILL_PUT = "EVENT://ADDR/PAYBILL/PUT";
```

### 3.2. Api中的方法定义

Zero中的API、方法定义相关信息相对比较复杂，它的完整规范参考：[SPC-002 API和方法设计规范](/specification/1-zerogui-fan/spc-002-apihe-fang-fa-she-ji-gui-fan.html)。

### 3.3. 服务通信

#### 3.3.1. 服务端

在Zero中开放Rpc服务端的代码如下：

```java
package com.htl.ipc;

import com.htl.micro.activity.ActivityStub;
import io.vertx.core.Future;
import io.vertx.core.json.JsonObject;
import io.vertx.up.annotations.Ipc;
import io.vertx.up.atom.Envelop;
import io.vertx.up.log.Annal;

import javax.inject.Inject;


public class TabularIvy {

    @Ipc("IPC://ADDR/TABULARS/MULTI")
    public Future<JsonObject> listAndMulti(final Envelop evenelop) {
        final JsonObject params = evenelop.data();
        return Ux.Jooq.on(SysTabularDao.class).on(Pojo.TABULAR)
                .fetchAndAsync(new JsonObject()
                        .put("sigma", params.getString("sigma"))
                        .put("type,i", params.getJsonArray("type").getList()))
                .compose(Ux.fnJArray(Pojo.TABULAR))
                .compose(array -> Ux.thenGroup(array, "type"));
    }
}
```

#### 3.3.2. 客户端

```java
    public static Future<JsonObject> rpcTabular(
            final String sigma,
            final JsonArray type) {
        final JsonObject params = new JsonObject()
                .put("sigma", sigma).put("type", type);
        return Ux.thenRpc("ipc-datum", "IPC://ADDR/TABULARS/MULTI", params);
    }
```

> 注：上边可以是静态方法，也可以是非静态方法。

所以针对服务通信的基本规范如下：

* 所有的@Ipc内的地址统一使用`IPC://ADDR`前缀标识；
* 客户端调用统一使用`Ux.thenRpc`方式调用；

### 3.4. 组件概念

通常我们会把Zero的前后端线程进行分离，接受请求线程和处理请求线程会使用Event Bus进行异步数据通信：

| 组件名称 | 职责 |
| :--- | :--- |
| Agent | 通常又称为接收请求的线程主体 |
| Handler | 处理器，接收请求线程主体中的执行线程，真正接收线程过程中的执行主体 |
| Sender | 往事件总线Event Bus发送数据的专用执行线程，一般位于Handler链的最后 |
| Worker | 处理请求的线程主体 |
| Consumer | 消费事件总线Event Bus中数据的专员执行线程，真正消费数据的执行主体，可执行Block的动作如IO操作、数据库访问、网络访问等 |

上边组件的基本关系如下：

* 一个Agent中可以包含多个Handler；
* 当多个Handler组成一个Handler的链式结构时，若启用了Event Bus，那么Sender一般位于最后，发送数据到Event Bus的Handler称为Sender；
* 一个Worker中可以包含多个Consumer，每个地址上会绑定一个Consumer；

### 3.5. Worker和Service通信

后端的Worker和Service通信的过程中，一般遵循两个基本规则：

* 如果业务逻辑不复杂，如单纯的CRUD操作时，可直接在Worker中调用`Ux.Jooq`完成相关动作，这种情况下可不使用`Stub/Service`的业务逻辑层结构。
* 当业务逻辑层出现了超过两次以上的`compose`对应的Monad运算操作时，开启`Stub/Service`的业务逻辑层来执行相关计算。
* 如果业务逻辑层的接口存在“复用”，同样需要开启`Stub/Service`的业务逻辑层来进行复用分离，直接在`Stub`中定义业务逻辑层接口。

### 3.6. 使用Helper

在Worker和Service中的核心代码部分，尽可能只包含过程，也就是说不在主体代码中塞逻辑，代码参考如下：

```java
    public Future<JsonObject> createDirect(final JsonObject data) {
        LOGGER.info("[ Hotel ] 直接入住: \n{0}", data.encodePrettily());
        // 后续添加入住记录使用
        data.put("datum", data.getJsonArray("travelers").copy());
        return TicketAid.asyncItems(data)
                // 创建订单
                .compose(processed -> OrderAider.createOrder(processed, "Registered"))
                // 创建订单项
                .compose(OrderAider::createOrderItem)
                // 保存宾客记录，目前更新 1)宾客状态，2) 入住次数。其它字段不在本处处理中更新
                .compose(OccupAider::saveTravelers)
                // 保存入住记录
                .compose(OccupAider::createInoccup)
                // 初始化账本
                .compose(AtmAider::createAccountBook)
                // 更新房间信息，更新成占用房
                .compose(order -> this.statusStub.updateEtat(
                        order.getString("sigma"),
                        TicketAid.getEtatParam(order),
                        null)
                        .compose(nil -> Future.succeededFuture(order)));
    }
```

这种模式下，可借用`Aider`中的可重用方法执行函数引用的注入、替换等相关操作，并且代码结构上会清晰很多。

