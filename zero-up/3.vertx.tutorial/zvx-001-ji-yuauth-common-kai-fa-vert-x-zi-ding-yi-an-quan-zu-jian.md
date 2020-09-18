# 基于Auth Common开发自定义认证组件

本文主要介绍vert.x中的认证授权流程，主要包含下边内容：

* 分析Vert.x中的认证授权框架（以auth-common项目为例）
* 开发自定义的完整独立认证/授权流程组件

*注意：代码示例中的代码段来自于真实项目，可能复杂度比简单的DEMO要大，希望读者耐心去分析。*
## 术语表

* Handler：用于表达Vert.x Web中Router注册的Handler组件，该组件通常实现了接口`Handler<RoutingContext>`。
* User：auth-common中定义的被认证的实体接口；
* Provider/AuthProvider：两个术语混用，本文中表示同样的意思，auth-common中定义的认证提供者组件。

## 1. 楔子

提到系统中认证授权，一般会使用RBAC（Role-Based Access Control）模型，该模型实际上描述了一个问题：Who, What, How——“Who对What进行了How的操作。”

* Who：权限拥有者（如Role、User、Group、Principal等）
* What：权限作用的主体对象，一般称为资源（Resource）
* How：具体的权限（Privilege）

本文不详细介绍该模型，细节可参考：[https://en.wikipedia.org/wiki/Role-based\_access\_control](https://en.wikipedia.org/wiki/Role-based_access_control)，本文主要对Vert.x中的认证授权部分进行详细介绍，分享在实际项目中用来处理认证授权部分的内容，官方文档地址：[http://vertx.io/docs/\#authentication\_and\_authorisation](http://vertx.io/docs/#authentication_and_authorisation)。

## 2. 深入Vert.x认证授权

Vert.x中主要包含了七个项目来处理认证授权流程，vertx-auth-common是Vert.x中认证和授权的接口定义，先看看官方项目中Vert.x Web如何设置Basic认证专用Handler的。

```java
AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

// All requests to paths starting with '/private/' will be protected
router.route("/private/*").handler(basicAuthHandler);
```

如果上述代码第一次出现在你眼前，可能对于初学者有点不容易理解，本章节的目的就是解决初学者在Vert.x认证和授权过程中的困惑。Vert.x中对于认证和授权部分定义了几个核心接口（vertx-auth-common中），这些接口可以让开发人员创建属于自己的认证、授权逻辑。上述代码段中，使用`BasicAuthHandler`类去创建了一个Handler组件，通常它创建的Handler组件和我们使用lambda表达式直接写的组件区别就在于——引入了认证过程中的Provider接口。在路由Route内的代码执行流程中，Provider的实现组件认证逻辑会被调用，而Provider实现组件的逻辑就是开发人员需要关注的地方。vertx-auth-common中的几个核心接口（包括抽象类）如下：

* `io.vertx.ext.auth.User`：被认证的实体，包含了认证授权中该实体包含的所有数据信息。
* `io.vertx.ext.auth.AuthProvider`：认证专用接口。
* `io.vertx.ext.auth.AbstractUser`：实现了User接口的抽象类，抽象类的主体逻辑实现了简单的权限缓存、标准权限处理流程。

### 2.1. `BasicAuthHandler`做了什么？

由于Vert.x中的在Router处理Handler的过程中官方示例都是使用的lambda表达式的写法，很容易让读者养成了一种固定习惯，那么在了解`BasicAuthHandler`之前先看看Handler的两种写法，Vert.x内置的很多Handler使用的并不是lambda表达式的写法，而是直接定义——其目的不言而言，就是为了“职责封装”，而不是将大段大段的代码写在Verticle的start方法内部。如：

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
```

大家有时候会好奇，因为下边有另外一种写法：

```java
router.route("/private/somepath").handler(routingContext -> {
  // This will have the value true
  boolean isAuthenticated = routingContext.user() != null;
});
```

本节主要对这两种写法进行剖析，最终使用哪种根据读者自己遇到的场景来定。先看一个改写的例子：

_（1）直接使用lambda表达式：_

```java
Metadata meta = new Metadata();
router.route("/api/*").handler(context -> {
    // 执行固定逻辑
    boolean isSecure = meta.isSecure();
    if(isSecure){
        // 执行额外逻辑
        // ......
    }else{
        context.next();
    }
});
```

上述代码表示这样一段逻辑：在**启动时**，构造一个`Metadata`对象（也可以是其他对象），并设置路由；在**执行请求时**，调用meta的`isSecure`方法，它的返回结果会影响请求的执行流程。实际上代码主体和lambda本身的逻辑在Vert.x**不是同时执行**的。

```java
Metadata meta = new Metadata();
```

上边代码段构造了Metadata对象，Vert.x中，它是在Verticle组件start方法调用时执行，也就是deploy阶段。

```java
    boolean isSecure = meta.isSecure();
```

上边代码是执行`Metadata`对象的isSecure方法，位于lambda表达式内部，它是在请求触发时被调用——在Vert.x的Verticle组件执行deploy过程中，这段代码并不会执行（不仅这段，Handler内部代码都不会被执行），理解透这两个生命周期过后，就可以对上边代码进行改写了（根据阅读代码的固有逻辑，如果读者对lambda中的方法引用不是很清楚，就不太容易理解这种写法的执行生命周期，相反，下边代码可能更容易理解一点）。

_（2）使用Handler定义_

将（1）代码改写成下边这种模式：定义一个额外的类`MetaHandler`用来创建Handler组件，并将`Metadata`对象的引用传给它。

主代码：

```java
Metadata meta = new Metadata();
MetaHandler handler = MetaHandler.create(meta);

router.route("/api/*").handler(handler);
```

Handler定义代码

```java
public class MetaHandler implements Handler<RoutingContext>{
    // 创建Handler的静态方法
    public static Handler<RoutingContext> create(final Metadata meta){
        return new MetaHandler(meta);
    }
    // 成员变量Metadata的对象引用
    private transient final Metadata reference;
    private MetaHandler(final Metadata reference){
        this.reference = reference;
    }
    @Override
    public void handle(final RoutingContext context){
        // 执行固定逻辑
        boolean isSecure = this.reference.isSecure();
        if(isSecure){
            // 执行额外逻辑
            // ......
        }else{
            context.next();
        }
    }
}
```

上边从读者最熟悉的lambda写法到Handler定义的改写，就很清楚了`BasicAuthHandler`那段代码的作用了，它和第二种的主代码逻辑是一致的。

### 2.2. `BasicAuthHandler`背后的结构

Vert.x中的`BasicAuthHandler`远比上边的Handler定义部分复杂，接下来的内容对于实现自定义的认证授权很有帮助，但由于是分析Vert.x的源代码，难免有些觉得枯燥，不过慢慢来。Vert.x中的`BasicAuthHandler`的代码定义如下：

```java
public interface BasicAuthHandler extends AuthHandler {
    String DEFAULT_REALM = "vertx-web";

    static AuthHandler create(AuthProvider authProvider) {
        return new BasicAuthHandlerImpl(authProvider, "vertx-web");
    }

    static AuthHandler create(AuthProvider authProvider, String realm) {
        return new BasicAuthHandlerImpl(authProvider, realm);
    }
}
```

在调用`BasicAuthHandler`的create方法时，返回值是`AuthHandler`，而我们在Handler定义中返回的类型应该是一个`Handler<RoutingContext>`，实际上`AuthHandler`就是一个`Handler<RoutingContext>`的子接口：

```java
public interface AuthHandler extends Handler<RoutingContext> {
    @Fluent
    AuthHandler addAuthority(String var1);

    @Fluent
    AuthHandler addAuthorities(Set<String> var1);

    void parseCredentials(RoutingContext var1, Handler<AsyncResult<JsonObject>> var2);

    void authorize(User var1, Handler<AsyncResult<Void>> var2);
}
```

分析最初的代码和我们自己定义Handler部分的代码：

```java
// 最初的认证授权代码（官方Demo）
AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);
// 自己的定义
MetaHandler handler = MetaHandler.create(meta);
```

上述两段代码最终都做了同样的事情，就是创建`Handler<RoutingContext>`对象，只有该对象会被Router识别（用于传递给Route中注册），目前还没有进入实现部分，在我们的代码中，简单利用了`MetaHandler`处理了实现，而Vert.x中的`BasicAuthHandler`实现则是通过`BasicAuthHandlerImpl`类来完成的。从我们定义的Handler部分可以发现实现部分的代码是请求流程执行时触发的，它的主逻辑在于调用`handle(RoutingContext)`方法，上述代码的实现类`BasicAuthHandlerImpl`中似乎找不到它的踪影？事实上Vert.x为了满足各种认证授权需求，进行了更加细粒度的设计，它的整个继承树结构如：

```java
class BasicAuthHandlerImpl extends AuthorizationAuthHandler{}

class AuthorizationAuthHandler extends AuthHandlerImpl{}

class AuthHandlerImpl implements AuthHandler{
    public void handle(RoutingContext ctx){
        // 请求主代码逻辑
    }
}
```

也就是说，真正在执行认证请求时候调用的是`AuthHandlerImpl`（顶层父类）中的`handle(RoutingContext)`方法，那么到这里，请求怎么来的，相信读者就能够理解了。

### 2.3. `BasicAuthHandler`背后的逻辑

既然已经找到了请求执行的入口，那么从入口开始分析BasicAuthHandlerImpl，让大家对整个请求流程有更加清晰的了解。为什么要分析？如果仅仅是针对开发人员，开发自定义的`User`和`Provider`足够完成任务，了解清楚这部分内容的目的是方便大家“知其所以然”，这样我们可以知道Vert.x究竟帮我们完成了什么事，这样从内到外抽丝剥茧，我们才知道自己开发的Provider和User最终是在整个认证授权框架的什么位置——如果我们要开发自定义的独立认证授权流程时从什么位置入手最符合实际项目的需要。先看下边的核心代码（方便大家理解，带上详细的注释）

**认证部分**

```java
    public void handle(RoutingContext ctx) {
        /**
         * 跨域访问中首次请求OPTIONS方法的判断逻辑，如果是OPTIONS则需要检查是否发送了请求头：
         * Access-Control-Request-Headers，如果包含了该请求头，那么需要检查对应的值中是否
         * 包含了认证需要的Authorization头信息，这个if判断描述了当前认证流程的入口条件。
         **/
        if (!this.handlePreflight(ctx)) {
            /**
             * 从RoutingContext中读取User对象
             **/
            User user = ctx.user();
            if (user != null) {
                // 授权：不解析请求头，直接从RoutingContext中拿到用户User执行授权
                this.authorizeUser(ctx, user);
            } else {
                /**
                 * 无法读取User信息，直接解析Http的头：Authorization，一般是第一次请求时调用
                 * 注意：parseCredentials方法是定义于AuthHandler中的
                 * 被解析的值格式一般是：Authorization: Schema XXXXXX，这里的Schema又称为认证模式，一般取下列值：
                 * Basic, Digest, Bearer, HOBA, Mutual, Negotiate, OAuth, SCRAM-SHA-1, SCRAM-SHA-256
                 * 不同的认证流程这个方法会被重写，因为解析逻辑会有所不同，BasicAuthHandlerImpl中的
                 * parseCredentials主要用于解析Basic中的头信息，格式如：Authorization: Basic XXXX
                 **/
                this.parseCredentials(ctx, (res) -> {
                    if (res.failed()) {
                        // 解析失败，直接报错
                        this.processException(ctx, res.cause());
                    } else {
                        // 解析成功，执行二次逻辑判断，判断Session中是否有当前用户，访问过一次的用户
                        // 会被缓存在Session会话对象中（默认行为）
                        User updatedUser = ctx.user();
                        if (updatedUser != null) {
                            Session session = ctx.session();
                            if (session != null) {
                                session.regenerateId();
                            }
                            // 授权：当前用户已经登陆过了，直接使用Session中的User对象执行授权
                            this.authorizeUser(ctx, updatedUser);
                        } else {
                            /**
                             * 从上述代码逻辑可以知道，直到主流程运行到这个位置，代码才真正到达Provider中，而这里就
                             * 会调用getAuthProvider方法去获取系统中已定义过的Provider，并且执行provider的
                             * 主逻辑authenticate方法。那么读者也许比较困惑的是res.result()返回的应该是什么，这里
                             * 返回的内容实际上是由上层调用parseCredentials来决定的，一般是一个JsonObject对象，但
                             * 具体数据由不同的Handler实现来决定，比如Basic类型的最后一行代码为：
                             * handler.handle(Future.succeededFuture(
                             *     (new JsonObject()).put("username", suser).put("password", spass))
                             * )
                             * 那么它返回的就是一个JsonObject对象，形如：
                             * {
                             *     "username":"xxxx"
                             *     "password":"xxxx"
                             * }
                             * 这也是官方教程中提到Basic的数据格式的原因，需要再提到的一点就是Provider接口会将一个
                             * JsonObject类型的对象转换成User被认证的实体对象，所以最终从authenticate第二参数返回
                             * 的数据类型就是User。
                             **/
                            this.getAuthProvider(ctx).authenticate((JsonObject)res.result(), 
                                (authN) -> {
                                    if (authN.succeeded()) {
                                        /**
                                         * 认证成功时，通过Provider获取认证的User对象，并将用户存储到Context中，
                                         * 并且根据认证基础信息创建会话
                                         **/
                                        User authenticated = (User)authN.result();
                                        ctx.setUser(authenticated);
                                        Session session = ctx.session();
                                        if (session != null) {
                                            session.regenerateId();
                                        }
                                        // 授权：认证成功，基础环境设置完成，执行授权流程
                                        this.authorizeUser(ctx, authenticated);
                                    } else {
                                        /**
                                         * authenticateHeader又是一个会被子类重写的方法，
                                         * 用于设置认证不成功时，
                                         * 在响应中提供WWW-Authenticate的头信息，
                                         * 对于Basic而言一般是Basic realm=xxx格式。
                                         * 记得区分请求头：Authorization和响应头WWW-Authenticate
                                         **/
                                        String header = this.authenticateHeader(ctx);
                                        if (header != null) {
                                            ctx.response().putHeader("WWW-Authenticate", header);
                                        }

                                        ctx.fail(401);
                                    }
                            });
                        }
                    }
                });
            }
        }
    }
```

**授权部分**（Vert.x中的权限模型比较简单）：

上述代码段中三次访问到授权流程的代码，即代码中的方法`authorizeUser`，这个方法内容如下：

```java
    private void authorizeUser(RoutingContext ctx, User user) {
        this.authorize(user, (authZ) -> {
            if (authZ.failed()) {
                this.processException(ctx, authZ.cause());
            } else {
                ctx.next();
            }
        });
    }
```

上述代码最终调用的方法还是`authorize`：

```java
    public void authorize(User user, Handler<AsyncResult<Void>> handler) {
        // 直接读取所需授权信息的数量（缓存过授权就会有该信息）
        int requiredcount = this.authorities.size();
        if (requiredcount > 0) {
            // 如果读取不到用户，则抛出FORBIDDEN的403异常信息
            if (user == null) {
                handler.handle(Future.failedFuture(FORBIDDEN));
                return;
            }
            AtomicInteger count = new AtomicInteger();
            AtomicBoolean sentFailure = new AtomicBoolean();
            // 执行一部授权检查，定义authHandler对象
            Handler<AsyncResult<Boolean>> authHandler = (res) -> {
                if (res.succeeded()) {
                    if (((Boolean)res.result()).booleanValue()) {
                        // 为空，则授权用户增加，使用AtomicInteger执行统计是因为多个线程共享，参考并发编程
                        if (count.incrementAndGet() == requiredcount) {
                            handler.handle(Future.succeededFuture());
                        }
                    } else if (sentFailure.compareAndSet(false, true)) {
                        // 出现异常，则抛出403的FORBIDDEN异常
                        handler.handle(Future.failedFuture(FORBIDDEN));
                    }
                } else {
                    handler.handle(Future.failedFuture(res.cause()));
                }

            };
            // 遍历每一个权限信息（authorities中存储了）
            Iterator var7 = this.authorities.iterator();

            while(var7.hasNext()) {
                String authority = (String)var7.next();
                if (!sentFailure.get()) {
                    // 针对每个权限信息调用User引用中的isAuthorised方法
                    user.isAuthorised(authority, authHandler);
                }
            }
        } else {
            handler.handle(Future.succeededFuture());
        }

    }
```

通过分析了认证和授权部分的源代码，就知道auth-common框架中的Provider和User的主方法在什么地方调用的：

* `Provider`：主要方法为`authenticate`方法，负责认证。
* `User`：主要方法为`isAuthorised`方法，负责授权。

至于Vert.x本身已经考虑了权限缓存、跨域OPTIONS的首次访问、开启用户会话的Session、以及默认情况下的403的处理等，当然这些内容细节读者可以参考Vert.x本身的源代码，这部分最后看看这两个主类的源代码定义（注释就不提供了）：

```java
// io.vertx.ext.auth.AuthProvider
@VertxGen
public interface AuthProvider {
  void authenticate(JsonObject authInfo, Handler<AsyncResult<User>> resultHandler);
}

// io.vertx.ext.auth.User
@VertxGen
public interface User {
  @Fluent
  User isAuthorized(String authority, Handler<AsyncResult<Boolean>> resultHandler);
  @Deprecated
  @Fluent
  default User isAuthorised(String authority, Handler<AsyncResult<Boolean>> resultHandler) {
    return isAuthorized(authority, resultHandler);
  }
  @Fluent
  User clearCache();
  JsonObject principal();
  void setAuthProvider(AuthProvider authProvider);
}
```

### 2.4.总结

需要注意的是：上边分析的BasicAuthHandler部分的内容是Vert.x Web项目的内容，并不是auth-common项目的内容，它们的整体结构图应该如下（仅红色和蓝色部分是连接auth-common的地方）：

![](/assets/images/zvx/001/auth-common.png)仔细结合源代码分析上述结构图，`BasicAuthHandler`和`BasicAuthHandlerImpl`这两个类都是vert.x web项目提供的内容，所有和认证授权相关的Handler部分在`io.vertx.ext.web.handler.impl`包中，如果你觉得下边的头信息不需要按照自己的逻辑进行解析，就不需要重写这两部分内容，否则的话，你也可以做深度定制，把这两个类重写。

```
[Basic认证]：Authorization: Basic XXXXXXX
[Digest认证]：Authorization: Digest realm="xxx", qop="auth", nonce="xxxx", opque="xxxx"
```

也就是说实现自定义的认证授权逻辑最简单的方式就是开发两个核心类，一个是AuthProvider（前文中提到的Provider），一个是User（实际上User的实现可以直接从AbstractUser继承，后边文章中会提到AbstractUser中的核心信息）。

## 3. 实现自定义认证授权流程

从第二节的分析中，我们已经知道了Vert.x中提供的认证授权框架的流程，那么本章节实现一个定制力度比较大的认证授权框架，一方面加深读者对第二节的理解，另外一方面让大家对认证授权的各种开发更加耳熟能详。实际上我们在项目开发过程中重写的目的有几点（不一定最优）：

* 重新定义了Token接口，解析流程参考了上边的parseCredentials方法，用于统一管理请求中的Authorization头；
* 参考Provider/User/Handler这几个结构，定义了新的CommonToken的基础模型来完成认证授权的重新定义；

### 3.1.基本类：定义User

Vert.x默认提供的AbstractUser可以称为是被认证授权的实体，主要包含三个核心逻辑：

* 实现了isAuthorized方法用于授权（授权模型使用`Set<String>`类型的权限集合）；
* 实现了doIsPermitted用于核心授权代码检查逻辑，和isAuthorized方法配合完成（二者可合并）；
* 实现了ClusterSerializable接口，用于在Vert.x的Cluster环境实现快速序列化和反序列化的操作；

那么在实现自定义的认证授权框架时，先定义属于自己的User，该User的定义如下：

```java
public class BasicUser extends AbstractUser {}
```

这里分析一下我们自己使用的Basic认证BasicUser类，定义的核心属性：

```java
    /**
     * User中需要使用的Provider对象引用
     **/
    @SuppressWarnings("unused")
    private transient AuthProvider provider;
    /**
     * 用户名信息
     **/
    private transient String username;
    /**
     * 用户Id信息
     **/
    private transient String id;
    /**
     * 用户的Password加密字符串
     **/
    private transient String password;
    /**
     * 用户授权信息
     **/
    private transient JsonObject principal;
```

这里扩展了原生的username和password两个字段，添加了id（数据库代理主键）和`JsonObject`类型的principal，一般principal存储了一些附加的数据（比如当前用户的角色、基本权限、资源集合等，这个根据实际需要而有所不同）。默认情况下如果要User对象在集群模式中可使用（应该说所有需要在集群模式中使用的数据对象），都会实现`io.vertx.core.shareddata.impl.ClusterSerializable`接口，AbstractUser本身实现了该接口，所以需要针对接口写类似下边的实现方法：

```java
    @Override
    public void writeToBuffer(final Buffer buffer) {
        /** 1.调用父类方法 **/
        super.writeToBuffer(buffer);
        /** 2.写入id，用户名和token **/
        Buffalo.write(buffer, id, username, password);
    }
    /** **/
    @Override
    public int readFromBuffer(int pos, final Buffer buffer) {
        /** 1.从父类读取 **/
        pos = super.readFromBuffer(pos, buffer);
        /** 2.读取信息 **/
        final String[] reference = new String[3];
        pos = Buffalo.read(pos, buffer, reference);
        /** 3.从引用中读取数据 **/
        this.id = reference[0];
        this.username = reference[1];
        this.password = reference[2];
        return pos;
    }
```

这两个方法只有一个地方要注意，就是**顺序**，上边代码封装了读写Buffer的通用逻辑（因为代码重复太高），两个方法的实现如下：

```java
    public static void write(@NotNull final Buffer buffer,
                             final String... data) {
        // 遍历数据
        for (final String item : data) {
            if (StringKit.isNonNil(item)) {
                // 字节数据
                final byte[] bytes = item.getBytes(Resources.ENCODING);
                buffer.appendInt(bytes.length);
                buffer.appendBytes(bytes);
            }
        }
    }

    public static int read(final int start,
                           @NotNull final Buffer buffer,
                           @NotNull final String[] reference) {
        int pos = start;
        for (int idx = 0; idx < reference.length; idx++) {
            // 先读取长度信息
            final int len = buffer.getInt(pos);
            // 计算偏移量
            pos += 4;
            // 读取本身内容
            final byte[] bytes = buffer.getBytes(pos, pos + len);
            reference[idx] = new String(bytes, Resources.ENCODING);
            pos += len;
        }
        return pos;
    }
```

从上边代码可以看到，writeToBuffer的顺序是`id, username, password`，那么在readFromBuffer的过程中，其读取的顺序也是`id, username, password`，对于Buffer类型的细节这里不讲，主要是提醒一下大家，如果定义User，必定会重写这两个方法，个人的建议就是把能够标识用户身份的数据写入到Buffer中（如果这些数据需要执行权限检查，那么也需要写入进去）写入/读取的核心点就在于属性字段的顺序，一定要小心（这是序列化和反序列化的操作），顺序出现错误会导致序列化反序列化无法得到你想要的对象。

最后需要说明的是AbstractUser已经实现了默认的isAuthorized方法，该方法的源代码如下：

```java
    public User isAuthorized(String authority, Handler<AsyncResult<Boolean>> resultHandler) {
        if (this.cachedPermissions.contains(authority)) {
            resultHandler.handle(Future.succeededFuture(true));
        } else {
            this.doIsPermitted(authority, (res) -> {
                if (res.succeeded() && ((Boolean)res.result()).booleanValue()) {
                    this.cachedPermissions.add(authority);
                }

                resultHandler.handle(res);
            });
        }
        return this;
    }
```

该方法会调用doIsPermitted方法用来检查权限信息，而AbstractUser开启了权限缓存，那么我们在重写过程中，仅仅需要实现doIsPermitted方法就足够了，如果你重写了isAuthorized方法，那么就丢失了Vert.x自带的权限缓存功能，需要自己去实现它。

```java
    /**
     * 权限检查
     **/
    @Override
    public void doIsPermitted(final String permission, 
                              final Handler<AsyncResult<Boolean>> resultHandler) {
        // ....权限检查的核心逻辑
    }
```

### 3.2.创建Provider类

前边章节已经定义了我们需要使用的User接口实现类，接下来就定义另外一个需要实现的核心类Provider，在这里我们取名为：`BasicJunctor`，Provider中包含了核心认证方法，那么在实现该方法的过程中，如果进一步设计，可以在这里实现一些细粒度的操作，我们在这个类中实现了两个核心概念：

* Options：根据Basic相关的认证细节，提供一个配置选项，用于配置AuthProvider的基本信息；
* Authenticator：自定义接口，数据库主体认证访问接口——用于访问数据库的专用类，而且该类可使用一些设计模式替换其实现，为了和AuthProvider区分开，所以取名为`Authenticator`。

AuthProvider的职责是将输入的JsonObject类型认证信息在认证成功过后转换成User对象（3.1中定义的对象），先看看`BasicJunctor`的代码：

```java
@Guarded
public class BasicJunctor implements AuthProvider {

    /**
     * 安全配置项
     **/
    @NotNull
    private transient final Options options;
    /**
     * 底层访问对象
     **/
    private transient final Authenticator authcator;

    /** **/
    public BasicJunctor(@NotNull final Options options) {
        this.options = options;
        this.authcator = new BasicAuthenticator(options);
    }

    /**
     * Basic认证用的Provider
     */
    @Override
    public void authenticate(@NotNull final JsonObject authInfo, 
    				      @NotNull final Handler<AsyncResult<User>> handler) {
        try {
            /** 1.读取用户 **/
            final Record record = this.authcator
                    .authorize(authInfo.getJsonObject(SecureKeys.CREDENTIAL)
                    .getString(SecureKeys.TOKEN));
            /** 2.构造User **/
            final User user = this.buildUser(record);
            /** 3.正确响应 **/
            handler.handle(Future.<User>succeededFuture(user));
        } catch (final AbstractException ex) {
            handler.handle(Future.<User>failedFuture(ex));
        }
    }

    private BasicUser buildUser(final Record record) throws AbstractException {
        /** 1.转换 **/
        final Transferer transferer = Instance.singleton(ObjectTransferer.class);
        /** 2.转换数据 **/
        final JsonObject data = transferer.fromRecord(record);
        {
            /** 将数据填充到系统中 **/
            if (data.containsKey(Constants.PID)) {
                /** 3.1.ID转换 **/
                data.put(Keys.ID, data.getString(Constants.PID));
                data.remove(Constants.PID);
                /** 3.2.UserName转换 **/
                data.put(Keys.USERID, data.getString(options.readOpts().getString("secure.user.id")));
            }
        }
        /** 4.用户名，密码 **/
        return new BasicUser(this, data);
    }

}
```

针对上述代码段做个简单总结：

1. 访问数据库的代码主要是依赖自定义的Authenticator的实现，如上边代码中的：

	```java
	final Record record = this.authcator
                    .authorize(authInfo.getJsonObject(SecureKeys.CREDENTIAL)
                    .getString(SecureKeys.TOKEN));
	```

2. 私有方法`buildUser`会根据数据库中的返回结果构造对应的BasicUser类，也可以不存在，那么就需要在主代码中实现这种转换；
3. 上边的核心逻辑可以根据不同的项目来，比如Record, Authenticator, Options等，这些类的定义不属于Vert.x中的内容；
4. Vert.x中需要实现的仅仅只有方法：authenticate，这个方法的内部逻辑就是开发人员应该真正关注的点。

### 3.3.令牌类：定义Token

我们在设计权限管理系统时，将Authorization头的信息进行了封装，提供了Token接口，该接口的定义如下：

```java
public interface Token {
    /**
     * Token是否读取到，读取到Token证明Authorization头解析成功
     * @return
     */
    boolean obtained();

    /**
     * Token中是否存在Error，不成功的解析会返回自定义的异常基类
     * @return
     */
    AbstractException error();
    
    /**
     * 获取安全Schema信息，如BASIC、BEARER、DIGEST等
     * @return
     */
    String schema();
    /**
     * 读取Token字符串，Authorization: BASIC XXXX
     * 中的XXXX部分的值
     * @return
     */
    String token();
    /**
     * 返回域信息：realm（OAuth2中的scope）
     * @return
     */
    String realm();
}
```

系统中的默认实现为CommonToken，也可以替换该实现内容，Token的核心逻辑和上边提到的Vert.x中的parseCredentials一致：

```java
    public CommonToken(final HttpServerRequest request) {
        /** 1.检查Authorization头 **/
        final MultiMap map = request.headers();
        if (map.contains(HttpHeaders.AUTHORIZATION)) {
            final String auth = map.get(HttpHeaders.AUTHORIZATION);
            if (StringKit.isNil(auth)) {
                this.error = new _401AuthHeaderMissingException(getClass(), request.path());
            } else {
                /** 解析Auth头信息 **/
                this.parseHeader(auth);
            }
        } else {
            this.error = new _401AuthHeaderMissingException(getClass(), request.path());
        }
    }
    
    private void parseHeader(final String authHeader) {
        final String[] authArr = authHeader.split(Literal.SPACE);
        if (Constants.TWO == authArr.length) {
            final String schemaStr = authArr[Constants.ZERO].toUpperCase(Locale.getDefault());
            final SecureMode requested = SCHMAP.get(schemaStr);
            if (null == requested) {
                this.error = new _401SchemaWrongException(getClass(), schemaStr);
            } else {
                if (requested == Security.MODE) {
                    /** 设置对应的信息，已经解析过Schema了 **/
                    this.schema = schemaStr;
                    this.token = authArr[Constants.ONE];
                    this.realm = Resources.Security.REALM;
                } else {
                    this.error = new _401SchemaConflictException(getClass(), 
                    	requested.name() + ":" + schemaStr, Security.MODE.name());
                }
            }
        } else {
            this.error = new _401AuthHeaderInvalidException(getClass());
        }
    }
```

定义Token的目的是实现数据对象的传输，让所有的认证专用Provider接收到的信息是一个Token接口引用，而不是其实现对象，Token本身会转换成它主逻辑中需要使用的JsonObject，个人觉得这样的方式更加适合扩展，也不一定是最好的，目前的实现主要针对

```
Authorization: Basic <TOKEN> 
// Token = username:password的Base64编码值
Authorization: Bearer <TOKEN> 
// Token = accessToken:userId的Base64编码值，accessToken为OAuth2授权码流程交换的Token信息
```

### 3.4. 核心Handler类：SecureKeaper

该类实现了前文提到的`Handler<RoutingContext>`，核心逻辑位于`handle(RoutingContext)`方法中：

```java
		     /** 1.提取Token **/
            final Token token = new CommonToken(event.request());
            /** 2.Token是否获取到 **/
            if (token.obtained()) {
                /** 3.验证，将Token转换成标准的JsonObject对象，传给Provider **/
                final JsonObject unauthorized = Credential.get(event, token);
                FlowMonitor.info(LOGGER, "--> Prepare to authorize.", unauthorized);
                // 读取3.3.中定义的BasicJunctor类的实例
                final AuthProvider provider = getProvider();
                if (null != provider) {
                    provider.authenticate(unauthorized, result -> {
                        if (result.succeeded()) {
                            /** 4.认证成功 **/
                            final User user = result.result();
                            /** 5.填充用户数据 **/
                            final JsonObject credential = user.principal();
                            event.put(VertxKeys.Request.USER, user.principal());
                            // TODO: 执行授权流程
                            FlowMonitor.info(LOGGER, "200 Authorization success.", credential);
                            Feature.next(event);
                        } else {
                            /** 4.认证失败 **/
                            final Throwable error = result.cause();
                            /** 捕捉不了401就直接500返回 **/
                            Envelop stumer = 
                            	Envelop.failure(new _500InternalServerErrorException(getClass()));
                            /** 4.1.认证失败的信息 **/
                            FlowMonitor.warn(LOGGER, "401 Authoration failure. ");
                            if (error instanceof AbstractException) {
                                stumer = Envelop.failure(
                                		(AbstractException) error, 
                                		StatusCode.UNAUTHORIZED);
                            }
                            /** ERROR-ROUTE：错误路由处理 **/
                            Feature.route(event, stumer);
                        }
                    });
                } else {
                    FlowMonitor.warn(LOGGER, "401 Lookup Reference null.");
                    final Envelop stumer = 
                    	Envelop.failure(new _500InternalServerErrorException(getClass()));
                }
            } else {
                /** 3.从Header中读取Token失败 **/
                FlowMonitor.warn(LOGGER, "401 Token missing");
                final Envelop stumer = Envelop.failure(token.error(), StatusCode.UNAUTHORIZED);
                /** ERROR-ROUTE：错误路由 **/
                Feature.route(event, stumer);
            }
 
```


上述代码是Handler的主逻辑，读者可以只关心代码中的注释部分的流程，不用去关心太多细节，这个流程实现了完整的Handler处理认证授权主逻辑的代码流程。

## 4.总结

从上边的几个章节可以知道，若你了解了Vert.x中的整个结构（如2.4中的图），则可以开发任意自己觉得适合的自定义认证授权框架，但真正在项目过程中，官方文档的Demo复杂度和我们需要的复杂度不成正比，如前文中的复杂代码结构，就应用了很多细粒度设计。不论怎么设计，请求处理流程是一致的，简单整理一下，希望大家结合2、3章节的例子真正去理解Vert.x中认证和授权的做法。Vert.x中的auth-common是为了简化开发人员的工作，定义了标准的流程，让开发人员可以仅仅通过写AuthProvider和User的实现完成认证（最小改动）。

1. 接收请求，解析RoutingContext中的HttpServerRequest中的认证头：`Authorization: Schema Value`格式——在Vert.x中调用方法`parseCredentials`来完成这个工作，在我们自定义的框架中，这部分逻辑被封装到`CommonToken`中去实现了；
2. 第一步中的解析需要得的几个核心信息：
	1. 当前认证使用的Schema信息，如：`Basic、Digest、Bearer`，一般称为认证模式。
	2. 当前认证使用的Value信息，不同模式对应的值有所区别，解析算法会不同。
	3. 一旦不满足上述情况，需要返回401的错误代码（认证不通过）。
3. 解析成功过后，需要构造AuthProvider中所需的Credential对象（JsonObject类型），该对象会存储当前认证所需要的一些核心业务数据，比如Basic模式中包含了username和password的信息，我们在开发过程中针对OAuth2重新开发了一些内容，里边包含了资源ID、交换过后的AccessToken等，由于数据格式是JsonObject，这个地方的传输数据非常自由。
4. 【在这一步自定义数据库访问，或认证服务器的访问】<br/>调用AuthProvider的核心认证方法：authenticate，该方法会将传入的JsonObject转换成User实现对象（User对象可自定义）；
5. 【这一步实现自定义授权，包括页面域、数据域、字段域等】<br/>在认证通过后，调用User中的doIsPermitted方法执行授权过程，授权过程也可以重写AbstractUser的isAuthorized方法来完成整个授权流程的重写（前文提过，重写isAuthorized会导致权限缓存不可用（需要自定义）；
6. 认证成功过后，执行成功逻辑，一般成功的情况会调用RoutingContext的next方法，进入下一个Handler，否则就直接调用fail方法，并且返回对应的状态代码：401-认证不通过、403-授权失败。

在实现上述流程的过程中，如果不使用Vert.x提供的AuthProvider，你也可以用过程式的代码去实现自己的认证授权逻辑，只要按照上述流程做即可，最终只是一个设计问题。


