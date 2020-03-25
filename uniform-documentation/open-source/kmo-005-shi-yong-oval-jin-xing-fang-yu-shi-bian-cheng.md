# 「译」使用AOP & OVal进行防御式编程

## 1 基本概念
什么是OVal框架，OVal实际上是一个可扩展的对象验证框架，它可以针对任何Java类进行验证，它的功能如下：

* 根据需求验证对象数据的合法性；
* 针对类的get方法和成员变量进行适当的约束；
* 基于EJB3和JPA的Annotation验证对象约束（实体对象Entity）；
* 可通过Annotation，POJOs和XML文件配置约束；
* 可使用脚本语言类似Groovy，BeanShell，JavaScript书写表达式用来执行简单的约束逻辑；
* 很容易创建自定义约束
* 使用它很容易开发可配置的新约束

如果使用AspectJ的契约式编程，还可以实现下边的功能：

* （precondition）在构造函数调用的时，提供约束自动检查构造函数参数；
* （precondition）在方法调用时，提供约束自动检查方法参数；
* （precondition）在方法调用之前获取关键对象的状态；
* （invariants）在对象创建过后强制对象验证；
* （invariants）当对象在方法调用之前/之后强制进行对象验证；
* （postcondition）在方法执行过后对方法的返回值进行约束；
* （postcondition）在方法调用之后获取对象状态；

## 2 依赖项

* 如果在程序中使用了AspectJ的功能则依赖【AspectJ】
* 如果使用了`JEXL`表达式语法则依赖【Apache Common JEXL】
* 如果使用了`BeanShell`表达式语法则依赖【BeanShell】
* 如果使用了`Groovy`表达式语法则依赖【Groovy】
* 如果使用`Ruby`表达式语法则依赖【JRuby】
* 如果使用了`JavaScript`表达式语法则依赖【Mozilla Rhino】
* 如果使用了`MVEL`表达式语法则依赖【MVEL】
* 如果使用了`OGNL`表达式语法则需要依赖【OGNL】
* 如果使用`XML`来配置OVal则需要依赖【XStream】
* 如果需要内部使用`GNU`的高性能集合，则需要依赖【GNU Trove】
* 如果需要使用`Javolution's`的高性能集合，则需要依赖【Javolution】
* 如果在约束目标定义的时候使用了`JXPath`表达式语法则需要依赖【XPath】
* 如果要运行Test Case测试则依赖【JUnit】

## 3 用法

### 3.1 定义类的成员约束

类中的定义，使用约束包：`net.sf.oval.constraint`

```java
	public class BusinessObject {

		@NotNull
		@NotEmpty
		@Length(max=32)
		private String name;
		...
	}
```
验证约束的用法，使用：`net.sf.oval.Validator`中的方法`public validate(Object validatedObject)`


```java

	Validator validator = new Validator();
	BusinessObject bo = new BusinessObject(); // name is null
	// collect the constraint violations
	List<ConstraintViolation> violations = validator.validate(bo);
	if(violations.size()>0)
	{
		LOG.severe("Object " + bo + " is invalid.");
		throw new BussinessException(violations);
	}
```



### 3.2 定义一个类中的`getter`方法返回值约束
针对`getter`方法，需要保证这个方法不会修改对象的状态，一般可使用`Annotation`——`@net.sf.oval.configuration.annotation.IsInvariant`
类中的定义：

```java
	public class BusinessObject
	{
		private String name = null;
		@IsInvariant
		@NotNull
		@Length(max = 4)
		public String getName()
		{
			return name;
		}
		...
	}
```
验证约束的使用代码：

```java
	Validator validator = new Validator();
	BusinessObject bo = new BusinessObject("blabla");
	// collect the constraint violations
	List<ConstraintViolation> violations = validator.validate(bo);
	if(violations.size()>0)
	{
		LOG.severe("Object " + bo + " is invalid.");
		throw new BussinessException(violations);
	}
```
### 3.3 使用表达式语法定制条件约束
如果需要在约束的字段或者方法中使用代码逻辑，则需要使用`@net.sf.oval.constraint.Assert`采用表达式语法：

```java
	public class BusinessObject
	{
		@NotNull
		public String deliveryAddress;
		@NotNull
		public String invoiceAddress;
		// mailingAddress must either be the delivery address or the invoice address
		@Assert(expr = "_value ==_this.deliveryAddress || _value == _this.invoiceAddress", lang = "groovy")
		public String mailingAddress;
	}
```
在检查这个约束的时候，__expr__属性中的表达式会被执行，如果返回__true__则约束满足，OVal中提供了两个特殊的变量：

* __\_value__：用约束验证对应的值（字段值或`getter`的返回值）
* __\_this__：引用被验证的Object对象

__lang__属性表示使用的表达式种类，OVal支持的种类如下：

* `bsh`或`beanshell`：BeanShell表达式；
* `groovy`：Groovy表达式；
* `jexl`：JEXL表达式；
* `js`或`javascript`：JavaScript表达式（Mozilla Rhino规范）
* `mvel`：MVEL表达式；
* `ognl`：OGNL表达式；
* `ruby`或`jruby`：Ruby表达式（JRuby实现）

如果要添加其他的表达式语言则使用：`Validator.addExpressionLanguage(String, ExpressionLanguage)`
### 3.4 表达式中定义约束的激活条件
表达式不仅仅可以定义约束，并且可以通过：__when__属性，这种情况下可以设置在某种条件下启用该约束

```java
	public class BusinessObject
	{
		private String fieldA;
		@NotNull(when = "groovy:_this.fieldA != null")
		private String fieldB;
	}
```
### 3.5 定义嵌套属性约束
除了__when__属性以外，还可以使用__target__属性设置对象中的嵌套属性的约束，例如：

```java
	public class BusinessObject
	{
		@AssertValid
		@NotNull(target="homeAddress.street")
		private Customer customer;
	}
```
上边的代码表示约束的对象是：`customer.homeAddress.street`，如果ClassPath中包含了JXPath的库，还可以使用XPath的语法来指定嵌套属性：

```java
	public class BusinessObject
	{
		@AssertValid
		@NotNull(target="jxpath:addresses[0]/street")
		private Customer customer;
	}
```
### 3.6 递归验证
看看下边的代码：

```java
	public class BusinessObject
	{
		@NotNull
		@AssertValid
		private Address address;
	}
```
上边的`@AssertValid`表示引用的`Address`类型的对象中的所有的验证约束都必须满足才行。
### 3.7 EJB3的验证
OVal的机制是可支持定制的，使用`net.sf.oval.configuration.Configurer`接口可基于XML Schemas写自己的约束逻辑。官方实现了一个基于EJB3 JPA的配置器：`net.sf.oval.configuration.annotation.JPAAnnotationsConfigurer`，里边包含了下边的约束映射：

* __@javax.persistence.Basic(optional=false) => @net.sf.oval.constraint.NotNull__
* __@javax.persistence.OneToOne(optional=false) => @net.sf.oval.constraint.NotNull__
* __@javax.persistence.OneToOne => @net.sf.oval.constraint.AssertValid__
* __@javax.persistence.OneToMany => @net.sf.oval.constraint.AssertValid__
* __@javax.persistence.ManyToOne(optional=false) => @net.sf.oval.constraint.NotNull__
* __@javax.persistence.ManyToOne => @net.sf.oval.constraint.AssertValid__
* __@javax.persistence.Column(nullable=false) => @net.sf.oval.constraint.NotNull__ <br/>(only applied for fields not annotated with @javax.persistence.GeneratedValue or @javax.persistence.Version)
* __@javax.persistence.Column(length=5) => @net.sf.oval.constraint.Length__

定义：

```java
	@Entity
	public class MyEntity
	{
		@Basic(optional = false)
		@Column(length = 4)
		public String id;
		@Column(nullable = false)
		public String descr;
		@ManyToOne(optional = false)
		public MyEntity parent;
	}
```
上边没有使用OVal的Annotation，但OVal中的配置器可直接验证EJB3中的Annotation：

```java
	// configure OVal to interprete OVal constraint annotations as well as EJB3 JPA annotations
	Validator validator = new Validator(new AnnotationsConfigurer(), new JPAAnnotationsConfigurer());
	MyEntity entity = new MyEntity();
	entity.id = "12345"; // violation - the max length is 4
	entity.descr = null; // violation - cannot be null
	entity.parent = null; // violation - cannot be null
	// collect the constraint violations
	List<ConstraintViolation> violations = validator.validate(entity);
```
### 3.8 Interpreting Bean Validation（JSR303）依赖注入
JSR303的实现方式和JPA的实现方式一致:

配置器：`net.sf.oval.configuration.annotation.BeanValidationAnnotationsConfigurer`

映射关系如下：

* __@javax.validation.constraints.AssertFalse => @net.sf.oval.constraint.AssertFalse__
* __@javax.validation.constraints.AssertTrue => @net.sf.oval.constraint.AssertTrue__
* __@javax.validation.constraints.DecimalMax => @net.sf.oval.constraint.Max__
* __@javax.validation.constraints.DecimalMin => @net.sf.oval.constraint.Min__
* __@javax.validation.constraints.Digits => @net.sf.oval.constraint.Digits__
* __@javax.validation.constraints.Future => @net.sf.oval.constraint.Future__
* __@javax.validation.constraints.Max => @net.sf.oval.constraint.Max__
* __@javax.validation.constraints.Min => @net.sf.oval.constraint.Min__
* __@javax.validation.constraints.NotNull => @net.sf.oval.constraint.NotNull__
* __@javax.validation.constraints.Null => @net.sf.oval.constraint.Null__
* __@javax.validation.constraints.Past => @net.sf.oval.constraint.Past__
* __@javax.validation.constraints.Pattern => @net.sf.oval.constraint.Pattern__
* __@javax.validation.constraints.Size => @net.sf.oval.constraint.Size__
* __@javax.validation.constraints.Valid => @net.sf.oval.constraint.AssertValid__

定义：

```java
	public class MyEntity
	{
		@javax.validation.constraints.NotNull
		@javax.validation.constraints.Size(max = 4)
		public String id;
		@javax.validation.constraints.NotNull
		public String descr;
		@javax.validation.constraints.NotNull
		public MyEntity parent;
	}
```
使用流程：

```java
	// configure OVal to interprete OVal constraint annotations as well as built-in JSR303 annotations
	Validator validator = new Validator(new AnnotationsConfigurer(), new BeanValidationAnnotationsConfigurer());
	MyEntity entity = new MyEntity();
	entity.id = "12345"; // violation - the max length is 4
	entity.descr = null; // violation - cannot be null
	entity.parent = null; // violation - cannot be null
	// collect the constraint violations
	List<ConstraintViolation> violations = validator.validate(entity);
```

## 4 使用OVal基于契约的编程方法
如果结合AspectJ OVal则可以实现基于契约的编程：

* （precondition）强制构造函数、方法的参数在满足定义的约束范围内执行
* （precondition/invariant）如果Object的状态是核心状态强制方法调用
* （postcondition）在满足了返回值的约束的时候强制返回对应的值
* （postcondition/invariant）如果方法执行过后，其对象的状态必须是核心状态对象

在Eclipse中扩展OVal GuardAspect的方法：

1. 点击项目右键：__Configure -> Convert To AspectJ Project__【依赖AJDT插件】;
2. 将**net.sf.oval_x.x.jar**放到类路径中；
3. 创建一个新的Aspect：**File -> New -> Aspect**，这个切面类从__net.sf.oval.guard.GuardAspect__中继承过来，如：
	```java
		public aspect DefaultGuardAspect extends GuardAspect{
			public DefaultGuardAspect(){
				super();
			}
		}
	```
4. 在创建了上边的类过后，就可以使用自定义约束了，直接在Class中使用`@net.sf.oval.guard.Guarded`的标记都会被识别并且被Aspect的切面代码执行。

这种方式主要用于自定义的约束逻辑放到AOP切面的时候使用，实际上直接使用`@net.sf.oval.guard.Guarded`也可以完成，但实现的都是默认逻辑！

### 4.1 前置条件【Preconditions】

#### （1）构造函数约束
构造函数的参数会在被调用的时候自动检查，如果不满足约束条件的话会产生异常：`net.sf.oval.exception.ConstraintsViolatedException`；

```java
	@Guarded
	public class BusinessObject
	{
		public BusinessObject(@NotNull String name)
		{
			this.name = name;
		}
		...
	}
```
下边的代码段会抛出异常：

```java
	// throws a ConstraintsViolatedException because parameter name is null
	BusinessObject bo = new  BusinessObject(null);
```
#### （2）方法参数约束
方法参数抛出的异常和构造函数的异常一样：`net.sf.oval.exception.ConstraintsViolatedException`；

```java
	@Guarded
	public class BusinessObject
	{
		public void setName(@NotNull String name)
		{
			this.name = name;
		}
		...
	}
```
下边的代码会抛出异常：

```java
	BusinessObject bo = new BusinessObject();
	bo.setName(null); // throws a ConstraintsViolatedException because parameter name is null
```
#### （3）统一约束
在一个类中，若多个属性的约束性质一致的话，可将这种字段约束性质引用到对应的`setter`方法中统一约束规则，使用Annotation：`@net.sf.oval.constraint.AssertFieldConstraints`。

* 如果不使用`@net.sf.oval.constraint.AssertFieldConstraints`的`value`属性，则标记的同名字段会被加上这个约束；
* 如果使用`@net.sf.oval.constraint.AssertFieldConstraints`的`value`属性，则`value`这个值的字段属性会被加上这个约束；

使用：

```java
	@Guarded
	public class BusinessObject
	{
		@NotNull
		@NotEmpty
		@Length(max=10)
		private String name;
		public void setName(@AssertFieldConstraints String name)
		{
			this.name = name;
		}
		public void setAlternativeName(@AssertFieldConstraints("name") String altName)
		{
			this.alternativeName = altName;
		}
		...
	}
```
下边的代码会报错：

```java
	BusinessObject bo = new BusinessObject();
	bo.setName(""); // throws a ConstraintsViolatedException because parameter is empty
	bo.setAlternativeName(null); // throws a ConstraintsViolatedException because parameter is null
```
如果要在这个类中将所有的字段的`setter`方法都统一约束，可以使用`@Guarded`标记的`applyFieldConstraintsToSetters`设置成__true__，这种方式比较适合`setter`方法特别多的时候：

```java
	@Guarded(applyFieldConstraintsToSetters=true)
	public class BusinessObject
	{
		@NotNull
		@NotEmpty
		@Length(max=10)
		private String name;
		public void setName(String name)
		{
			this.name = name;
		}
		...
	}
```
这个标记包含了两个选项：

* __applyFieldConstraintsToSetters__：针对所有的`setter`方法参数；
* __applyFieldConstraintsToConstructors__：针对类中所有的构造函数的参数；

#### （4）使用表达式

和上边的的用法一样，使用表达式主要使用：`@net.sf.constraint.Assert`标记在字段上，同样可以使用`@net.sf.constraint.Pre`用于方法的前置条件：

```java
	@Guarded
	public class Transaction
	{
		private BigDecimal amount;
		// ensure that amount is not null, ensure that value2add is greater than amount
		@Pre(expr = "_this.amount!=null && amount2add > _this.amount", lang = "groovy")
		public void increase(BigDecimal amount2add)
		{
			amount = amount.add(amount2add);
		}
	}
```
在方法调用之前：__expr__属性中的表达式会被执行，返回__true__的情况下约束满足则执行该方法：

* __\_args[]__：表示方法的参数表
* __\_this__：表示当前对象的引用
* 额外的变量如果参数的名称是匹配的也属于合法变量

__lang__属性表示使用的表达式种类，OVal支持的种类如下：

* `bsh`或`beanshell`：BeanShell表达式；
* `groovy`：Groovy表达式；
* `jexl`：JEXL表达式；
* `js`或`javascript`：JavaScript表达式（Mozilla Rhino规范）
* `mvel`：MVEL表达式；
* `ognl`：OGNL表达式；
* `ruby`或`jruby`：Ruby表达式（JRuby实现）

#### （5）禁用前置条件
OVal中提供了一种方法禁用前置条件检查：`Guard.setPreConditionsEnabled(boolean)`

```java
	MyAspect.aspectOf().getGuard().setPreConditionsEnabled(false);
```

### 4.2 后置条件【Postconditions】

#### （1）方法返回值约束
后置条件针对方法的返回值进行约束，所以不可用于返回值为`void`的方法，抛出的异常信息和上边一样：`net.sf.oval.exception.ConstraintsViolatedException`；<br/>
<font style="color:red">*：注意这个约束虽然会抛出异常，但如果方法改动了数据，那么所有的改变都需要手动回滚。</font>
如果一个带参数的返回值不为`void`的方法标记了`@IsInvariant`标记，则它会在`Validator.validate(Object)`的被验证。

```java
	@Guarded
	public class BusinessObject
	{
		private String name = null;
		@IsInvariant
		@NotNull
		@Length(max = 4)
		public String getName()
		{
			return name;
		}
		@NotNull
		@Length(max = 4)
		public String getNameOrDefault(String default)
		{
			return name == null ? default : name;
		}
		...
	}
```
使用的时候用下边的方式：

```java
	BusinessObject bo = new BusinessObject();
	// throws a ConstraintsViolatedException because field name is null
	bo.getName();
	// throws a ConstraintsViolatedException because the field "name" is null and 
	// therefore the default parameter will be returned which has a length &lt; 4 characters
	bo.getNameOrDefault("abc");
	Validator validator = new Validator();
	// returns one ConstraintViolation because the getter method getName() is 
	// declared as invariant and returns an invalid value (null)
	List<ConstraintViolation> violations = validator.validate(bo);
```
#### （2）后置条件使用表达式
后置条件使用表达式和前边的`@net.sf.guard.Pre`差不多，使用的Annotation是：`@net.sf.guard.Post`；

```java
	@Guarded
	public class Transaction
	{
		private BigDecimal amount;
		// ensure that amount after calling the method is greater than it was before
		@Post(expr = "_this.amount>_old", old = "_this.amount", lang = "groovy")
		public void increase(BigDecimal  amount2add)
		{
			amount = amount.add(amount2add);
		}
	}
```
在方法调用之后：__expr__属性中的表达式会被执行，返回__true__的情况下约束满足则执行该方法：

* __\_args[]__：表示方法的参数表
* __\_this__：表示当前对象的引用
* __\_returns__：表示当前方法的返回值
* __\_old__：表示参数描述的参数的传入方法时的值，__old__值
* 额外的变量如果参数的名称是匹配的也属于合法变量

__lang__属性表示使用的表达式种类，OVal支持的种类如下：

* `bsh`或`beanshell`：BeanShell表达式；
* `groovy`：Groovy表达式；
* `jexl`：JEXL表达式；
* `js`或`javascript`：JavaScript表达式（Mozilla Rhino规范）
* `mvel`：MVEL表达式；
* `ognl`：OGNL表达式；
* `ruby`或`jruby`：Ruby表达式（JRuby实现）

#### （3）禁用后置表达式
OVal中提供了一种方法禁用后置条件检查：`Guard.setPostConditionsEnabled(boolean)`

```java
	MyAspect.aspectOf().getGuard().setPostConditionsEnabled(false);
```

### 4.3 不变量【Invariants】

#### （1）禁用不变量自动检查
默认情况下OVal会在任何标记了@Guarded的类中的构造函数执行之后，以及非private方法的执行前后启用不变量检查，如果有必要则可以全局禁用这种自动检查：

* __Guard.setInvariantsEnabled(boolean)__：禁用所有的自动检查
* __Guard.setInvariantsEnabled(Class<?>, boolean)__：针对某个Class禁用自动检查

示例代码如下：

```java
	MyAspect.aspectOf().getGuard().setInvariantsEnabled(false);
```

#### （2）强制执行方法调用前检查
如果禁用了Global的自动检查过后，则可以使用`@net.sf.oval.guard.PreValidateThis`来进行方法前检查。

定义代码：

```java
	@Guarded
	public class BusinessObject
	{
		@NotNull
		private String name = null;
		@PreValidateThis
		public void save()
		{
			// do something fancy
		}
		...
	}
```

使用例子：

```java
	// create a new business object and leaving the field name null 
	BusinessObject bo = new  BusinessObject();
	// the save() method will throw a ConstraintsViolatedException because field name is null
	bo.save();
```

#### （3）强制执行构造函数执行后检查
执行构造方法后检查使用：`@net.sf.oval.guard.PostValidateThis`，如果构造函数调用不成功或者对象状态不对则会抛出对应的异常。示例代码：

```java
	@Guarded
	public class BusinessObject
	{
		@NotNull
		private String name;

		@PostValidateThis
		public BusinessObject()
		{
			super();
		}
		...
	}
```
下边的代码会抛出期望异常：

```java
	// throws a ConstraintsViolatedException because the name field is null 
	BusinessObject bo = new  BusinessObject();
```
*：因为构造函数并没有给属性`name`赋值，`name`的约束又不能为空，所以在构造函数调用完成过后因为name的值为null会抛出异常。

#### （4）强制执行方法执行后检查
方法执行后检查也是使用：`@net.sf.oval.guard.PostValidateThis`，用法和构造函数差不多：

```java
	@Guarded
	public class BusinessObject
	{
		@Length(max=10)
		private String name = "12345";
		@PostValidateThis
		public appendToName(String appendix)
		{
			name += appendix;
		}
		...
	}
```
下边的代码会抛出期望异常：

```java
	BusinessObject bo = new BusinessObject();
	bo.appendToName("123456"); // throws a ConstraintsViolatedException because field name is now too long
```
### 4.4 使用Probe模式进行简单的用户输入验证
这种用法属于特殊用法，当用户从UI输入数据的时候如果和BusinessObject的`setter`方法的约束冲突则会抛出异常，这些对应的约束信息会被：`ConstraintsViolatedListener`，这个类会检查UI中对应的BusinessObject中的所有约束。
定义中的用法：

```java
	@Guarded
	public class Person
	{
		@NotNegative
		private int age;
		@Min(5)
		private String name = "";
		@Length(min=5, max=5)
		private String zipCode = "";
		public void setAge(@AssertFieldConstraints int age)
		{
			this.age = age;
		}
		public void setName(@AssertFieldConstraints String name)
		{
			this.name = name;
		}
		public void setZipCode(@AssertFieldConstraints String zipCode)
		{
			this.zipCode = zipCode;
		}
		...
	}
```
使用方法如下：

```java
	/* *****************************************************
	 * somewhere in the UI layer
	 * *****************************************************/
	inputForm.setName("1234");
	inputForm.setAge(-4);
	inputForm.setZipCode("123");
	...
	/* *****************************************************
	 * later in the business layer
	 * *****************************************************/
	public Person createPerson(PersonInputForm inputForm) throws ConstraintsViolatedException
	{
		Person person = new Person();
		Guard guard = MyGuardAspect.aspectOf().getGuard();
		// enable the probe mode in the current thread for the person object
		guard.enableProbeMode(person);
		// simulate applying the values to the person bean
		person.setName(inputForm.getName());
		person.setAge(inputForm.getAge());
		person.setZipCode(inputForm.getZipCode());
		// disable the probe mode in the current thread for the person object
		ProbeModeListener result = guard.disableProbeMode(person);
		// check if any constraint violations occured
		if(result.getConstraintViolations().size() > 0)
		{
			// report the collected constraint violations to the UI layer
     		throw new ConstraintsViolatedException(result.getConstraintViolations());
		}
		else
		{
			// apply the values to the person bean
			result.commit();
			dao.save(person);
			return person;
		}
	}
```
### 4.5 转换异常`ConstraintsViolatedExceptions`
如果不想使用OVal中定义的`ConstraintsViolatedExceptions`异常信息，则可使用JRE的标准异常替代：`IllegalArgumentException or IllegalStateException`，OVal允许注册异常转换器，例如下边的代码将异常进行了转换：

```java
	public aspect MyAspect extends GuardAspect
	{
		public MyAspect()
		{
			super();
			// specify an exception translator
			getGuard().setExceptionTranslator(new net.sf.oval.exception.ExceptionTranslatorJDKExceptionsImpl());
		}
	}
```
## 5 定义自定义约束
OVal允许根据不同的需求开发自己的自定义约束，开发流程如下：

1. 创建一个约束类，这个约束类必须实现接口：`net.sf.oval.AnnotationCheck`或从`net.sf.oval.AbstractAnnotationCheck`继承。
	```java
		public class UpperCaseCheck extends AbstractAnnotationCheck<UpperCase>
		{
			public boolean isSatisfied(Object validatedObject, Object valueToValidate, OValContext context, Validator validator)
			{
				if (valueToValidate == null) return true;
				String val = valueToValidate.toString();
				return val.equals(val.toUpperCase());
			}
		}
	```
2. 创建一个约束用的Annotation，并且使用Annotation`@net.sf.oval.configuration.annotation.Constraint`进行标注，并且将上边定义的类设置到`check`属性中：

	```java
		Retention(RetentionPolicy.RUNTIME)
		@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
		@net.sf.oval.configuration.annotation.Constraint(checkWith = UpperCaseCheck.class)
		public @interface UpperCase
		{
			/**
			* message to be used for the ConstraintsViolatedException
			*
			* @see ConstraintsViolatedException
			*/
			String message() default "must be upper case";
		}
	```
3. 在代码中使用自定义的Annotation进行标记。

	```java
		public class BusinessObject 
		{
			@UpperCase 
			private String userId;
			...
		}
	```
如果要针对自定义的Annotation实现国际化操作，则需要安装下边的步骤完成：

1. 为约束使用的Annotation设置一个默认的消息Key值，这个Key必须是Unique的：

	```java
		@Retention(RetentionPolicy.RUNTIME)
		@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
		@net.sf.oval.configuration.annotation.Constraint(checkWith = UpperCaseCheck.class)
		public @interface UpperCase
		{
			/**
			* message to be used for the ConstraintsViolatedException 
			* @see ConstraintsViolatedException
			**/
			String message() default "UpperCase.violated";
		}
	```
2. 创建自定义的Message Bundles（每一种语言创建一个），设置下边的代码段：

		UpperCase.violated={context} must be upper case
	* __{context}__ = 验证的对象，比如一个字段、方法返回值、方法参数、构造函数参数；
	* __{invalidValue}__ = 需要检查的值

	如果需要定义自己的值，则可以重写方法：`createMessageVariables`，比如添加：__{max}, {min}, {size}__等：

	```java
		@Override
		public Map<String, String> createMessageVariables()
		{
			Map<String, String> messageVariables = new HashMap<String, String>(2)
			messageVariables.put("max", Integer.toString(max));
			messageVariables.put("min", Integer.toString(min));
			return messageVariables;
		}
	```
	*：这个地方的方法`createMessageVariables`只会执行一次，OVal会将这个值缓存起来，如果需要销毁缓存中的变量并且重建这个值，可调用方法`requireMessageVariablesRecreation`：比如下边代码：

	```java
		public void setMax(final int max)
		{
			this.max = max;
			requireMessageVariablesRecreation();
		}
	```
3. 最后将不同语言的Bundle注册到OVal中：

	```java
		ResourceBundleMessageResolver resolver = (ResourceBundleMessageResolver) Validator.getMessageResolver();
		resolver.addMessageBundle(ResourceBundle.getBundle("mypackage/CustomMessages"));
	```

## 6 针对特殊约束的复杂表达式定义

### 6.1 使用@ValidateWithMethod

```java
	private static class TestEntity
	{
		@Min(1960)
		private int year = 1977;
		@Range(min=1, max=12)
		private int month = 2;
		@ValidateWithMethod(methodName = "isValidDay", parameterType = int.class)
		private int day = 31;
		private boolean isValidDay(int day)
		{
			GregorianCalendar cal = new GregorianCalendar();
			cal.setLenient(false);
			cal.set(GregorianCalendar.YEAR, year);
			cal.set(GregorianCalendar.MONTH, month - 1);
			cal.set(GregorianCalendar.DATE, day);
			try {
				cal.getTimeInMillis(); // throws IllegalArgumentException
			} catch (IllegalArgumentException e) {
				return false;
			}
			return true;
		}
	}
```
### 6.2 @CheckWith

使用的Annotation：`@net.sf.oval.constraint.CheckWith`

```java
	private static class DayEntity
	{
		@Min(1960)
		private int year;
		@Range(min=1, max=12)
		private int month;
		@CheckWith(DayCheck.class)
		private int day;
		private static class DayCheck implements CheckWithCheck.SimpleCheck
		{
			public boolean isSatisfied(Object validatedObject, Object value)
			{
				try {
					GregorianCalendar cal = new GregorianCalendar();
					cal.setLenient(false);
					cal.set(GregorianCalendar.YEAR, ((DayEntity) validatedObject).year);
					cal.set(GregorianCalendar.MONTH, ((DayEntity) validatedObject).month - 1);
					cal.set(GregorianCalendar.DATE, ((DayEntity) validatedObject).day);
					cal.getTimeInMillis(); // may throw IllegalArgumentException return true;
				} catch (IllegalArgumentException e) {}
				return false;
			}
		}
	}
```
