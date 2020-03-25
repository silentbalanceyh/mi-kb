# 那些年的伤痛

## 1. 个人的痛点

曾几何时，我们在代码里看多了类似`ArrayUtil.sort`等各种错综复杂的API，我们一直都在开发完美的工具类，它需要：

* 支持所有通用的功能
* 有很好的复用性
* 尽可能少的BUG

于是我们开始分类，也于是，我们有了类似：ArrayUtil，StringUtil等各种不同的工具类，形成了工具库，我从来都没有说过我反感这样的做法，可是从实战上看来，典型的`2/8`法则，很多时候我们在实际项目过程中，根本用不了这么多“大而全”的东西，一个项目从最初到收尾，我们都无法将所有的`Util`了熟于心。

当一个新的项目来临，我们又开始了漫长的`Util`的工作，发现一个新的工具类，又开始替换原始的，然后又开始重复，一旦当两个工具类本身出现了不兼容的时候，痛点就开始体现出来了：我们不愿意自己去封装，所有的东西都是开箱即用，箱子开多了，我们不得不去追溯某个API的源头，或者索性使用黏贴复制，代码的臭味开始泛滥，

## 2. 命名的追求

### 2.1. 关于缩写

很早的时候，我们在书本上学过，不要使用混乱的缩写命名，因为别人不知道那个缩写代表什么，但并没有说过不让我们使用缩写，实际上很多时候缩写应该有它自己的位置。我一直无法苟同于Spring中Repository部分的代码生成方法的做法，虽然完美表达了意思，但方法的长度实在是让我不能认同。比如：

```java
@Repository
public interface GroupRepository extends JpaRepository<Group, String> {

    @Query(value = "select distinct sec_group from Group sec_group left join fetch sec_group.users",
        countQuery = "select count(distinct sec_group) from Group sec_group")
    Page<Group> findAllWithEagerRelationships(Pageable pageable);

    @Query(value = "select distinct sec_group from Group sec_group left join fetch sec_group.users")
    List<Group> findAllWithEagerRelationships();

    @Query("select sec_group from Group sec_group left join fetch sec_group.users where sec_group.id =:id")
    Optional<Group> findOneWithEagerRelationships(@Param("id") String id);

}
```

所以在很长一段时间里我一直都在追求极致，甚至比较优雅而不失实用性的命名方式。

### 2.2. 关于约定

曾经，我们约定我们的数据访问层叫做`Dao`，那么它对应的实现叫做`DaoImpl`，我们业务逻辑层叫做`Service`，它们对应的实现叫做`ServiceImpl`，是的，这样的方式一直在延续——但是，这真的是一种喜闻乐见的好方式，还是说我们懒惰得不想去思考？在微服务流行的今天，究竟一个项目是“分层”还是“分模块”已经成为了一个喋喋不休的问题，而我在设计Zero的时候使用了很多“奇怪”的命名，但由于是“分模块”的，所以更多的时候只要心里明白，那么这个东西自然就成为了一种新的约定，渐渐的，原来我们不是习惯了约定，而是忘记了思考带来的福利。（Zero中的组件说明可以参考：[SPC-004 Zero内部组件说明](/specification/1-specification-zero/spc-004-zeronei-bu-zu-jian-shuo-ming.html)）

我看到过一个项目，有一个用户相关的服务叫做：UserService，一个接口就是20个左右的方法，他告诉我不能拆，实际上我们忘记了PMD中的一个规则：一个接口最好不要超过7个左右的方法，方法太多就可以考虑重构了。事实证明，重构工作是需要更大粒度的思考的、而且需要技术负责人拥有一定的视野、勇气和魄力，虽然我们心里清楚这个接口中本来就应该如此这般，但其实是有更加流畅的方式来解决这种问题的。

**于是在Zero的开发过程中，再也没有Dao/DaoImpl、再也没有Service/ServiceImpl，而是换成了另外的名字，职责更加清晰的名字。**

我一直都告诉我身边的人，在面对技术的时候需要学会的是谦卑，当很多人给我说代码写得如何时，我就会想到韩寒写的那篇文章——《[我也曾对那种力量一无所知](http://www.sohu.com/a/216401108_155921)》，所以一直都提醒自己，多思考，就像女蜗造人一样，造出更加精致而不失美感的东西。这里分享一份Zero中某个服务下的命名，并非法典，但是我们目前项目用得比较多的，大家可以参考：[SPC-001 Zero基本开发规范](/specification/1-specification-zero/spc-001-zeroji-ben-kai-fa-gui-fan.html)——我从来都不说这是一份完美的规范，但至少在实战上感觉，思路比以前更加清晰，当然一个东西的好用与否，仁者见仁智者见智。

### 2.3. 以职责为主的命名方式

曾经是一个“分层”的时代，有很长一段时间，我们都在考虑`MVC`，或者说直接使用分层架构，有展示层、控制层、业务逻辑层、数据访问层、领域模型层，这样的方式支撑了整个系统架构很长一段时间，于是那个时候我们系统中会有Factory、Manager类似的字眼，但是也引起了一个问题——有时候这些东西的职责不够单一。在最新的Redux框架从前端诞生时，官方的标配也是：Reducer、Types、Action在不同的层（实际上在前端会将不同的内容放到不同目录中），姑且不论这种方式的好坏，通过我们在开发中的时间证明，这种方式也许符合一个单项目的思路，但在“微架构”下，似乎有些问题。

* 这种分层的方式在“微架构“下，某一个模块的不同层组件位于同一的层中，其内聚性的表达分散得太厉害。
* 当开发人员去更改或开发某一个业务模块的组件时，必须先去定位层，然后在众多文件中找到自己要改动的文件，然后进行开发，而且每个开发人员在层中开发的内容是共享的，分离开发变得复杂。
* 如果要执行某个模块的重构，由于层中代码是共享的，定然会影响到别人的代码，使得模块和模块的隔离性变得很麻烦。

举个例子，如果有`User`和`Role`两个模块，现在有一批操作，传统的文件结构如下：

```shell
# 控制层
com/htl/controller/UserController.java
com/htl/controller/RoleController.java
# 业务逻辑层文件结构
com/htl/service/UserService.java
com/htl/service/RoleService.java
com/htl/service/UserServiceImpl.java
com/htl/service/RoleServiceImpl.java
# 数据访问层文件结构
com/htl/dao/UserDao.java
com/htl/dao/RoleDao.java
com/htl/dao/UserDaoImpl.java
com/htl/dao/RoleDaoImpl.java
# 领域模型
com/htl/domain/User.java
com/htl/domain/Role.java
```

在Zero中不推荐使用“分层”的架构，而是分模块的架构（以服务集合为中心），先参考下下边的结构：

> 为了和上边的代码结构对比，直接使用同样的文件名，暂时不以Zero中的文件名为中心。

```
# 用户模块
com/htl/micro/user/UserController.java
com/htl/micro/user/UserService.java
com/htl/micro/user/UserServiceImpl.java
com/htl/micro/user/UserDao.java
com/htl/micro/user/UserDaoImpl.java
# 角色模块
com/htl/micro/role/RoleController.java
com/htl/micro/role/RoleService.java
com/htl/micro/role/RoleServiceImpl.java
com/htl/micro/role/RoleDao.java
com/htl/micro/role/RoleDaoImpl.java
# 领域模型
com/htl/domain/User.java
com/htl/domain/Role.java
```

从一个完整的项目看来，第一种方式很不错，但是如果跨越的模块过多，那么在变更模块的内部执行逻辑时，会使得开发人员在`controller、service、dao`三个目录之间来回切换，而且如果只是改动Role中的内容，由于User和Role共享目录，有时候开发人员会不小心改动到User的东西，那么这样的方式显然在如今的“微架构“时代会变得不是那么实用，这也是Zero推荐第二种的原因。

那么领域模型为什么要独立出来呢？简单说，因为它是DTO，而且不包含逻辑，从另外一个意义上讲，领域模型一旦定义过后，出现改动的几率很小，很多时候不会去轻易改动一个系统的领域模型，而且即使是改动，遵循JavaBean的规范，也只是在属性和字段层面进行变更，而不会牵涉到主体逻辑的变更。再加上很多时候在书写程序逻辑时，领域模型是可共享的，如：当用户查询一个角色下所有的用户，可直接使用：`List<User> users = role.getUsers();`这种代码，这种情况下，角色的逻辑中会使用到`User`这个领域模型，所以领域模型独立出来也是有必要的。

最后说一点，在微架构下，只有两个完全独立的服务各自不共享领域模型，同一个服务中，使用第二种结构会更加清晰。

### 2.4. 以函数为中心

Java语言一直有一个弊端，就是”函数不是一等公民“，所以很多时候我们在代码里不得不写类似下边的代码：

```java
// 字符串比较
StringUtil.equal(a,b);
// 数组排序
ArrayUtil.sort(array);
```

这样的方式不是不好，而是很难让开发人员对工具有一个强化的记忆，这种模式实际还不是直接使用规范命名的全局函数，但是Java是不支持全局函数的，所以才有了Zero中的`Ut`和`Ux`两个核心工具类。

## 3. Zero中的改动

刚开始的开发人员可能不太会适应这样的方式，但久而久之，真正等你使用到时，慢慢就习惯了。

### 3.1. 概貌

这里对几个本章节会讲到的工具类进行说明：

| 类名（项目） | 说明 |
| :--- | :--- |
| Ut（vertx-zero） | Utility Tool的缩写，直接使用Java语言的工具类，全部统一使用方法前缀命名，通用工具类，spring-up也使用了Ut中的很多方法。 |
| Ux（vertx-zero） | Utility X的缩写，和Ut不同的是，Ux中的方法大部分位于业务逻辑层的专用方法，只能运行于Zero容器中，不可以在其他容器中使用，可理解为Zero容器中的专用工具类。 |
| Fn（vertx-zero） | Function的缩写，函数式编程的辅助方法，提供了很多更流畅的FP的处理。 |
| Ux（vertx-ui） | Utility X的缩写，Zero UI的专用前端JavaScript方法类，所有的工具都从Ux中沿用。 |
| Fn（ox-ui） | Function的缩写，由于Origin X运行在vertx-ui之中，所以可直接使用Ux，同样这里也提供了Origin X的专用工具类，使用Fn来处理（这个也是Zero UI规范定义的）。 |

### 3.2. 示例代码

使用了上述方法后，代码本身就简洁很多了，比如下边的方法：

```java
package com.htl.micro.order;

import com.htl.cv.Addr;
import io.vertx.core.Future;
import io.vertx.core.json.JsonObject;
import io.vertx.up.aiki.Ux;
import io.vertx.up.annotations.Address;
import io.vertx.up.annotations.Queue;
import io.vertx.up.atom.Envelop;

import javax.inject.Inject;

@Queue
public class TicketWorker {

    @Inject
    private transient TicketStub ticketStub;

    @Address(Addr.ORDER_PUT)
    public Future<JsonObject> edit(final Envelop envelop) {
        // 直接调用Ux方法，从Agent中传入的参数里读取第一个参数，由于定义的是String类型的，直接使用getString来读取。
        final String id = Ux.getString(envelop);
        // 直接调用Ux方法，从Agent中传入的参数里读取第二个参数，JsonObject类型。
        final JsonObject data = Ux.getJson1(envelop);
        return this.ticketStub.update(id, data);
    }
}
```

上述代码这样看起来可能比较费解，看看下边关于这个地址（`Addr.ORDER_PUT`）下的RESTful定义就明白了。

```java
    /**
     * 按ID更新单张订单数据
     *
     * @param id   订单ID
     * @param data 传入需要更新的订单数据
     * @return 返回读取的订单数据
     */
    @Path("/order/{id}")
    @PUT
    @Address(Addr.ORDER_PUT)
    String put(@PathParam("id") String id, @BodyParam JsonObject data);
```

> 实际上，Ux.getString读取的就是orderId参数，而Ux.getJson1读取的就是JsonObject类型的Body参数，对开发人员而言，它不需要知道Agent和Worker是怎么使用Event Bus的，同样不需要知道Sender和Consumer是如何通信的，直接定义了接口后，在对应的方法内调用Ux的API就可以取到参数了，而Agent和Worker中的绑定关系就是依靠Addr.ORDER\_PUT这个地址来绑定的。

## 4. 总结

实际上，Zero只是站在开发人员的视角，从”简易开发“去考虑Utility X的设计，主要目的是帮助开发人员思考，有时候更加傻瓜式地直接使用Zero来完成“需求层面”的任务。

