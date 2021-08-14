# Spring Boot中的测试

[Reference: Spring Boot Testing](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html)

Spring Boot提供了一批Annotation注解帮助您测试应用，有两个模块支持测试功能：

* `spring-boot-test`包含了核心项；
* `spring-boot-test-autoconfigure`支持测试用例的自动配置；

大部分开发者仅仅需要使用`spring-boot-starter-test`项目，这个项目导入了上边两个项目：JUnit，AssertJ，Hamcrest等。

## 1. 测试依赖库

若您在测试范围（scope = test）中使用了`spring-boot-starter-test`，下边的库是已经提供的：

* JUnit——标准Java应用单元测试框架
* Spring Test & Spring Boot Test——和Spring Boot应用集成的测试功能包
* AssertJ——一个Fluent验证框架
* Hamcrest——用来匹配对象专用库
* Mockito——一个Java模拟框架
* JSONassert——针对JSON的断言库
* JsonPath——支持XPath语法的JSON功能

_**NOTES**：默认的Spring Boot使用了Mockito 1.x，当然您也可以直接使用2.x。_

## 2. 测试Spring应用

依赖注入的一个很大的优势就是可以让您的代码变得更加容易测试，您可以使用`new`简单实例化一个对象，当然您也可以直接模拟一个真实依赖对象。通常除了“单元测试”，您还需要转移到“集成测试”中（可直接使用Spring中的`ApplicationContext`），这个功能可以在不需要部署您的应用情况下进行集成测试。Spring Framework中包含了专注于集成测试的模块，您可以直接定义一个`org.springframework:spring-test`的依赖、或者使用`spring-boot-starter-test`依赖项。

若您没有使用`spring-test`的模块，则您需要阅读官方文档来处理。

## 3. 测试Spring Boot应用

Spring Boot应用仅仅是一个Spring的`ApplicationContext`，所以并不需要在测试时做什么特殊处理，仅仅和使用普通Spring环境一致。若您默认使用`SpringApplication`来创建它，有一点需要关注的就是关于外部资源属性、日志、以及默认没有安装在Spring Boot的上下文环境中的功能。

若您想要使用Spring Boot功能，它提供了`@SpringBootTest`注解，可用来替换标准的`spring-test`项目中`@ContextConfiguration`注解。这个注解会在测试`SpringApplication`时创建一个`ApplicationContext`上下文对象。

您可设置`@SpringBootTest`中的`webEnvironment`属性来测试下边功能：



