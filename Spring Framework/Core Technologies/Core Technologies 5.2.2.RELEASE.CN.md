# Core Technologies

version 5.2.2.RELEASE

------

这部分参考文档涵盖了 Spring Framework 所有绝对不可或缺的技术。

其中最重要的是 Spring Framework 的控制反转（IoC）容器。在介绍完 Spring 框架的 IoC  容器之后，紧接着全面介绍 Spring 的面向切面编程（AOP）技术。Spring Framework 有自己的 AOP  框架，它在概念上易于理解，并且成功地定位了 Java 企业编程中 AOP 需求的 80％ 最佳击球点。

Spring 提供与 AspectJ （目前功能最丰富，也是Java企业领域中最成熟的 AOP 实现）的集成。

## 1. The IoC Container

本章讨论Spring的控制反转(IoC)容器。

### 1.1. Introduction to the Spring IoC Container and Beans

本章介绍了 Spring Framework 控制反转（IoC）的实现原理。IoC  也称为依赖注入（DI）。通过这个机制，对象可以通过构造方法参数、工厂方法参数以及通过工厂方法构建或返回的实例上设置的属性来定义它们的依赖关系（即它们使用的其他对象）。然后容器在创建 bean 时注入这些依赖项。这个过程从根本上反转了 Bean  自身通过直接调用构造方法或服务定位模式等机制来控制实例化或定位其依赖的模式，因此叫做控制反转。

`org.springframework.beans` 和 `org.springframework.context` 包是Spring框架的IoC容器的基础。[BeanFactory](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供了能够管理任何类型对象的高级配置机制。ApplicationContext是BeanFactory的子接口。它补充的内容有：

- 更容易与Spring的AOP特性集成
- 消息资源处理(用于国际化)
- 事件发布
- 应用层特定的上下文，例如在 Web 应用程序中使用的 WebApplicationContext

简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext添加了更多针对企业级的功能。ApplicationContext是BeanFactory的一个完整超集，在本章描述Spring的IoC容器时专门使用它。有关使用BeanFactory而不是ApplicationContext的更多信息，请参见[beans-beanfactory](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-beanfactory)。

在Spring中，构成应用程序架构并由Spring IoC容器管理的对象称为beans。bean是由Spring IoC容器实例化、组装和管理的对象。除此之外，bean只是应用程序中的众多对象之一。bean 及其之间的依赖关系反映在容器使用的配置元数据中。

### 1.2. Container Overview

`org.springframework.context.ApplicationContext` 接口代表 Spring IoC 容器，负责实例化、配置和组装bean。容器通过读取配置元数据获取有关需要实例化、配置和组装对象的指令。配置元数据以 XML、Java 注解或Java 代码表示，通过它可以表示构成应用的对象以及对象之间丰富的依赖关系。

Spring 提供了多种 `ApplicationContext` 接口实现。在传统单机应用中，通常会创建一个 [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 或 [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html) 的实例。虽然 XML 是定义配置元数据的传统格式，但你也可以通过提供少量 XML 配置声明容器启用对其他元数据格式的支持后使用 Java 注解或代码作为元数据格式。

在大多数应用程序方案中，不需要显式使用代码来实例化 Spring IoC 容器实例。例如，在 Web 应用程序场景中，应用程序文件 web.xml 中一个 8 行左右的 web 描述模板 XML 就足够了（请参阅  [context-create](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#context-create)）。如果你使用 [Spring Tool Suite](https://spring.io/tools/sts)（一个基于 Eclipse 的开发环境），只需点击几下鼠标或按几下键盘即可轻松创建此模板配置。

下图是 Spring 工作原理的高级视图。应用程序类与配置元数据相结合，在 ApplicationContext 创建并初始化之后，即可拥有完全配置且可执行的系统或应用程序。

![container magic](container-magic.png)

图1所示。Spring IoC容器

#### 1.2.1. Configuration Metadata

如上图所示，Spring IoC 容器使用一系列配置元数据。这些配置元数据描述了Spring 容器在应用程序中如何实例化，配置和组装对象。

传统配置元数据使用简单直观的 XML 格式，本章大部分内容也是用 XML 来表达 Spring IoC 容器的关键概念和功能。

> XML 不是唯一的配置元数据格式。Spring IoC 容器与实际编写配置元数据的格式完全解耦。目前，许多开发人员为其 Spring 应用程序选择[基于 Java 的配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-java)。

有关在 Spring 容器中使用其他形式的元数据的信息，请参阅：

- [基于注解的配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-annotation-config): Spring 2.5 引入了对基于注解的配置元数据的支持。
- [基于java的配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-java): 从 Spring 3.0 开始，Spring JavaConfig 项目提供的许多功能成为 Spring Framework 核心的一部分。因此，你可以使用 Java 而不是 XML 文件来定义应用程序类外部的 bean。要使用这些新功能，请参阅 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)和[`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) 注解

Spring 配置信息由至少一个(通常不止一个) 必须由容器进行管理的 bean 定义组成。基于 XML 的配置元数据将这些 bean 配置为顶级元素 `` 内的 `` 元素。基于 Java 的配置通常在使用 `@Configuration` 注解的类中使用带有 `@Bean` 注解的方法。

这些 bean 定义对应构成应用程序的实际对象。我们一般会定义服务层对象，数据访问对象（DAO），展示层对象（例如 Struts Action  实例），基础结构对象（例如 Hibernate SessionFactories、JMS Queues  等）。一般不会在容器中配置细粒度的域对象，因为通常由 DAO 和业务逻辑负责创建和加载域对象。然而，你可以使用 Spring 集成  AspectJ 来配置非 IoC 容器创建的对象。请参阅[使用 Spring 和 AspectJ 进行域对象的依赖注入](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#aop-atconfigurable)。

以下示例显示了基于 XML 的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

1. id 属性是一个标识单个 bean 定义的字符串。
2. class 属性使用完整的类名定义 bean 的类型。

id 属性的值表示协作对象。在此示例中未包含用于引用协作的对象的 XML。有关更多信息，请参阅[依赖](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-dependencies)。

#### 1.2.2. Instantiating a Container

提供给 ApplicationContext 构造函数的位置路径是一个资源字符串，它允许容器从各种外部资源加载配置元数据，例如本地文件系统、Java 环境变量等。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> 在了解了 Spring 的 IoC 容器之后，你可能想要了解有关 Spring `Resource` 抽象化(参照[resources](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#resources))的更多信息，特别是 `Resource` 路径用于构建应用程序上下文(请参考[resources-app-ctx](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#resources-app-ctx)），它提供了一种便捷的机制从 URI 语句中定义的位置读取 InputStream。

以下示例展示了服务层对象配置文件(services.xml)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->
</beans>
```

以下示例展示了数据访问对象文件(daos.xml)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的例子中，服务层由`PetStoreServiceImpl`类和`JpaAccountDao`和`JpaItemDao`类型的两个数据访问对象(基于JPA对象-关系映射标准)组成。property name元素引用JavaBean属性的名称，而ref元素引用另一个bean定义的名称。id和ref元素之间的这种链接表达了协作对象之间的依赖关系。有关配置对象依赖项的详细信息，请参阅[Dependencies](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-dependencies)。

##### 编写基于XML配置元数据（Composing XML-based Configuration Metadata）

让bean定义跨越多个XML文件可能很有用。通常，每个单独的XML配置文件表示体系结构中的逻辑层或模块。

可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。此构造函数接受多个资源位置，如前一节所示。或者，使用一个或多个`<import/>`元素来从另一个或多个文件加载bean定义。下面的例子演示了如何做到这一点:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的例子中，从三个文件中加载外部 Bean 定义，分别是 services.xml、messageSource.xml 和 themeSource.xml。对于执行导入的定义文件来说，所有路径都是相对路径，因此，services.xml必须与导入文件位于相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于导入文件位置之下的资源位置。可以看到，前面的斜杠被忽略了。但是，考虑到这些路径是相对的，所以最好不要使用斜杠。根据Spring模式，要导入的文件的内容，包括顶级的`<beans/>`元素，必须是有效的XML bean定义。

> 虽然可以使用相对路径“../”引用父目录中的文件，但不建议这样使用，因为这样做会使得当前应用依赖程序之外的文件。非常不建议使用 `classpath:`URL（例如，classpath:../services.xml）引用文件，因为运行时解析过程会选择“最近”的环境变量根目录，然后查找其父目录。环境变量配置的更改可能导致目录选择不正确。
>
> 可以使用完整的资源位置替代相对路径，例如，file:C:/config/services.xml 或  classpath:/config/services.xml。然而需要注意应用程序的配置将会与特定的绝对路径耦合。通常最好为这些绝对路径保持间接联系，例如通过在运行时通过“$ {...}”占位符替代 JVM 系统属性。

名称空间本身提供了import指令特性。除了普通bean定义之外，还有一些配置特性可以在Spring提供的XML名称空间选择中找到——例如，上下文和`util`命名空间。

##### The Groovy Bean Definition DSL

作为外化配置元数据的另一个示例，bean 定义也可以在 Spring 的 Groovy Bean 定义 DSL 中表示，就像 Grails 框架。通常此类配置位于“.groovy”文件中，其结构如下例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置风格在很大程度上等同于XML bean定义，同样支持Spring的XML配置名称空间。它还允许通过importBeans指令导入XML bean定义文件。

#### 1.2.3. Using the Container

`ApplicationContext` 是一个高级工厂接口，主要负责维护不同 bean 及其依赖项的注册。通过使用`T getBean(String name, Class requiredType)`方法可以获得bean的实例。

通过 `ApplicationContext` 可以读取 bean 定义并访问它们，如下例所示：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用 Groovy 配置时，初始化程序看起来非常相似。不同的是使用了支持 Groovy 的上下文实现类（也支持 XML bean 定义）。以下示例展示了 Groovy 配置：

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的使用方式是 `GenericApplicationContext` 与读取器委派结合使用，例如针对 XML 文件使用 `XmlBeanDefinitionReader`，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

还可以使用针对 Groovy 文件使用 `GroovyBeanDefinitionReader` ，如以下示例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

你可以在相同的 `ApplicationContext` 中混合使用此类读取器委托, 从不同的配置源读取 bean 定义。

你可以使用 `getBean` 方法来获取 Bean 实例。`ApplicationContext` 接口还有一些其他方法可以获取 bean，但理想情况下你的应用程序不应该使用它们。实际上，你的应用程序代码根本不应该调用  getBean()方法，也应该不依赖于 Spring API。例如，Spring 集成的 Web 框架为各种 Web 框架组件（如控制器和  JSF 托管的 bean）提供依赖注入，以便通过元数据声明对特定 bean 的依赖关系，例如自动装配注解。

### 1.3. Bean Overview

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的(例如，以XML <bean/>定义的形式)。

在容器内部，这些bean定义被表示为BeanDefinition对象，其中包含(其他信息)以下元数据:

- 包限定的类名：通常是 bean 的实际实现类
- Bean 行为配置元素，定义 bean 在容器中的行为方式（范围，生命周期回调等）
- 引用 bean 执行工作所需的其他 bean，这些引用也称为协作者或依赖项
- 要在新创建的对象中设置的其他配置。例如连接池中连接池的大小和连接数。

元数据可以转换为构成每个 bean 定义的一组属性。下表描述了这些属性：

| 属性                                     | 转换目标        |
| ---------------------------------------- | --------------- |
| 类(Class)                                | 实例化 Bean     |
| 名称(Name)                               | Bean 名称       |
| 范围(Scope)                              | Bean 作用范围   |
| 构造函数参数(Constructor arguments)      | 依赖注入        |
| 属性(Properties)                         | 依赖注入        |
| 自动装配模式(Autowiring mode)            | 自动装配协作者  |
| 延迟初始化模式(Lazy initialization mode) | 延迟初始化 Bean |
| 初始化方法(Initialization method)        | 初始化回调      |
| 销毁方法(Destruction method)             | 销毁回调        |

除了包含如何创建特定 bean 的 bean 定义之外，`ApplicationContext` 实现类还允许用户已经在容器外创建的对象进行注册。这些任务通过使用 `getBeanFactory()` 方法访问应用程序上下文的 Bean 工厂来完成，该方法返回 Bean 工厂的实现类 `DefaultListableBeanFactory`。`DefaultListableBeanFactory` 通过 `registerSingleton(..)` 和 `registerBeanDefinition(..)` 方法支持此注册过程。然而一般情况下应用程序仅使用通过常规 bean 定义元数据定义的 bean。

> Bean  元数据和手动提供的单例实例需要尽可能早的注册，以便容器在自动装配和其他内省步骤期间正确的推理。虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时（与对工厂的实时访问同时）注册新 bean 并未得到官方支持，并且可能导致并发访问异常或 bean 容器中的状态不一致。

#### 1.3.1. Naming Beans

每个bean都有一个或多个标识符。这些标识符在承载bean的容器中必须是惟一的。一个bean通常只有一个标识符。但是，如果需要多个，则可以将额外的视为别名。

在基于xml的配置元数据中，可以使用id属性、name属性或两者都使用来指定bean标识符。`id` 属性允许您指定一个准确的 id。一般来说这些名称包含字母和数字（'myBean'，'someService'等），但它们也可以包含特殊字符。如果你想要为 bean 引入其他别名，还可以在 `name` 属性中指定它们，使用逗号（,）、分号（;）或空格进行分隔。作为历史记录，在 Spring 3.1 之前的版本中，`id` 属性被定义为一种 `xsd:ID` 类型，它限制了可用的字符。从 3.1 开始，它被定义为一种 `xsd:string` 类型。请注意，bean `id` 在容器内必须唯一，但 XML 解析器不会进行限制。

bean 的 `name` 和 `id` 并不是必填的。如果您没有显式地提供name或id，则容器将为该bean生成唯一的名称。但是，如果希望通过名称引用该bean，但是如果需要通过使用 ref 元素或[Service Locator](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-servicelocator) 样式按名称引用该 bean 则必须提供名称。在使用内部 Bean（ [inner beans](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-inner-beans)） 或自动装配协作者（[autowiring collaborators](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)）时可以不提供名称。

> **Bean命名约定** Bean Naming Conventions
>
> 在命名 bean 时约定使用 Java 实例字段命名规范。也就是说，bean 名称以小写字母开头并遵循驼峰法。这样的名字的例子包括 `accountManager`、`accountService`、`userDao`、`loginController`等等。
> 命名 bean 始终使你的配置更易于阅读和理解。此外，如果使用 Spring AOP，那么在将切面织入使用 name 进行关联的一组 bean 时，它会有很大帮助。

> 通过类路径中的组件扫描，Spring 按照前面描述的规则为未命名的组件生成 bean 名称：实质上是简单的使用类名并将其初始字符转换为小写。但是，在特殊情况下，当类名中有多个字符且第一个和第二个字符都是大写字母时，原始格式将被保留。Spring 遵循 `java.beans.Introspector.decapitalize` 定义的规则。

##### Aliasing a Bean outside the Bean Definition （在 Bean 定义之外为其创建别名）

在 bean 定义本身中，可以组合使用单个 `id` 属性和任意数量的 `name` 属性为 bean 提供多个名称。这些名称可以是同一个 bean 的等效别名，在某些情况下很有用，例如让应用程序中的每个组件通过使用特定于该组件本身的 bean 名称来引用公共依赖项。

```xml
<alias name="fromName" alias="toName"/>
```

在这种情况下，名为fromName的bean(在同一个容器中)，在使用别名定义之后，也可以被称为toName。

例如，subsystem A的配置元数据可以通过子`subsystemA-dataSource`的名称引用数据源。subsystem B的配置元数据可以通过`subsystemB-dataSource`的名称引用数据源。在组合使用这两个子系统的主应用程序时，主应用程序以`myApp-dataSource`的名称引用数据源。要使所有三个名称都引用同一个对象，可以将以下别名定义添加到配置元数据:

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过一个唯一的名称引用 dataSource，并保证不与任何其他定义冲突（实际上创建了一个命名空间），但它们引用相同的 bean。

> **基于 Java 的配置**
>
> 如果使用 Javaconfiguration，则 `@Bean` 注解可以提供别名。有关详细信息，请参阅[beans-java-bean-annotation](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation) 。

#### 1.3.2. Instantiating Beans

bean定义本质上是创建一个或多个对象的方法。容器在被请求时查找已经命名的 bean 的方法，并使用由该 bean 定义封装的configuration元数据来创建（或获取）实际对象。

如果使用基于xml的configuration元数据，则指定要在<bean/>元素的class属性中实例化的对象的类型(或类)，这个`class`属性在`BeanDefinition`实例上内部是`Class` property，通常是必需的。 (对于例外，请参阅[Instantiation by Using an Instance Factory Method](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-class-instance-factory-method) 和[Bean Definition Inheritance](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions).)你可以通过以下两种方式之一使用Class属性:

- 容器通过反射调用 Bean 的构造函数（类似于使用 new 运算符的 Java 代码）直接创建 Bean 的情况下指定要构造的 bean 类型。
- 调用指定包含静态工厂方法的实际类来创建对象，在不太常见的情况下，容器调用类中的静态工厂方法来创建 bean。通过调用静态工厂方法返回的对象类型可以完全是同一个类或另一个类。

>内部类名
>如果要为静态内部类配置 bean 定义则必须使用嵌套类的二进制名称。
>例如，如果在 `com.example` 包中有一个类叫做 `SomeThing`，并且 `SomeThing` 类有一个静态内部类叫做 `OtherThing`，则 bean 定义中的 `class`属性值将为 `com.example.SomeThing$OtherThing`。
>请注意，类名中 `$` 字符用于将嵌套类名与外部类名分开。

##### Instantiation with a Constructor

当您通过构造方法创建 bean 时，所有普通类都可以使用并与 Spring  兼容。也就是说，需要开发的类不需要实现任何特定接口或以某种特定方式进行编码。简单地指定 bean 的类就足够了。取决于指定 bean 使用的  IoC 类型，有时候你可能需要一个默认构造函数。

Spring IoC 容器几乎可以管理你希望它管理的任何类。它不仅限于管理真正的 JavaBean。大多数 Spring  用户更倾向于使用只有一个默认构造函数且在属性之后由合适的 setter 和 getter 的 JavaBean。你还可以在容器中包含一些外部的非 bean 样式类。例如，如果需要使用完全不符合 JavaBean 规范的传统连接池，Spring 也可以对其进行管理。

使用基于 XML 的配置元数据，你可以按如下方式指定 bean 类：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关为构造函数提供参数的机制（如果需要）以及在构造对象后设置对象实例属性的详细信息，请参阅[Injecting Dependencies](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators)

##### Instantiation with a Static Factory Method

在定义使用静态工厂方法创建的bean时，使用class属性指定包含静态工厂方法的类，使用factory-method属性指定工厂方法本身的名称。您应该能够调用这个方法(带有可选参数，如后面所述)并返回一个活动对象，该对象随后被视为是通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用静态工厂。

以下 bean 定义演示通过调用工厂方法来创建 bean。该定义未指定返回对象的类型（类），仅指定了包含工厂方法的类。在此示例中，`createInstance()`方法必须是静态方法。以下示例显示如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了一个可以使用之前 bean 定义的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关向工厂方法提供(可选)参数和在从工厂返回对象后设置对象实例属性的机制的详细信息，请参阅[Dependencies and Configuration in Detail](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed)。

##### Instantiation by Using an Instance Factory Method

与通过 [static factory method](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-class-static-factory-method)实例化类似，使用实例工厂方法进行实例化会调用容器现有 bean 的非静态方法来创建新 bean。要使用此机制，请将 `class` 属性保留为空，并在 `factory-bean` 属性中指定当前（或父或祖先）容器中包含调用后创建对象的实例化方法的 bean 的名称。使用 `factory-method` 属性设置工厂方法本身的名称。以下示例显示如何配置此类 bean：

```xml


<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>


```

以下示例展示了相应的Java类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如以下示例所示：

```xml


<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>


```

以下示例展示了相应的Java类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明工厂 bean 本身可以通过依赖注入（DI）进行管理和配置。请参阅[Dependencies and Configuration in Detail](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed)

### 1.4. Dependencies

典型的企业应用程序不会只包含单个对象（或 Spring 术语中的  bean）。即使是最简单的应用程序也是由很多对象进行协同工作，以呈现出最终用户所看到的有条理的应用程序。下一节将介绍如何从定义多个独立的  bean 到实现对象之间相互协作从而实现可达成具体目标的应用程序。

#### 1.4.1. Dependency Injection

依赖注入（DI）是一钟对象处理方式，通过这个过程，对象只能通过构造函数参数、工厂方法参数或对象实例化后设置的属性来定义它们的依赖关系（即它们使用的其他对象）。然后容器在创建 bean 时注入这些依赖项。这个过程从本质上逆转了 bean  靠自己本身通过直接使用类的构造函数或服务定位模式来控制实例化或定位其依赖的情况，因此称之为控制反转。

使用 DI 原则的代码更清晰，当对象和其依赖项一起提供时，解耦更有效。对象不查找其依赖项，也不知道依赖项的位置或类。因此，尤其是依赖允许在单元测试中使用模拟实现的接口或抽象基类时，类会变得更容易测试。

DI 存在两个主要变体：[Constructor-based dependency injection](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-constructor-injection) 和[Setter-based dependency injection](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-setter-injection).

##### Constructor-based Dependency Injection

基于构造函数的 DI 由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。和调用具有特定参数的静态工厂方法来构造 bean 几乎一样，本次讨论用相同的方式处理构造函数和静态工厂方法的参数。以下示例显示了一个只能通过构造函数注入进行依赖注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，这个类没有什么特别之处。它是一个不依赖于容器特定接口、基类或注释的POJO。

###### Constructor Argument Resolution

通过使用的参数类型进行构造函数参数解析匹配。如果 bean 定义的构造函数参数中不存在潜在的歧义，那么在 bean 定义中定义构造函数参数的顺序就是在实例化 bean 时将这些参数提供给适当的构造函数的顺序。参考以下类：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设 `ThingTwo` 类和 `ThingThree` 类没有继承关系，则不存在潜在的歧义。那么，以下配置可以正常工作，你也不需要在 `` 元素中显式指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个 bean 时，类型是已知的，并且可以进行匹配（与前面的示例一样）。当使用简单类型时，例如 `true`，Spring 无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。参考以下类：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

Constructor argument type matching

在前面的场景中，如果使用 `type` 属性显式指定构造函数参数的类型，则容器可以使用指定类型与简单类型进行匹配。如下例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

Constructor argument index

您可以使用 `index` 属性显式指定构造函数参数的索引，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义。

> 索引从0开始。

Constructor argument name

也可以使用构造函数参数名称消除歧义，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，为了使这项工作开箱即用，必须在启用调试标志的情况下编译代码，以便 Spring 可以从构造函数中查找参数名称。如果您不能或不想使用 debug 标志编译代码，则可以使用 JDK 批注 [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) 显式命名构造函数参数。然后，示例类必须如下所示：

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### Setter-based Dependency Injection

基于 setter 的 DI 由容器在调用无参数构造函数或无参数静态工厂方法来实例化 bean 之后调用 setter 方法完成。

以下示例显示了一个只能通过使用纯 setter 注入进行依赖注入的类。这个类是传统的 Java 类。它是一个POJO，它不依赖于特定容器接口、基类或注释。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext` 支持它管理的 bean 使用基于构造函数和基于 setter 的 DI。它还支持在通过构造函数方法注入了一些依赖项之后使用基于 setter 的 DI。你可以以 `BeanDefinition` 的形式配置依赖项，可以将其与 `PropertyEditor` 实例结合使用将属性从一种格式转换为另一种格式。然而，大多数 Spring 用户不直接使用这些类（即编码），而是用 XML `bean` 定义、注解组件（也就是带 `@Component`， `@Controller`等注解的类）或基于 Java 的 `@Configuration` 类中的 `@Bean` 方法。然后，这些源在内部转换为 `BeanDefinition` 实例并用于加载整个 Spring IoC 容器实例。

> 基于构造函数或基于 setter 的 DI？
>
> 由于可以混合使用基于构造函数和基于 setter 的 DI，因此将构造函数用于必填依赖项的同时 setter 方法或配置方法用于可选依赖项是一个很好的经验法则。请注意， 在 setter 方法上使用  [@Required](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-required-annotation)注解可使属性成为必需的依赖项，然而更推荐使用编程式参数验证的构造函数注入。
>
> Spring 团队通常提倡构造函数注入，因为它允许你将应用程序组件实现为不可变对象，并确保所需的依赖项不是 `null`。此外，构造函数注入的组件始终以完全初始化的状态返回给客户端（调用）代码。旁注：大量的构造函数参数是一个糟糕的代码味道，意味着该类可能有太多的责任，应该重构以更好地进行关注点的分离。
>
> Setter 注入应仅用于可在类中指定合理默认值的可选依赖项。否则，必须在代码使用依赖项的所有位置执行非空检查。setter 注入的一个好处是 setter 方法使该类的对象可以在以后重新配置或重新注入。因此，通过 [JMX MBean](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/integration.html#jmx) 进行管理是 setter 注入的一个很好的使用场景。
>
> 使用对特定类最有意义的 DI 方式。有时在处理没有源码的第三方类时需要你自己做选择。例如，如果第三方类没有暴露任何 setter 方法，那么构造函数注入可能是唯一可用的 DI 方式。

##### Dependency Resolution Process

容器执行 bean 依赖性解析过程如下：

- 创建 `ApplicationContext`，之后根据描述所有 Bean 的配置元数据进行初始化。配置元数据可以由 XML、Java代码或注解指定。
- 每个 bean 的依赖关系都以属性、构造函数参数或静态工厂方法参数（如果使用它而不是普通的构造函数）的形式表示。实际创建 bean 时，会将这些依赖项提供给 bean。
- 每个属性或构造函数参数都实际定义了需要设置的值或对容器中另一个 bean 的引用。
- 每个属性或构造函数参数都是一个从其指定的格式转换为该属性或构造函数参数实际类型的值。默认情况下，Spring 能够将提供的字符串格式转换成所有内置类型的值，例如 `int`、`long`、`String`、`boolean`等等。

Spring 容器在创建时验证每个 bean 的配置。但是在实际创建 bean 之前不会设置其属性。作用域为单例且被设置为预先实例化（默认值）的 Bean 会在创建容器时创建。作用域在 [Bean 作用域](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-scopes)中定义。否则 bean 仅在需要时才会创建。创建 bean 可能会导致很多 bean 被创建，因为 bean  的依赖项及其依赖项的依赖项（依此类推）被创建和分配。请注意，这些依赖项之间不匹配的问题可能会较晚才能被发现 - 也就是说，受影响的 bean  首次创建时。

你通常可以相信 Spring 会做正确的事。它会在容器加载时检测配置问题，例如引用不存在的 bean 和循环依赖关系。当实际创建 bean  时，Spring 会尽可能晚地设置属性并解析依赖关系。这意味着容器正常加载后，如果在创建对象或其中一个依赖项时出现问题，Spring  容器会捕获一个异常 - 例如，bean 因属性缺失或无效而抛出异常。可能会稍后发现一些配置问题，所以 `ApplicationContext` 默认情况下实现预实例化单例 bean。在实际需要之前创建这些 bean 是以前期时间和内存为代价的，`ApplicationContext` 会在创建时发现配置问题，而不是更晚。你仍然可以覆盖此默认行为，以便单例 bean 可以延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，当一个或多个协作 bean 被注入到依赖 bean 时，每个协作 bean 在注入到依赖 bean  之前会被完全配置。这意味着，如果 bean A 依赖于 bean B，那么 Spring IoC 容器在调用 bean A 上的 setter  方法之前会完全配置 bean B。换句话说，bean  已经被实例化（如果它不是预先实例化的单例），依赖项已经被设置，并调用了相关的生命周期方法（如[configured init method](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)或  [InitializingBean callback method](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)）。

##### Examples of Dependency Injection

以下示例将基于 XML 的配置元数据用于基于 setter 的 DI。Spring XML 配置文件的一小部分指定了一些 bean 定义，如下所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例展示了相应的 ExampleBean 类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明 setter 与 XML 文件中指定的属性进行匹配。以下示例使用基于构造函数的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例展示了相应的 `ExampleBean` 类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

静态工厂方法的参数由 ``  元素提供，与实际使用的构造函数完全相同。工厂方法返回的类的类型不必与包含静态工厂方法的类相同（尽管在本例中是）。实例（非静态）工厂方法可以以基本相同的方式使用（除了使用 factory-bean 属性而不是 class 属性），因此我们不在此讨论这些细节。

#### 1.4.2. Dependencies and Configuration in Detail

如前一节所述，可以将bean属性和构造函数参数定义为对其他托管bean(协作者)的引用或内联定义的值。Spring基于xml的配置元数据支持其<property/>和<constructor-arg/>元素中的子元素类型。

##### Straight Values (Primitives, Strings, and so on)直接值（基本类型，字符串等）

`<property/>`元素的 `value` 属性指定一个属性或构造器参数为可读的字符串。Spring 的[conversion service](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#core-convert-ConversionService-API)用于将这些值从 `String` 转换为属性或参数的实际类型。以下示例展示了要设置的各种值：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

以下示例使用  [p-namespace](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-p-namespace)进行更简洁的 XML 配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

前面的 XML 更简洁。但是，除非您在创建 bean 定义时使用支持自动属性完成的 IDE（例如 [IntelliJ IDEA](https://www.jetbrains.com/idea/) 或 [Spring Tool Suite](https://spring.io/tools/sts)），否则会在运行时而不是设计时发现拼写错误。强烈建议使用此类 IDE 帮助。

还可以配置 `java.util.Properties` 实例，如下所示：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

###### The `idref` element

`idref` 元素只是一种防错方法，可以将容器中另一个 bean 的 `id` 属性（字符串值 - 而不是引用）传递给 `` 或 `` 元素。以下示例展示了如何使用它：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的 bean 定义代码段与以下代码段完全等效（运行时）：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用 `idref` 标签可以让容器在部署时验证引用的命名 bean 是否存在。在第二个变体中，不会对传递给 `client` bean `targetName` 属性的值执行验证。只有在 `client` bean 真正实例化时才会发现拼写错误（很可能是致命的结果）。如果 `client` bean 是[prototype](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-scopes) bean，那么可能在部署容器后很长时间才能发现此错误和产生的异常。

> 4.0 beans XSD 不再支持 `idref` 元素的 `local` 属性，因为它不再提供除常规 `bean` 引用以外的值。升级到 4.0 版本时，需要将现有 `idref local` 引用更改 `idref bean`。

其中一个共同的地方（至少早于 Spring 2.0 版本），`<idref/>` 元素的值是  `ProxyFactoryBean` bean 定义中   [AOP interceptors](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#aop-pfb-1)的配置项。指定拦截器名称时使用`<idref/>` 元素可防止拼写错误的拦截器 ID。

##### References to Other Beans (Collaborators)引用其他 Bean (协作者)

`ref` 元素是 `<constructor-arg/>` 或 `<property/>` 定义元素内部的最终元素。在这里，你将 bean 指定属性的值设置为对容器管理的另一个 bean（协作者）的引用。引用的 `bean `是要设置属性的 bean 的依赖项，并且在设置该属性之前根据需要对其进行初始化。（如果协作者是单例 bean，它可能已经被容器初始化。）所有引用最终都是对另一个对象的引用。作用域和有效性取决于是否通过 `bean`、`local `或 `parent` 属性指定其他对象的 ID 或名称。

通过 `<ref />` 标签的 `bean `属性指定目标 bean 是最常用的方式，允许创建对同一容器或父容器中的任何 `bean `的引用，不管它是否在同一 XML 文件中。`bean `属性的值可以与目标 Bean 的 `id` 属性相同，也可以与目标 `bean `的 `name `属性之一相同。以下示例演示如何使用 `ref `元素：

```xml
<ref bean="someBean"/>
```

通过 `parent `属性指定目标 bean 会创建对当前容器的父容器中的 `bean `的引用。`parent `属性的值可以与目标 `bean `的 `id `属性相同，也可以与目标 `bean `的 `name `属性之一相同。目标 bean 必须位于当前 `bean `的父容器中。当容器具备层次结构，且希望和父级 `bean `一样通过代理的方式包装父容器中现有的 `bean `时，应该主要使用 `bean `引用变量。以下一对列表演示了如何使用该 `parent` 属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> 4.0 beans XSD 不再支持 `ref `元素的 `local `属性，因为它不再提供除常规 `bean `引用以外的值。升级到 4.0 架构时，需要将现有 `ref ` `local` 引用更改 `ref`  `bean`。

##### Inner Beans内部 Bean

`<property/>` 或 `<constructor-arg/>` 元素中的 `<bean/>` 元素定义一个内部 bean，如下面的示例所示：

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部 bean 定义不需要定义 ID 或名称。即使指定，容器也不使用其作为标识符。容器也会在创建时忽略 `scope` 标志，因为内部 bean 始终是匿名的，并且始终和外部 bean 一起创建。内部 bean 无法单独访问或注入协作 bean 中。

作为一个极端情况，可以从自定义作用域接收销毁回调，例如包含在单例 bean 中作用域为 request 的内部 bean。内部 bean  实例的创建与包含它的 bean 相关联，但是销毁回调允许它参与 request 作用域的生命周期。这不是常见的情况。内部 bean  通常只是共享包含它的 bean 的作用域。

##### Collections集合

`<list/>`、`<set/>`、`<map/>` 和 `<props/>` 元素分别设置 Java Collection 类型 List、Set、Map 和 Properties 的属性和参数。以下示例演示了如何使用它们：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map 的键或值，或 set 的值，可以是以下任何元素：

```xml
bean | ref | idref | list | set | map | props | value | null
```

###### Collection Merging

Spring 容器支持合并集合。应用程序开发人员可以定义父级 `<list/>`、`<map/>`、`<set/>` 或 `<props/> `元素，并定义子元素 `<list/>`、`<map/>`、`<set/>` 或` <props/> `继承并覆盖父集合的值。也就是说，子集合的元素会覆盖父集合中指定的值，子集合的值是合并父集合和子集合的元素的结果。

关于合并的这一部分讨论了父子 bean 机制。不熟悉父子 bean 定义的读者可能希望在继续阅读之前阅读之前阅读[relevant section](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions)。

以下示例演示了集合合并：

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意 `child` bean 定义中 `adminEmails` 属性的 `<props/>` 元素的 `merge=true` 属性的使用。当容器解析并实例化 `child` bean 时，生成的实例有一个 `adminEmails` `Properties` 集合，其中包含将子集合 `adminEmails` 与父 `adminEmails` 集合合并的结果 。以下清单显示了结果：

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

子元素的 `Properties` 集合的值继承了父元素 `<props/>`的所有属性元素，子元素中 `support` 的值将覆盖父集合的值。

这一合并行为同样适用于` <list/>`、`<map/>` 和 `<set/>` 集合类型。在 `<list/> `元素的特定场景中，建议保持与 List 集合类型（即有序集合的概念）相关联的语义。父级的值位于所有子级列表的值之前。在 Map、Set 和 Properties 集合类型场景中，不存在次序的。因此，容器内部使用的 Map、Set 以及 Properties 实现类也是无序的。

###### Limitations of Collection Merging(集合合并的局限性)

你无法合并不同的集合类型（例如 `Map` 和 `List`）。如果你尝试这样做，则会抛出对应的 `Exception`。`merge` 属性必须在较低层级的子定义上指定。在父集合定义上指定 `merge` 属性是多余的，不会导致所需的合并。

###### Strongly-typed collection(强类型集合)

通过在 Java 5 中引入的泛型类型，你可以使用强类型集合。也就是说，可以声明一种 `Collection` 类型，使得它只能包含（例如）`String` 元素。如果使用 Spring 将强类型 `Collection` 依赖注入到 bean 中，可以使用 Spring 的类型转换支持，以便强类型 `Collection` 实例的元素在添加到 `Collection` 之前转换为适当的类型。以下 Java 类和 bean 定义演示了如何执行此操作：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当 `something` bean 的 `accounts` 属性已经准备注入时，通过反射可获得关于强类型的元素 `Map` 的泛型信息。因此，Spring 的类型转换机制将各种值识别为 `Float` 类型，并将字符串值（`9.99`, `2.75` 和 `3.99`）转换为实际 `Float` 类型。

##### Null and Empty String Values(null 和空字符串)

Spring 将属性等的空参数视为空字符串。以下基于 XML 的配置元数据片段将 `email` 属性设置为空字符串（""）。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的示例等效于以下 Java 代码：

```java
exampleBean.setEmail(null);
```

`<null/>` 元素可以处理 `null` 值。以下显示了一个示例：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

上述配置等同于以下 Java 代码：

```java
exampleBean.setEmail(null);
```

##### XML Shortcut with the p-namespace(带有 p 命名空间的 XML 快捷方式)

p 命名空间允许你使用 `bean` 元素的属性（而不是嵌套 `<property/>` 元素）来描述属性值或协作 `bean`。

Spring 支持Spring 支持[具备命名空间](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#xsd-schemas)的可扩展配置格式，这些命名空间基于 XML Schema 定义。本章中讨论的 `beans` 配置格式在一个 XML Schema 文档中定义。但是，p 命名空间未在 XSD 文件中定义，仅存在于 Spring 的核心中。的可扩展配置格式，这些命名空间基于 XML Schema 定义。本章中讨论的 `beans` 配置格式在一个 XML Schema 文档中定义。但是，p 命名空间未在 XSD 文件中定义，仅存在于 Spring 的核心中。

以下示例显示了两个 XML 片段（第一个使用标准 XML 格式，第二个使用 p 命名空间）解析为相同的结果：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例演示了 bean 定义中调用的 p 命名空间设置 `email` 属性。这告诉 Spring 包含一个属性声明。如之前所述，p 命名空间没有模式定义，因此可以将属性的名称设置为属性名。

下一个示例包括另外两个 bean 定义，它们都引用了另一个 bean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

此示例不仅包含使用 p 命名空间的属性值，还使用特殊格式来声明属性引用。第一个 bean 定义使用 `<property name="spouse" ref="jane"/> `创建从 bean john 到 bean jane 的引用 ，而第二个 bean 定义使用 `p:spouse-ref="jane" `作为属性来执行完全相同的操作。在这种情况下，`spouse `是属性名称，而 `-ref` 部分表示这不是直接值，而是对另一个 bean 的引用。

> p 命名空间不如标准 XML 格式灵活。例如，声明属性引用的格式与 `Ref` 结尾的属性冲突，而标准 XML 格式则不会。我们建议仔细选择你的方法并将其传达给其他团队成员，以避免生成同时使用三种方法的 XML 文档。

##### XML Shortcut with the c-namespace(带有 c 命名空间的 XML 快捷方式)

与 [XML Shortcut with the p-namespace](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-p-namespace)类似，Spring 3.1 中引入的 c 命名空间允许使用内联属性来配置构造函数参数，而不是嵌套 `constructor-arg` 元素。

以下示例使用  `c:` 命名空间执行与 [Constructor-based Dependency Injection](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-constructor-injection)相同的操作：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

通过名称设置构造函数参数时，`c:` 命名空间和 `p:` 使用相同的约定（尾部 -ref 的 bean 引用）。类似地，它需要在 XML 文件中声明，即使它没有在 XSD schema 中定义（它存在于 Spring 核心内部）。

对于构造函数参数名称不可用的罕见情况（通常在没有调试信息的情况下编译字节码），可以使用备用的参数索引，如下所示：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

因为采用 XML 语法，且 XML 属性名称不能以数字开头（即使某些 IDE 允许），所以索引表示法要求存在前缀 `_`。`<constructor-arg>` 元素也可以使用相应的索引符号，但不常用，因为声明顺序通常就足够了。

实际上，[mechanism(构造函数解析机制)](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-ctor-arguments-resolution)在匹配参数时非常有效，因此除非确实需要，否则我们建议在整个配置中使用名称表示法。

##### Compound Property Names(复合属性名称)

设置 bean 属性时，可以使用复合或嵌套属性名称，只要除最终属性名称之外的路径的所有组件都不是 null。请看以下 bean 定义：

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

`something `bean 具有一个 `fred `属性，`fred `属性包含 `bob `属性，`bob `属性包含 `sammy `属性，最终 `sammy `属性的值被设置为` 123`。为了确保其可以正常运行，在构造 bean 之后，`something `的 `fred `属性和 `fred `属性的 `bob `属性不得为 null。否则将会抛出 `NullPointerException`。

#### 1.4.3. Using `depends-on`

如果一个 bean 是另一个 bean 的依赖项，那通常意味着将一个 bean 设置为另一个 bean 的属性。通常情况下可以使用基于 XML 的配置元数据中的 `<ref/>` 元素来完成此操作。但是，有时 bean 之间的依赖关系不那么直接。例如需要在类中触发的静态初始化程序，例如数据库驱动程序注册。`depends-on` 属性可以在初始化使用此元素的 bean 之前显式的强制初始化一个或多个 bean。以下示例使用 `depends-on` 属性表示对单个bean的依赖关系：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个 bean 的依赖关系，请提供 bean 名称列表作为 `depends-on` 属性的值（逗号，空格和分号是有效的分隔符）：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

`depends-on` 属性可以指定初始化时的依赖性，也可以指定[singleton](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) bean 销毁时的依赖性。通过 `depends-on` 定义的依赖项会在指定 bean 本身销毁之前被销毁。这样，`depends-on` 也可以控制停止顺序。

#### 1.4.4. Lazy-initialized Beans(延迟初始化的 Bean)

默认情况下，`ApplicationContext` 实现会在初始化时创建和配置所有[singleton](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) bean，作为初始化过程的一部分。通常，这种预先实例化是合理的，因为配置或环境中的错误可以立刻被发现，而不是几小时甚至几天后。当不希望出现这种情况时，可以通过将 bean 定义标记为延迟初始化来阻止单例 bean 的预实例化。延迟初始化的 bean 告诉 IoC 容器在第一次请求时创建 bean  实例，而不是在启动时。

在 XML 中，此行为由` <bean/>` 元素的 `lazy-init` 属性控制，如以下示例所示：

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当前面的配置被 `ApplicationContext` 使用时，`lazy` bean 不会在 `ApplicationContext` 启动时预先实例化，而 `not.lazy` bean 会被预先实例化。

但是当延迟初始化的 bean 是未进行延迟初始化的单例 bean 的依赖项时，`ApplicationContext` 会在启动时创建延迟初始化的 `bean`，因为它必须满足单例的依赖关系。延迟初始化的 `bean` 会被注入到其他不是延迟初始化的单例 bean 中。

你还可以通过使用 `<beans/>` 元素的 `default-lazy-init` 上的属性来控制容器级别的延迟初始化，如以下示例显示：

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

