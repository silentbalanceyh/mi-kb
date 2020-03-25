# 第一个Zero

> 本篇主要介绍如何开发第一个运行在Zero中的程序，若要给Zero一个定义，那么我能给的就是：基于`vert.x`的微服务应用框架。

## 1.JSR311规范

关于RESTful的基础，可以参考《逐陆记》中写过的：[正确打开REST](https://silentbalanceyh.gitbooks.io/vert-x/content/2jin-se-de-xing-guang/21restfulji-chu.html)，这里我就不再重复。

JSR311是什么？JSR311是曾经的Sun公司在2007年2月发布的Java规范：RESTful Web服务的Java API，同年9月23日，规范的1.0草案通过了JCP执行委员会的赞成投票，这基本意味着它已经定稿了。

> 小知识：
> JCP的全称为：Java Community Process，Java社区进程，成立于1998年，由社会各界Java组成的社区、规划和引导Java发展的组织。
> JSR的全称为：Java Specification Requests，Java规范请求，由JCP成员向委员会提交的Java发展议案，经过一系列的流程后，如果通过则会体现在未来的Java语言中。

[JAX-RS](https://jsr311.dev.java.net/)是Java中用于实现以HTTP协议为基础的RESTful Web服务的基于注解的API，本质上——注解类和方法的信息能让运行时（Runtime）将它们发布为资源，这种方法和通过Servlet编程模型来发布类和方法的做法上会有很大的区别。实现JAX-RS的运行时（Runtime）周旋于HTTP协议和Java类之间，需要考虑URI、被请求和被接受的内容类型、HTTP方法。

JSR311的官方实现是Sun提供的[Jersey](https://jersey.dev.java.net/)，除此之外，可用的其他实现都可以称为支持JSR311的应用框架。原生的`vert.x`是不支持JSR311的，Zero中考虑到支持JSR311，而不是自己定义一套新的RESTful的注解，主要是基于开发人员的学习曲线考虑，而Zero本身也对JSR311规范有很大力度的支持，这样开发人员就不需要在使用Zero的过程中重新去学习。

> 上一个章节已经提到过，Zero中所有的案例都是在项目`vertx-zeus`中，所以初学者可以参考这个项目搭建开发环境。

## 2.启动模式

Zero的启动模式主要分为两种：独立模式/微服务模式

### 2.1.独立模式

【项目】

* `vertx-zeus/up-rhea`：独立模式运行

在独立模式中，Zero本身可以像spring-boot的单机Web应用一样直接运行，而且启动很简单，这也是“零”的意义。

```java
package up.god;

import io.vertx.up.VertxApplication;
import io.vertx.up.annotations.Up;

@Up
public class io.god.Anchor {
    public static void main(final String[] args) {
        VertxApplication.run(io.god.Anchor.class);
    }
}
```

直接运行上述代码，就可以启动Zero了，有一点需要说明的是：**Zero的默认端口是6083**。

### 2.2.微服务模式（初学可跳过）

【项目】

* `vertx-zeus/up-athena`：微服务模式中的ApiGateway，端口号：6100
* `vertx-zeus/up-uranus`：微服务模式中的Service节点，端口号：6888

微服务模式下的Zero没有spring那么自由，有一定的限制，如下：

* Zero的服务注册中心和配置中心统一使用了`etcd3`。
* Zero的微服务内部通讯统一使用了`gRPC`。
* Zero的`Api Gateway`要等所有服务Ready之后再启动（这应该是个BUG）。

#### 「壹」，启动etcd3──注册中心

最新的版本代码中，Zero已经将开发环境需要使用的`etcd3`的工具集成好了，开发者可以直接通过脚本启动它，脚本文件位于根目录：`./start-etcd3.sh`，内容如下：

```shell
#!/usr/bin/env bash
cd tool/e3w
docker-compose up
```

#### 「貳」，Yaml配置

虽然Zero支持零配置启动，但对于实际项目，真正实现“零”配置似乎只是一个理想，就像spring-boot一样，如果在实战过程中，还是存在一定的配置文件。所以：配置和部署也是一个开发人员的基本功，不要觉得那只是运维人员的事情。

* `vertx.yml`：Zero的核心配置文件，不论是独立模式还是微服务模式都会用到该文件。
* `vertx-server.yml`：Zero的容器配置，可配置端口号、角色（是Gateway还是服务）、以及所有和HTTP服务器相关的配置信息（一个Zero的实例可启动多个角色）。
* `vertx-etcd3.yml`：微服务模式才会使用的配置文件，用于配置etcd3实例信息。

##### vertx.yml

`vertx.yml`配置文件是Zero的“中枢神经”，其他所有可用的配置文件都是从这个文件的配置扩展而来，它的内容如下：

```yaml
zero:
  lime: etcd3
  vertx:
    instance:
    - name: vx-zero
      options:
        # Fix block 2000 limit issue.
        maxEventLoopExecuteTime: 30000000000
```

简单说明一下上述配置：

* `lime`节点主要用于扩展，它表示Zero使用的配置文件后缀，如果包含多个配置可以用逗号隔开。如上述配置中`lime: etcd3`，那么在类路径中，您必须存在一个文件：`vertx-etcd3.yml`——它表示：`vertx-etcd3.yml`是从`vertx.yml`扩展出来的扩展配置。
* `vertx`节点中的`instance -> options`则直接和`vert.x`中的`Options`连接，包括它所使用的属性信息，详细内容可参考：[VertxOptions](https://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html)。
* 除此之外，一个Zero实例中可以运行多个`vert.x`实例，这种设计仅仅是针对一个Zero多个`vert.x`实例模式，但在Zero实际运行的项目中，这种用法**未经过生产环境验证**，所以这一点需要读者小心。

##### vertx-server.yml

由于在微服务模式下，Gateway和Service扮演了不同的角色，所以这个配置文件的内容可能会根据角色有所不同：

*Gateway*：

```yaml
server:
- name: up-athena
  type: api
  config:
    port: 6100
    host: 0.0.0.0
```

*Service*：

```yaml
server:
- name: up-uranus
  type: http
  config:
    port: 6888
    host: 0.0.0.0
```

简单说明一下上述配置：

* 一个Zero实例中，除了可以支持多个`vert.x`实例以外，还可以支持多个服务器实例（原生`vert.x`是支持的）。
* 上述配置的`type`用于区分当前服务的角色，Zero中主要包含以下几种角色：
	* **http**：普通的Http服务器（微服务模式中的服务节点，独立模式中的Http服务器）。
	* **api**：仅用于微服务模式中的服务路由（Api Gateway）。
	* **ipc**：启用了服务通信时，当前Zero实例开放的内部gRPC服务器。
	* **sock**：启用WebSocket协议时，当前Zero实例开放的websocket服务器。
* 最后需要说明的是微服务模式中，运行的每个Service服务实例的名称必须是唯一的，就像配置中的`up-uranus`和`up-athena`，它提供了当前服务的默认配置名空间。

##### vertx-etcd3.yml

```yaml
etcd:
  micro: zero-istio 
  nodes:
  - host: localhost
    port: 6181
  - host: localhost
    port: 6180
  - host: localhost
    port: 6179
  timeout: 2
```

简单说明一下上述配置：

* Zero中默认的etcd3并不是一个单实例，而是一个三节点实例的集群，这和它使用的[e3w](https://github.com/soyking/e3w)项目有关，如果开发人员要管理etcd3中的配置信息，可使用工具[Etcd Viewer](https://github.com/nikfoundas/etcd-viewer)。
* 对于一个完整应用中的服务实例（包括Gateway和Service）而言，这个应用的名称是统一的（唯一且相同），上述配置中的`micro`节点的值，该值会在etcd3中创建独一无二的配置数据节点。

> 这个章节对于最开始接触Zero的初学者可以跳过，里面的大部分信息会在后续的微服务模式中详细说明，所以若读者消化起来困难也不要着急。

## 3.第一个Zero

任何一个软件的新技术，往往都会使用类似`Hello World`的程序开始，而Zero中，我们就不使用它了，所以在官方我称为「Origin（起源）」，也许个人觉得这样的开场更加梦幻；前文已经介绍过Zero中的环境配置，这里就不重复，本章节我们来写一个最简单的Zero，也就是第二章节中提到的「独立模式」。

> 直接让初学者从微服务模式出发，可能会带来很大的学习曲线，所以后续教程中的大部分内容会像官方文档一样，在「独立模式」下先做尝试。初学`vert.x`的人一直对一个问题不太理解：vert.x究竟能做什么？所以很多人会把它和spring、或者spring-boot直接做比较，不过很可惜，它们不是同一个维度的东西。vert.x实际上是一个“工具箱”——它不是像很多人所想象的是为了RESTful而设计的一个“工具箱”，所以才会有`vertx-web`项目登场，——Zero只是在`vertx-web`的基础之上的框架，它是面向RESTful的应用程序框架，主要是让一些基础开发人员更容易上手，屏蔽了很多原生的`vert.x`的细节——实际上在我看来，它束缚了`vert.x`的能力。

一个最简单的Zero的RESTful应用你可以写成下边这种样子（别忘了前文提到过的启动器）：

```java
package up.god.micro.origin;

import io.vertx.up.annotations.EndPoint;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@EndPoint
@Path("/api")
public class FirstHi {

    @GET
    @Path("hi")
    public String hi(@QueryParam("name") final String name) {
        return null == name ?
                "Hi, Input your name" :
                "Hi " + name + ", welcome to Origin";
    }
}
```

好了，一个程序完成了，启动Zero，您就可以打开：<http://localhost:6083/api/hi> 测试这个刚刚写过的接口了，甚至您还可以带上一个参数：<http://localhost:6083/api/hi?name=Lang>，我曾说过这个框架的名字叫做Zero Up，那么到此处，一个简单的类似`Hello World`的程序完成了——**Zero is Up！**

### 让人困惑的data？

细心的读者会发现，发送请求后，您将收到下边的响应数据：

```json
{
	"data":"Hi Lang, welcome to Origin"
}
```

可能很多自由的开发人员不太习惯这样的做法，觉得这样的数据格式是一种限制，甚至于让它们更喜欢发送请求后收到的是直接的一句：`Hi Lang, welcome to Origin`——实际上Zero这种设计是故意而为之。在很多RESTful的业务场景中，目前对于企业服务这个范畴，使用最多的数据格式是JSON，而这样的设计主要是为了解决以下几个问题：

* 统一处理简单返回值和复杂对象，方便前端UI程序做统一解析，比如上述返回，前端会直接提取`data`节点的数据，如果`data`本身是一个JSON对象（通常在后端是反序列化Pojo的结果），那么前端程序在解析响应的过程中可以统一。
* 定义轻量级数据基本规范，在很多业务场景中，我们都会使用到插件、数据驱动、配置驱动这些从元数据出发的程序，在这种情况下，直接在`data`节点添加扩展（不论是扩展配置还是脚本），那么前端程序依然可以维持统一解析流程，这给前端工程师带来了很大的便利。

举个例子，如果现在返回的数据格式如下：

```json
{
	"data":{
	},
	"config":{
	},
	"script":"Ux.resolveTraffic"
}
```

上述响应格式存在于Zero的真实生产环境中，实际上它完成了这样一个事情：对于返回的数据`data`而言，前端使用了动态解析流程，当数据返回到前端，会使用`script`节点中配置的脚本来执行，执行规则存储在`config`节点中，当执行脚本读取了执行规则后，会解析`data`的数据，然后让前端来做交互和展现。简单说——Zero中引入了侵入性很强的设计，我不保证这种设计一定是你需要的，但它在我们的生产环境中确实解决了很多实际问题；在一种理想的状态下，我们总是希望前后端分离，通过一个接口就将二者联系在一起，而这个接口处我们可以很自由，可又有几个团队真正做到了“自由”和“轻松”？真实的情况下，前端改了调用接口，后端改了发布接口比比皆是，毕竟没有一个软件系统是一蹴而就的，所以从我个人使用Zero的心得来说，它只是想帮着开发人员多做点事而已。

## 4.Zero启动设计心得

最开始在设计Zero容器时，一直比较担心一个问题——当使用了类似注解扫描的方式过后，会不会对`vert.x`本身的性能有所耗损，所以在实现该容器时针对这一块内容做过很多“重构”，这里分享一下设计过程，让读者也对Zero本身有个基本概念上的认知。

### 4.1. Zero的启动原理

1. Zero容器在启动的时候并不会扫描所有的Java包，而只是扫描用户自己定义的`package`，也就是说一些比较特殊的包会直接被滤掉，比如`java`、`javax`等这些常用的第三方包都不会出现在Zero的扫描范围中（这里只是取巧的做法），详情可参考源代码`io.vertx.zero.mirror.Pack`类中的`FORBIDDEN`定义。
2. 针对所有Zero中定义的注解，Zero会开启多线程扫描模式，并且在启动时一次性扫描并将结果存储在内存中，不论是扫描到的类、方法还是成员，都会在Zero中初始化成定义的数据结构并存储，在扫描完成之前，并不会启动`vert.x`容器——也可以将扫描过程理解为容器的“预启动”过程。
3. Zero中所有的组件大面积使用了单例、池化单例（按Map分流）、线程单例（和Router绑定）的模式，这种模式下，可以保证过度创建这一类“职责对象”，但凡和数据对象无关的部分，Zero都会以`singleton`的模式出现。
4. Zero启动器在启动过程中使用了`VertxApplication.run`的API去启动，但对于开启了`etcd`模式的会出现一次检查，而调用内部的`DansApplication.run`来启动`Api Gateway`（主要是Api Gateway的执行流程和Zero本身的服务执行流程大相径庭）。

>  综上所述，从目前在生产环境中的运行情况看来，vert.x的性能并没有Zero的出现而有所减损，至于将来是否会发现相关的BUG，这个只能拭目以待。

### 4.2. Zero对JSR的适配

由于Spring中定义了类似`@HttpMethod`的注解，最开始在RESTful这一块，Zero也打算定义自己的注解来描述Web服务，但经过了长时间的重考，最终还是回归到`JSR`规范中。

1. **JSR311**——Zero并没有完全支持JSR311，像`@MatrixParam`这种不太常用的注解，Zero中并不支持，更多的是支持实际项目过程中更加常用的部分，支持表可参考Zero的官方文档。
2. **JSR330**——对于依赖注入部分，Zero可以说是实现了一个简单的DI容器（这部分将来可能会面临很大程度的重构），一方面因为没使用第三方的DI容器，显得Zero本身很纯，另外一方面主要是仅仅支持了`singleton`模式，使得Zero在针对JSR330的支持过程中略微有些限制，但参考实际项目这些限制并不会影响实际的项目开发。
3. **JSR303**——对于目前出现过的所有1.x和2.x的`Bean Validation`的部分，Zero是支持的，底层使用了`hibernate-validator`，不仅仅如此，还在JSR303的基础之上进行了扩展。
4. **JSR340**——这部分可能是Java开发人员最**水土不服**的地方，主要原因是`vert.x`中的请求和响应对象使用了自己定义的内容，并没有使用类似`HttpServletRequest`这种，这样使得兼容完整度比较高的JSR340变得有些棘手，所以Zero提供了兼容方式，尽可能保证语义层面不会有太大的变动。
5. **扩展**——“也许这就是传说中的达摩克里斯之剑！”，Zero围绕`vert.x`中出现的`JsonObject、JsonArray、Buffer`等数据结构进行了JSR的扩展，官方成为`Zero JSR`，主要目的是给开发人员提供更加强大的工具，并且让“数据内容”的验证、读取变得更加容易解析，同时还定义了许多内置的`WebException`，拥有自己的一套容错规范。

>  支持JSR原生Annotation，并不是定义新的Annotation，其目的就是保证Java开发人员更加熟悉，并且不需要去记忆太多内容就可以上手，所以也可以毫不含糊地说——Zero的目的就是为了偷懒，并且是在vert.x的生态中偷懒。

## 5.结语

到此处，关于Zero的启航篇，似乎就没有什么需要再讲的了，如果它只是一个纯粹的技术性的框架，那么像`Utility X`这种东西就完全没有存在的必要。可是在我们开发的很多企业系统中，业务复杂度有时候像井喷一样，如果开发人员仅专注于技术本身，那么很有可能就会出现“产品经理和开发打架”的局面。其实我一直都不觉得现今的企业系统很成熟，大部分公司内部使用的系统都是买个A、买个B、外包个C来连接A和B，然后循环、然后继续——如果你还沉醉于这样的模式中，或者说你还一直没有意识到这部分成本的开销，那么企业的IT成本将会成为一种灾难。