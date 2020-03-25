# Spring Boot数据源配置

[Reference: 29. Working with SQL database](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html)

Spring中提供了SQL数据库的扩展，根据JDBC的方言【Direct】模板，使用`JdbcTemplate` 来完成关系数据库映射，如Hibernate；Spring Data中提供了额外的数据访问层功能，通过创建一个`Repository` 实现将Query转换成所需要的方法名。

## 1. 配置DataSource

Java中的`javax.sql.DataSource` 接口提供了标准的数据连接池对应方法，传统数据连接池则是使用`URL` 来实现，数据连接池可自定义，参考["How-to" Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html#howto-configure-a-datasource)

### 1.1. 数据库嵌入支持

通常很方便的开发应用使用了嵌入式内存数据库【In-Memory Embedded Database】，实际上这种数据库没提供持久层存储，您需要在应用启动时填充您使用的数据库，并且在应用结束时将这些数据清除。

Spring Boot中支持的自动配置嵌入式数据库包括：H2、HSQL、Derby，您不需要提供任何链接URL，仅仅将您使用的依赖项构建到您使用的数据库中即可。

_**NOTES**：若您想要在测试中使用该功能，您需要在您使用的数据库中实现链接重用，针对不同的测试套件使用统一的上下文环境。若您想要在每一个上下文环境中使用分开的嵌入式数据库环境，则您需要设置：_`spring.datasource.generate-unique-name = true`

**例如**：您的POM文件如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

_**NOTES**：您需要为每个嵌入式数据库配置_`spring-jdbc`_ 的依赖项，这个例子中使用了_`spring-boot-starter-data-jpa`_ 来实现。_

_**NOTES**：不论什么原因，您都需要配置嵌入式数据库中的链接URL，而且必须注意数据库的自动关闭功能被禁用。若您使用了H2，需要设置_`DB_CLOSE_ON_EXIT=FALSE`_ ；若您使用了HSQLDB，则您需要保证_`shutdown=true`_ 没使用。禁用数据库的自动关闭功能可以让Spring Boot在数据库关闭时控制数据库，因此确保数据库关闭时没有这种情况（自动关闭）发生。_

### 1.2. 连接生产环境数据库

生产环境数据库连接可直接通过`DataSource` 自动配置，这是选择连接的实现算法：

* 考虑到性能和并发，我们倾向于使用Tomcat中的`DataSource` ，所以它是一直选择的方案。
* 其次，若可以使用HikariCP，则优先选择。
* 其次选择Commons DBCP，但是我们不推荐在生产环境使用它，因为Commons DBCP已经是Deprecated的。
* 最后，若Commons DBCP2可用，则使用它。

若您使用`spring-boot-starter-jdbc` 或`spring-boot-starter-data-jpa` 时，您会自动获取`tomcat-jdbc` 的依赖项。

_**NOTES**：您可以略过完全的选择算法，而直接使用属性_`spring.datasource.type`_ 来配置，特别是当您的应用在Tomcat容器中使用了_`tomcat-jdbc`_ 作为默认连接池使用时这个配置特别重要。_

_**NOTES**：您也可以手动配置额外的连接池，若您定义了自己的_`DataSource`_ ，自动配置将不会发生。_

DataSource配置主要依赖外部配置`spring.datasource.*` 来控制，如：您在配置文件`application.properties` 中配置了下边代码片段：

```
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

_**NOTES**：您至少应该指定_`spring.datasource.url`_ 属性，否则Spring Boot将会自动配置嵌入式数据库。通常您也需要指定_`driver-class-name`_ ，因为Spring Boot将会根据_`url`_ 中的属性来推断该值。若要创建一个_`DataSource`_ 则您需要验证_`Driver`_ 是否可用，而且它在做其他事前会验证完成。若您设置_`spring.datasource.driver-class-name=com.mysql.jdbc.Driver`_ ，之后这个类将会被载入。_

您可参考`DataSourceProperties` 去查找更多支持选项，无论实际选项如何，都可以根据具体实现使用各自的前缀：`spring.datasource.tocmat.*` ，`spring.datasource.hikari.*` 和`spring.datasource.dbcp2.*` ，这些前缀需要参考特殊连接池实现文档指定细节。

如下边是Tomcat连接池的自定义配置：

```
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```

### 1.3. 连接JNDI DataSource

若您将Spring Boot应用部署到应用程序服务器中，您则想要使用应用程序服务器内置的方式来管理连接池如JNDI。

`spring.datasource.jndi-name` 属性可以用来替换`spring.datasource.url` ，`spring.datasource.username` 和`spring.datasource.password` 而指定JNDI位置访问`DataSource` 。如下边的`application.properties` 片段演示了在JBoss中访问JNDI。

```
spring.datasource.jndi-name=java:jboss/datasources/customers
```

## 2. 使用JdbcTemplate

Spring中的`JdbcTemplate` 和`NamedParameterJdbcTemplate` 都是自动配置的，您可以在您的Bean中直接用`@Autowire` 来引用：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // ...

}
```

## 3. JPA和'Spring Data'

Java持久化API是一种标准技术用来映射对象到关系数据库，属性`spring-boot-starter-data-jpa` POM中提供了快速使用方法，它提供了下边的关键依赖项：

* **Hibernate**：比较流行的一个JPA实现
* **Spring Data JPA**：更加简单实现基于JPA的Repository
* **Spring ORMs**：Spring框架支持的核心ORM

_**NOTES**：Spring Boot默认使用了Hibernate 5.0.x，但是您也可以自己来配置其他版本如4.3.x或5.2.x。_

### 3.1. 实体类

传统的，JPA”实体“类是在`persistence.xml` 中配置的，而在Spring Boot中这个文件可忽略（它可以扫描实体类），默认所有标记了`@EnableAutoConfiguration` 和`@SpringBootApplication` 包中的主类都会被扫描。

其他的标记了`@Entity` ，`@Embeddable` 或`@MappedSuperclass`类也会被扫描。

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.country = country;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }

    // ... etc

}
```

_**NOTES**：您可以在您的实体扫描位置使用_`@EntityScan`_ 标记，参考：_[_Separate @Entity definitions from Spring configuration_](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html#howto-separate-entity-definitions-from-spring-configuration)

### 3.2. Spring Data JPA Repositories

Spring Data JPA Repositories定义了一套访问数据的接口，JPA查询将会从您的方法名中自动创建。例如一个`CityRepository` 接口定义了`findAllByState(String state)` 方法来根据state查找所有城市。对于更多f谁阿查询，您可以参考注解[`@Query` ](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html)。

您定义的接口通常从`Repository` 和`CrudRepository` 接口继承，若您想使用自动配置，这些Repository将会搜索注解了`@EnableAutoConfiguration` 和`@SpringBootApplication` 的类。

下边是一个接口定义：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);

}
```

### 3.3. 创建/销毁JPA数据库

默认的，若您使用了H2，HSQL，Derby时，JPA数据库将会自动创建，您也可以使用`spring.jpa.*` 属性显示指定JPA配置。例：您可以在`application.properties` 中添加下边配置：

```
spring.jpa.hibernate.ddl-auto=create-drop
```

_**NOTES**：Hibernate自己拥有支持该功能的属性_`hibernate.hbm2ddl.auto`_，您可直接使用_`spring.jpa.properties.*`_ 来设置Hibernate原生配置。_

```
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

这样就可以传递`hibernate.globallyguotedidentifiers` 给Hibernate中的Entity Manager，默认的DDL执行（或验证）是不会执行的，它会在`ApplicationContext` 启动完成后执行。这儿有一个`spring.jpa.generate-ddl` 标记，但如果Hibernate自动配置激活了，它就没有用了——会使用原生的`ddl-auto` 配置。

### 3.4. 在视图中开启EntityManager

如果您运行的是一个Web应用，Spring Boot将默认注册[`OpenEntityManagerInViewInterceptor`](http://docs.spring.io/spring/docs/4.3.7.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html) 到当前模式。为了延迟加载Web视图，若您不想要这些行为，您应该在`application.properties` 中设置：

```
spring.jpa.open-in-view=false
```

## 4. 使用H2的Web Console

H2 Database提供了基于浏览器的控制台，Spring Boot可以自动配置，若下边条件满足的话则它可以自动配置：

* 您在开发一个Web应用
* `com.h2database:h2` 在类路径中
* 您在使用[Spring Boot's Developer Tools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)

_**NOTES**：若您没有使用Spring Boot开发工具，但想要使用H2 Web Console，则您可以配置_`spring.h2.console.enabled`_为true，但是这个控制台仅用于开发环境，生产环境中记得关闭Console。_

### 4.1. 改变H2控制台路径

默认H2的Console使用路径`/h2-console` ，您可通过属性`spring.h2.console.path`自己定义。

### 4.2. H2控制台安全性

若类路径中启用了Basic的安全认证，H2 Console将会自动配置成Basic认证，下边的属性可定义相关元数据信息。

* security.user.role
* security.basic.authorize-mode
* security.basic.enabled

## 5. 使用jOOQ

jOOQ——Java Object Oriented Querying是很流行的一个产品，它可以让您从数据库生成Java代码，而让您基于Fluent API使用安全的SQL查询。

### 5.1. 代码生成

若要使用jOOQ安全类型查询，您需要从数据库Schema中生成Java类，您可参考[jOOQ User Manual](http://www.jooq.org/doc/3.6/manual-single-page/#jooq-in-7-steps-step3)查看详情。若您使用了`jooq-codegen-maven` 插件（您同样使用了`spring-boot-starter-parent` 父POM项目），您可忽略`<version>` 标记。您同样可以使用Spring Boot定义版本变量（如`h2.version`）定义数据库依赖插件的版本。下边是一个例子：

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
        ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:~/yourdatabase</url>
        </jdbc>
        <generator>
            ...
        </generator>
    </configuration>
</plugin>
```

### 5.2. 使用DSLContext

jOOQ中的接口`org.jooq.DSLContext`提供的Fluent API，Spring Boot将会自动配置`DSLContext` 为一个Spring Bean来连接您的`DataSource`。若要使用`DSLContext`则您仅仅需要使用`@Autowire`:

```java
@Component
public class JooqExample implements CommandLineRunner {

    private final DSLContext create;

    @Autowired
    public JooqExample(DSLContext dslContext) {
        this.create = dslContext;
    }

}
```

您同样可以使用`DSLContext`来构造您的查询：

```java
public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
        .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
        .fetch(AUTHOR.DATE_OF_BIRTH);
}
```

### 5.3. 定义jOOQ

您可以在`application.properties`中配置`spring.jooq.sql-dialect`定制jOOQ中的SQL方言：

```
spring.jooq.sql-dialect=Postgres
```

更多深入的自定义配置可在`@Bean` 定义中，这些配置将会在jOOQ的`Configuration` 创建时使用，您同样可以定义下边的jOOQ类型：

* ConnectionProvider
* TransactionProvider
* RecordMapperProvider
* RecordListenerProvider
* ExecuteListenerProvider
* VisitListenerProvider

您同样可创建自己的`org.jooq.Configuration` 的`@Bean` 来控制jOOQ的配置。





