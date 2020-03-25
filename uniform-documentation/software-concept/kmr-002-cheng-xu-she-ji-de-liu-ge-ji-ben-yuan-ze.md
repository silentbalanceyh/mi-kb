## 程序设计的六个基本原则

* 单一职责原则
* 里氏代换原则
* 依赖倒置原则
* 接口隔离原则
* 迪米特法则
* 开闭原则

### 1.单一职责原则
全称：Single Responsibility Principle，又称为SRP原则。<br/>
<font style="color:blue">*There should never be more than one reason for a class to change.*</font><br/>
__在类（Class）、接口（Interface）、方法（Method）的设计中，一个类、接口或方法在某种意义上只做一件事，严格地将程序中的“职责”区分开来。__

<font style="color:red">【注意】单一职责原则提出了程序编写的一个标准，用“职责”或“变化原因”来衡量接口或类是否设计得优良，但“职责”和“变化原因”本身都是无法度量得，因项目和环境而异。单一职责的优点：</font>

* 类的复杂性降低，实现什么职责都有了清晰明确的定义；
* 可读性提高，因为复杂性降低了所以可读性自然提高了；
* 可维护性提高，可读性提高导致维护成本更低；
* 变更引起的风险更低，变更在软件设计里面是必不可少的，如果接口的单一职责做得好，一个接口的修改只会对实现类有影响，对其他的接口没有任何影响，这对系统的扩展性、维护性都有非常大的帮助；

### 2.里氏代换原则
全称：Liskov Substitution Principle，又称为LSP原则。

* <font style="color:blue">*If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T.*</font>
* <font style="color:blue">*Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.*</font>

第一条理解起来比较困难，但两条的意思是一样的，简单讲：__只要父类出现的地方子类就可以出现，而且替换成子类也不会产生任何错误或异常，使用者可能根本就不需要知道是父类还是子类；但是反之就不行，有子类出现的地方，父类未必就能适应。__

### 2.1.关于继承
*优点*

* 代码共享，减少创建类的工作量，每个子类都拥有父类的方法和属性；
* 提高代码的重用性；
* 子类可以形似父类，但又异于父类，“龙生龙、凤生凤、老鼠生来会打洞”就是说子拥有父的“种”，“世界上没有两片完全相同的叶子”是指明子与父的不同；
* 提高代码的可扩展性，实现父类的方法就可以“为所欲为”了，君不见很多开源框架的扩展接口都是通过继承父类来完成的；
* 提高产品或项目的开放性；

*缺点*

* 继承是侵入性的，只要继承，就必须拥有父类的所有属性和方法；
* 降低代码的灵活性。子类必须拥有父类的属性和方法，让子类自由的世界中多了些约束；
* 增强了耦合性。当父类的常量、变量和方法被修改时，需要考虑子类的修改，而且在缺乏规范的环境下，这种修改可能带来非常糟糕的结果——打断的代码需要重构；

### 2.2.里氏代换原则的四层含义

1. __子类必须完全实现父类的方法__<br/>
<font style="color:red">【注意】在类中调用其他类时务必要使用父类或接口，如果不能使用父类或者接口，那么这个类的设计违背了LSP原则。</font><br/>
<font style="color:red">【注意】如果子类不能完整地实现父类的方法，或者父类的某些方法在子类中已经发生“畸变”，则建议断开父子继承关系，采用依赖、聚集、组合等关系代替继承。</font>
2. __子类可以有自己的个性__
3. __覆盖或实现父类的方法时输入的参数可以被放大__<br/>
子类中方法的前置条件必须与超类中北覆写的方法的前置条件相同或者更加宽松
4. __覆盖或实现父类的方法时输出的结果可能被缩小__<br/>
父类的一个方法返回值是一个T，子类的相同方法（重载或重写）的返回值为S，那么里氏代换原则就要求S必须小于等于T，也就是说S和T要么是同一个类型，要么是T的子类。

#### *前置条件&后置条件*

Design By Contract（契约设计）

* 前置条件：你要让我执行，就必须满足我的条件，主要针对输入；
* 后置条件：我执行完了需要反馈，标准是什么，主要针对输出；

### 3.依赖倒置原则
全称：Dependence Inversion Principle，又称为DIP

<font style="color:blue">*High level modules should not depend upon low level modules. Both should depend upons abstractions. Abstractions should not depend upon details. Details should depend upon abstractions.*</font>

三层含义

* 高层模块不应该依赖低层模块，两者都应该依赖其抽象；
* 抽象不应该依赖细节；
* 细节应该依赖抽象；

在Java语言中，依赖倒置的提现为：

* 模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或抽象产生的；
* 接口或抽象类不依赖实现类；
* 实现类依赖接口或抽象类；

精简定义：面向接口编程——OOD（Object-Oriented Design，面向对象设计）的精髓之一，即基于契约的编程思维。<br/>

<font style="color:red">【注意】设计是否具备稳定性，只要适当地“松松土”，观察“设计的蓝图”是否还可以茁壮地成长就可以得出结论，稳定性较高的设计，在周围环境频繁变化的时候，依然可以做到“我自依然不动”。</font>

在设计以来的过程中，尽可能地使用接口、抽象类的依赖关系而不使用实现类的依赖关系，通过这种方式来实现松耦合的系统，虽然在实际应用中实例化的时候会使用类似`new ClassName()`的方式来实例化某个类，但真正实现过程可屏蔽掉这样的代码（通过Reflection）。

<font style="color:red">【注意】在Java语言中，只要定义变量就必然要有类型，一个变量可以有两种类型：表面类型（引用类型）和实际类型（对象类型），表面类型是在定义的时候赋予的类型，实际类型则是对象的类型。</font>

	IDriver driver = new Driver()

#### *依赖注入的三种方式*

* 使用构造函数传递依赖对象
* Setter方法传递依赖对象
* 接口传递依赖对象


#### *最佳实践*

* 每个类尽量都有接口 or 抽象类，或者抽象类和接口两者都具备；
* 变量的表面类型尽量是接口或抽象类
* 任何类都不应该从具体类派生而来
* 尽量不要覆写基类的方法
* 结合里氏代换原则使用

<font style="color:red">依赖倒置原则是6个设计原则中最难以实现的原则！</font>

### 4.接口隔离原则

接口主要分为两种：

1. 实例接口（Object Interface），比如Java中实现一个类，然后用new关键字产生一个实例，它是对一个类型的事物的描述，这是一种接口。比如Person类是定义的具体类，使用的时候常常用`Person person = new Person()`，这个时候Person类本身属于一个接口，Java类也属于一个接口。
2. 类接口（Class Interface），Java中经常使用interface关键字定义的接口。

接口隔离原则的两种定义：

* <font style="color:blue">Clients should not be forced to depend upon interfaces that they don't use.</font>（客户端不应该依赖它不需要的接口）。
* <font style="color:blue">The dependency of one class to another one should depend on the smallest possible interface.</font>（类间的依赖关系应该建立在最小的接口上）。

接口中的方法如果过多会导致“封装过度”的情况发生，而且接口隔离原则和单一职责原则是相反的设计，有可能接口隔离会导致单一职责被破坏，所以接口隔离原则必须满足一个基本原则：接口的纯洁性。

#### *保证接口的纯洁性*

* 接口要尽量小：在接口隔离原则和单一职责原则发生冲突时，根据接口隔离原则拆分接口时，首先必须满足单一职责原则。
* 接口要高内聚
* 定制服务：定义服务就是单独为一个个体提供优良的服务
* 接口的设计是有限度的：接口的设计粒度越小，系统越灵活，但是灵活的同时有可能带来接口的复杂化，开发难度增加，而维护性降低。

#### *最佳实践*

* 一个接口只服务于一个子模块或业务逻辑；
* 通过业务逻辑压缩接口中的public方法，接口时常去回顾，尽量让接口达到“满身筋骨肉”，而不是“肥嘟嘟”的一大堆方法；
* 已经被污染了的接口，尽量去修改，若变更的风险较大，则采用适配器模式进行转化处理；
* 了解环境，拒绝盲从：每个项目有特定的环境因素，不能直接拷贝过来就使用，需要从自身的项目设计出发考虑接口的拆分、合并以及设计。

<font style="color:red">接口粒度太小，导致接口数量剧增，开发人员会呛死在接口中；接口粒度太大，灵活性降低、无法提供定制服务，给整体项目代理无法预测的风险。</font>

### 5.迪米特法则
全称：Law Of Demeter，又称为LoD，又为最少知识原则（Least Knowledge Principle，LKP）

<font style="color:blue">Only talk to your immediate friends.</font>（只和直接的朋友通信。）

1. 只和朋友交流：每个对象都必然会和其他对象有耦合关系，两个对象之间的耦合就成为了朋友关系；<br/>
<font style="color:red">【注意】一个类只和朋友交流，不于陌生类交流，不要出现`getA().getB().getC().getD()`这种情况（在一种极端的情况下允许出现这种访问，及每一个点号后边的返回值类型都相同），类与类之间的关系是建立在类间的，而不是方法间，因此一个方法尽量不引入一个类中不存在的对象，当然，JDK本身提供的API中某些类除外。</font>
2. 朋友之间也是有距离的：一个类公开的public的属性和方法越多，修改涉及面也就越大，变更引起的风险扩散也就越大，因此，为了保持朋友类间的距离，在设计时要反复衡量：是否可以在减少public方法和属性，是否可修改为private，package-private（Java中的Package Only），protected等访问权限，是否可以加上final关键字。<br/>
<font style="color:red">【注意】迪米特法则要求类“羞涩”一点，尽量不要对外公布太多的public方法和非静态的public变量，尽可能多使用private，package-private，protected等。</font>
3. 是自己的就是自己的：如果一个方法放在奔雷中，既不增加类间的关系，也对本类不产生负面影响，那就放置在本类中。
4. 谨慎使用Serializable

#### *最佳实践*
迪米特法则的核心观念就是类间解耦，弱耦合，只有弱耦合了以后，软件的复用率才可以提高。如果一个类跳转两次以上才能访问到另一个类，就需要想办法进行重构了。

### 6.开闭原则
开闭原则是Java世界里最基础的设计原则，也是其他五个原则的灵魂。

<font style="color:blue">*Software entities like classes, modules and functions should be open of extension but closed for modifications.*</font>（一个软件实体如类、模块和函数应该对扩展开放，对修改关闭）

软件实体的定义：

* 项目或软件产品中按照一定的逻辑规则划分的模块；
* 抽象和类；
* 方法；

典型的三种修改一个现存系统的方法：

1. 修改接口定义
2. 修改实现类
3. 通过扩展，在现有的类基础之上扩展子类

<font style="color:red">【注意】开闭原则对扩展开放，对修改关闭，并不意味着不做任何修改，低层模块的变更，必须要有高层模块进行耦合，否则就是一个孤立无意义的代码片段。</font>

一般变化分为三种类型：

1. 逻辑变化
2. 子模块变化
3. 可见视图变化

实现开闭原则的方法

* 抽象约束
* 元数据（metadata）控制模块行为：尽可能地使用元数据的行为，减少重复开发
* 制定项目章程
* 封装变化：<br/>1）将相同的变化封装到一个接口或抽象类中；<br/>2）将不同的变化封装到不同的接口或抽象类中，不应该有两个不同的变化出现在同一个抽象类或接口中