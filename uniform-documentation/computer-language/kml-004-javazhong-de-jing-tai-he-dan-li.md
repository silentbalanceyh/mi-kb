## 单例模式 vs 静态方法

### 1.代码段
#### 单例模式
定义

```java
    public final class A{
		private static A instance;
		private A(){}
		public static A singleton(){
			synchronized(A.class){
				if(null == instance){
					instance = new A();
				}
				return instance
			}
		}
		public void run(){}
	}
```
使用

```java
	A a = A.singleton();
	a.run();
```

#### 静态方法
定义

```java
	public final class A{
		private A(){}
		public static void run(){}
	}
```
使用

	A.run();

### 2.二者区别

1. 静态类比单例具有更好的性能，因为静态方法在编译器绑定；
2. 它们拥有override的能力，因为Java中的静态方法是不可覆盖的，导致其没有太多的灵活性，另外一方面，可以通过继承方式覆盖单例类中定义的方法；
3. 静态类很难模拟，因此不容易进行单元测试，单例更容易模式，因为比静态类易于编写单元测试，不论什么期望什么，你都可以传递模拟对象，例如构造方法或方法参数；
4. 如果你的需求中需要维护状态信息，则单例比静态类更适合，因为后者在维护状态信息方面是很可怕的，会导致不可预知的BUG；
5. 如果是一个非常重的类，单例可以懒加载，但是静态类没有这样的优势，属于优先加载；
6. 许多依赖注入的框架对单例都有良好的管理，如Spring

### 3.什么时候选择单例？
单例和静态主要优点是前者更加具有面向对象的能力，使用单例可通过继承和多态扩展基类，实现接口和更有能力提供不同的实现，如果讨论java.lang.Runtime，在Java中它是单例，调用getRuntime()方法，会基于不同的JVM返回不同的实现，但也保证了每个JVM中实有一个实例，如果java.lang.Runtime是一个静态类，则不太可能因不同的JVM返回不同的实现。