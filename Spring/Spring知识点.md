# bean

[@bean注解使用](https://segmentfault.com/a/1190000039989422)

[@lazy说明](https://zhuanlan.zhihu.com/p/661020313?utm_id=0)

### 1.将一个类声明为bean的注解（注意是类）

`@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。

`@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。

`@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。

`@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。

### 2.@bean与@component区别

`@Component` 注解作用于类，而`@Bean`注解作用于方法。

`@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。

`@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
//下面这个例子是通过 @Component 无法实现的
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### 3.注入bean的注解

| Annotation   | Package                            | Source       |
| ------------ | ---------------------------------- | ------------ |
| `@Autowired` | `org.springframework.bean.factory` | Spring 2.5+  |
| `@Resource`  | `javax.annotation`                 | Java JSR-250 |
| `@Inject`    | `javax.inject`                     | Java JSR-330 |

`@Autowired` 和`@Resource`使用的比较多一些。

* @resource和@autowired区别

  * autowired：`Autowired` 属于 Spring 内置的注解，**默认的注入方式为`byType`**（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类），当一个接口存在多个实现类的话，`byType`这种方式就无法正确注入对象了，因为这个时候 Spring 会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。**这种情况下，注入方式会变为 `byName`（根据名称进行匹配）**，这个名称通常就是类名（首字母小写）。

    **注：`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。如果出现如下情况，则会报错**

    ```java
    // 报错，byName 和 byType 都无法匹配到 bean
    @Autowired
    private SmsService smsService;
    // 正确注入 SmsServiceImpl1 对象对应的 bean
    @Autowired
    private SmsService smsServiceImpl1;
    // 正确注入  SmsServiceImpl1 对象对应的 bean
    // smsServiceImpl1 就是我们上面所说的名称
    @Autowired
    @Qualifier(value = "smsServiceImpl1")
    private SmsService smsService;
    
    ```

  * @Resource：属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

    `@Resource` 有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）

    ```java
    public @interface Resource {
        String name() default "";
        Class<?> type() default Object.class;
    }
    
    ```
  
    如果仅指定 `name` 属性则注入方式为`byName`，如果仅指定`type`属性则注入方式为`byType`，如果同时指定`name` 和`type`属性（不建议这么做）则注入方式为`byType`+`byName`。
  
    ```java
    // 报错，byName 和 byType 都无法匹配到 bean
    @Resource
    private SmsService smsService;
    // 正确注入 SmsServiceImpl1 对象对应的 bean
    @Resource
    private SmsService smsServiceImpl1;
    // 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
    @Resource(name = "smsServiceImpl1")
    private SmsService smsService;
    ```

###     3.bean的作用域

注：以下作用域肯定使用@bean修饰的方法，至于@component等是否有用需要确定

* **singleton** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
* **prototype** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
* **request** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
* **session** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
* **application/global-session** （仅 Web 应用可用）：每个 Web 应用在启动时创建一个 Bean（应用 Bean），该 bean 仅在当前应用启动时间内有效。
* **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

### 4.bean生命周期

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

# IOC

> - 什么是 IoC？
> - IoC 解决了什么问题？
> - IoC 和 DI 的区别？

IoC （Inversion of Control ）即控制反转/反转控制，不通过 new 关键字来创建对象，而是通过 IoC 容器(Spring 框架) 来帮助我们实例化对象。我们需要哪个对象，直接从 IoC 容器里面去取即可。

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（IoC 容器）

### 1.IOC解决什么问题

1. 对象之间的耦合度或者说依赖程度降低；
2. 资源变的容易管理；比如你用 Spring 容器提供的话很容易就可以实现一个单例。

### 2.IOC与DI

DI是IOC的一种实现方式，IoC（Inverse of Control:控制反转）是一种设计思想或者说是某种模式。这个设计思想就是 **将原本在程序中手动创建对象的控制权交给第三方比如 IoC 容器**。

# AOP

> - 什么是 AOP、AOP 解决了什么问题？
> - AOP 的应用场景有哪些？
> - AOP 为什么叫做切面编程？
> - AOP 实现方式有哪些？

AOP是面向切面编程，AOP能够将那些与**业务无关**，却为业务模块所**共同调用的逻辑或责任**（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理

> 注意：代理方案分情况，实现了接口的类直接使用JDK proxy进行代理速度更快，没实现的用cglib进行代理

AOP常见的应用场景有：

> - 日志记录：自定义日志记录注解，利用 AOP，一行代码即可实现日志记录。
> - 性能统计：利用 AOP 在目标方法的执行前后统计方法的执行时间，方便优化和分析。
> - 事务管理：`@Transactional` 注解可以让 Spring 为我们进行事务管理比如回滚异常操作，免去了重复的事务管理逻辑。`@Transactional`注解就是基于 AOP 实现的。
> - 权限控制：利用 AOP 在目标方法执行前判断用户是否具备所需要的权限，如果具备，就执行目标方法，否则就不执行。例如，SpringSecurity 利用`@PreAuthorize` 注解一行代码即可自定义权限校验。
> - 接口限流：利用 AOP 在目标方法执行前通过具体的限流算法和实现对请求进行限流处理。
> - 缓存管理：利用 AOP 在目标方法执行前后进行缓存的读取和更新

### 1.Spring AOP与AspectJ AOP区别

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多

### 2.AOP基本术语

- 切入点（pointcut）：在哪些类、哪些方法上切入，通常是一个正则表达式

- 执行点（JoinPoint）：通过pointcut选取出来的集合中的具体的一个执行点，我们就叫JoinPoint
- 通知（advice）：在方法前、方法后、方法前后、异常等做什么。
- 切面（aspect）：切面 = pointcut + advice。即在什么时机、什么地方、做什么。
- 织入（weaving）：把切面加入对象，并创建出代理对象的过程。

### 3.AspectJ通知类型

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知）：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法

### 4.多个切面控制顺序

1、通常使用`@Order` 注解直接定义切面顺序

```java
// 值越小优先级越高
@Order(3)
@Component
@Aspect
public class LoggingAspect implements Ordered {
```

**2、实现`Ordered` 接口重写 `getOrder` 方法。**

```java
@Component
@Aspect
public class LoggingAspect implements Ordered {

    // ....

    @Override
    public int getOrder() {
        // 返回值越小优先级越高
        return 1;
    }
}
```



# SpringMVC

### 1.MVC是什么

MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

### 2.SpringMVC核心组件

[SpringMVC流程代码分析](https://blog.51cto.com/u_15942107/6019688)

[SpringMVC参数解析分析](https://www.cnblogs.com/leanring/p/14043224.html)

- **`DispatcherServlet`**：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`**：**处理器映射器**，根据 URL 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`**：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`**：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`**：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端

![img](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/de6d2b213f112297298f3e223bf08f28.png)

### 3.统一异常处理

使用到 `@ControllerAdvice` + `@ExceptionHandler` 这两个注解 

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

异常处理器匹配原理如下：

会给所有或者指定的 `Controller` 织入异常处理的逻辑（AOP），当 `Controller` 中的方法抛出异常的时候，由被`@ExceptionHandler` 注解修饰的方法进行处理。

`ExceptionHandlerMethodResolver` 中 `getMappedMethod` 方法决定了异常具体被哪个被 `@ExceptionHandler` 注解修饰的方法处理异常。从源代码看出：**`getMappedMethod()`会首先找到可以匹配处理异常的所有方法信息，然后对其进行从小到大的排序，最后取最小的那一个匹配的方法(即匹配度最高的那个)**

```java
@Nullable
  private Method getMappedMethod(Class<? extends Throwable> exceptionType) {
    List<Class<? extends Throwable>> matches = new ArrayList<>();
    //找到可以处理的所有异常信息。mappedMethods 中存放了异常和处理异常的方法的对应关系
    for (Class<? extends Throwable> mappedException : this.mappedMethods.keySet()) {
      if (mappedException.isAssignableFrom(exceptionType)) {
        matches.add(mappedException);
      }
    }
    // 不为空说明有方法处理异常
    if (!matches.isEmpty()) {
      // 按照匹配程度从小到大排序
      matches.sort(new ExceptionDepthComparator(exceptionType));
      // 返回处理异常的方法
      return this.mappedMethods.get(matches.get(0));
    }
    else {
      return null;
    }
  }
```

# Spring事务

### 1.手动事务

通过 `TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用，但是对于理解 Spring 事务管理原理有帮助。

```java
//使用TransactionTemplate 进行编程式事务管理的示例代码如下：
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
//使用 TransactionManager 进行编程式事务管理的示例代码如下
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // ....  业务代码
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```

### 2.声明式事务

推荐使用（代码侵入性最小），实际是**通过 AOP 实现**（基于`@Transactional` 的全注解方式使用最多）

```java
@Transactional(propagation = Propagation.REQUIRED)
public void aMethod {
  //do something
  B b = new B();
  C c = new C();
  b.bMethod();
  c.cMethod();
}
```

### 3.Spring事务类与接口

我们可以把 **`PlatformTransactionManager`** 接口可以被看作是事务上层的管理者，而 **`TransactionDefinition`** 和 **`TransactionStatus`** 这两个接口可以看作是事务的描述。

**`PlatformTransactionManager`** 会根据 **`TransactionDefinition`** 的定义比如事务超时时间、隔离级别、传播行为等来进行事务管理 ，而 **`TransactionStatus`** 接口则提供了一些方法来获取事务相应的状态比如是否新事务、是否可以回滚等等。

Spring 框架中，事务管理相关最重要的 3 个接口如下：

- **`PlatformTransactionManager`**：（平台）事务管理器，Spring 事务策略的核心。

  **Spring 并不直接管理事务，而是提供了多种事务管理器** 。Spring 事务管理器的接口是：**`PlatformTransactionManager`** 。

  通过这个接口，Spring 为各个平台如：JDBC(`DataSourceTransactionManager`)、Hibernate(`HibernateTransactionManager`)、JPA(`JpaTransactionManager`)等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

  ```java
  package org.springframework.transaction;
  
  import org.springframework.lang.Nullable;
  
  public interface PlatformTransactionManager {
      //获得事务
      TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
      //提交事务
      void commit(TransactionStatus var1) throws TransactionException;
      //回滚事务
      void rollback(TransactionStatus var1) throws TransactionException;
  }
  ```

- **`TransactionDefinition`**：事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)。

  事务管理器接口 **`PlatformTransactionManager`** 通过 **`getTransaction(TransactionDefinition definition)`** 方法来得到一个事务，这个方法里面的参数是 **`TransactionDefinition`** 类 ，这个类就定义了一些基本的事务属性。

  **什么是事务属性呢？** 事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。

  事务属性包含了 5 个方面：

  - 隔离级别
  - 传播行为
  - 回滚规则
  - 是否只读
  - 事务超时

- **`TransactionStatus`**：事务运行状态。

  `TransactionStatus`接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息。

  `PlatformTransactionManager.getTransaction(…)`方法返回一个 `TransactionStatus` 对象

  ```java
  public interface TransactionStatus{
      boolean isNewTransaction(); // 是否是新的事务
      boolean hasSavepoint(); // 是否有恢复点
      void setRollbackOnly();  // 设置为只回滚
      boolean isRollbackOnly(); // 是否为只回滚
      boolean isCompleted; // 是否已完成
  }
  ```

### 4.事务传播行为(重要)

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

举个例子：我们在 A 类的`aMethod()`方法中调用了 B 类的 `bMethod()` 方法。这个时候就涉及到业务层方法之间互相调用的事务问题。如果我们的 `bMethod()`如果发生异常需要回滚，如何配置事务传播行为才能让 `aMethod()`也跟着回滚呢？这个时候就需要事务传播行为的知识了，如果你不知道的话一定要好好看一下。

```java
@Service
Class A {
    @Autowired
    B b;
    @Transactional(propagation = Propagation.xxx)
    public void aMethod {
        //do something
        b.bMethod();
    }
}

@Service
Class B {
    @Transactional(propagation = Propagation.xxx)
    public void bMethod {
       //do something
    }
}
```

* **`TransactionDefinition.PROPAGATION_REQUIRED`**

  `@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

  - 如果外部方法没有开启事务的话，`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
  - 如果外部方法开启事务并且被`Propagation.REQUIRED`的话，所有`Propagation.REQUIRED`修饰的内部方法和外部方法均属于同一事务 ，只要一个方法回滚，整个事务均回滚。

  ```java
  //如果我们上面的aMethod()和bMethod()使用的都是PROPAGATION_REQUIRED传播行为的话，两者使用的就是同一个事务，只要其中一个方法回滚，整个事务均回滚
  @Service
  Class A {
      @Autowired
      B b;
      @Transactional(propagation = Propagation.REQUIRED)
      public void aMethod {
          //do something
          b.bMethod();
      }
  }
  @Service
  Class B {
      @Transactional(propagation = Propagation.REQUIRED)
      public void bMethod {
         //do something
      }
  }
  
  ```

* **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

  创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰

  ```java
  //如果我们上面的bMethod()使用PROPAGATION_REQUIRES_NEW事务传播行为修饰，aMethod还是用PROPAGATION_REQUIRED修饰的话。如果aMethod()发生异常回滚，bMethod()不会跟着回滚，因为 bMethod()开启了独立的事务。但是，如果 bMethod()抛出了未被捕获的异常并且这个异常满足事务回滚规则的话,aMethod()同样也会回滚，因为这个异常被 aMethod()的事务管理机制检测到了
  @Service
  Class A {
      @Autowired
      B b;
      @Transactional(propagation = Propagation.REQUIRED)
      public void aMethod {
          //do something
          b.bMethod();
      }
  }
  
  @Service
  Class B {
      @Transactional(propagation = Propagation.REQUIRES_NEW)
      public void bMethod {
         //do something
      }
  }
  ```

* **`TransactionDefinition.PROPAGATION_NESTED`**

  如果当前存在事务，就在嵌套事务内执行；如果当前没有事务，就执行与`TransactionDefinition.PROPAGATION_REQUIRED`类似的操作。也就是说：

  - 在外部方法开启事务的情况下，在内部开启一个新的事务，作为嵌套事务存在。
  - 如果外部方法无事务，则单独开启一个事务，与 `PROPAGATION_REQUIRED` 类似。

  ```java
  //如果 bMethod() 回滚的话，aMethod()不会回滚。如果 aMethod() 回滚的话，bMethod()会回滚
  @Service
  Class A {
      @Autowired
      B b;
      @Transactional(propagation = Propagation.REQUIRED)
      public void aMethod {
          //do something
          b.bMethod();
      }
  }
  
  @Service
  Class B {
      @Transactional(propagation = Propagation.NESTED)
      public void bMethod {
         //do something
      }
  }
  
  ```

* **`TransactionDefinition.PROPAGATION_MANDATORY`**

  如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常

* **`TransactionDefinition.PROPAGATION_SUPPORTS`**

  如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行

* **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**

  以非事务方式运行，如果当前存在事务，则把当前事务挂起（什么叫挂起）

* **`TransactionDefinition.PROPAGATION_NEVER`**

  以非事务方式运行，如果当前存在事务，则抛出异常

### 5.事务隔离级别

- **`TransactionDefinition.ISOLATION_DEFAULT`** :使用后端数据库默认的隔离级别，MySQL 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别.
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`** :最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **`TransactionDefinition.ISOLATION_READ_COMMITTED`** : 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`** : 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`** : 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 6.回滚规则

默认情况下，事务只有遇到运行期异常（`RuntimeException` 的子类）时才会回滚，`Error` 也会导致事务回滚，但是，在遇到检查型（Checked）异常时不会回滚。

```java
//如果你想要回滚你定义的特定的异常类型的话
@Transactional(rollbackFor= MyException.class)
```

### 7.@Transactional使用范围

- **方法**：推荐将注解使用于方法上，不过需要注意的是：**该注解只能应用到 public 方法上，否则不生效。**
- **类**：如果这个注解使用在类上的话，表明该注解对该类中**所有的 public 方法都生效**。
- **接口**：不推荐在接口上使用

### 8.事务超时设置

就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 `TransactionDefinition` 中以 int 的值来表示超时时间，其单位是秒，默认值为-1，这表示事务的超时时间取决于底层事务系统或者没有超时时间

```java
@Transactional(timeout=)
```

### 9.自调用问题

当一个方法被标记了`@Transactional` 注解的时候，Spring 事务管理器只会在被其他类方法调用的时候生效，而不会在一个类中方法调用生效。

这是因为 Spring AOP 工作原理决定的。因为 Spring AOP 使用动态代理来实现事务的管理，它会在运行的时候为带有 `@Transactional` 注解的方法生成代理对象，并在方法调用的前后应用事物逻辑。如果该方法被其他类调用我们的代理对象就会拦截方法调用并处理事务。但是在一个类中的其他方法内部调用的时候，我们代理对象就无法拦截到这个内部调用，因此事务也就失效了**。解决办法就是避免同一类中自调用或者使用 AspectJ 取代 Spring AOP 代理。**

```java
//MyService 类中的method1()调用method2()就会导致method2()的事务失效。
@Service
public class MyService {

private void method1() {
     method2();
     //......
}
@Transactional
 public void method2() {
     //......
  }
}

```

或者用如下解决方案

```java
@Service
public class MyService {

private void method1() {
     ((MyService)AopContext.currentProxy()).method2(); // 先获取该类的代理对象，然后通过代理对象调用method2。
     //......
}
@Transactional
 public void method2() {
     //......
  }
}
```



# Spring参数校验

### 1.请求参数校验

https://javaguide.cn/system-design/framework/spring/spring-common-annotations.html#_6-%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C

### 2.配置参数校验

使用@ConfigurationProperties+@Validated+@EnableConfigurationProperites，当properties参数校验不通过时项目会抛出异常，无法正常启动

# Spring设计模式

https://javaguide.cn/system-design/framework/spring/spring-design-patterns-summary.html

# SpringBoot自动装配

> 1. 什么是 SpringBoot 自动装配？
> 2. SpringBoot 是如何实现自动装配的？如何实现按需加载？
> 3. 如何实现一个 Starter？

SpringBoot 定义了一套接口规范，这套规范规定：SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot。

### 1.@SpringBootApplication注解

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan`：扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除`TypeExcludeFilter`和`AutoConfigurationExcludeFilter`。

![img](https://oss.javaguide.cn/p3-juejin/bcc73490afbe4c6ba62acde6a94ffdfd~tplv-k3u1fbpfcp-watermark.png)

### 2.@EnableAutoConfiguration

`EnableAutoConfiguration` 只是一个简单地注解，自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类。

`AutoConfigurationImportSelector` 类实现了 `ImportSelector`接口，也就实现了这个接口中的 `selectImports`方法，该方法主要用于**获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中**。

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}

```

getAutoConfigurationEntry通过扫描加载的jar中对应的/META-INF/spring.factories加载自动配置类

```java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.判断自动装配开关是否打开
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>.获取所有需要装配的bean
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            //getAutoConfigurationEntry
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }

```

//getAutoConfigurationEntry分析

```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>.
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>. 用于获取EnableAutoConfiguration注解中的 exclude 和 excludeName。
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>.获取需要自动装配的所有配置类，读取META-INF/spring.factories
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>.这一步有经历了一遍筛选，@ConditionalOnXXX 中的所有条件都满足，该类才会生效
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
//1isEnabled
//判断自动装配开关是否打开。默认spring.boot.enableautoconfiguration=true，可在 application.properties 或 application.yml 中设置
protected boolean isEnabled(AnnotationMetadata metadata) {
		if (getClass() == AutoConfigurationImportSelector.class) {
			return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
		}
		return true;
	}

```

### 3.starter

[实现自定义starter](https://javaguide.cn/system-design/framework/spring/spring-boot-auto-assembly-principles.html)

SpringBoot Starters 是一系列依赖关系的集合，因为它的存在，项目的依赖之间的关系对我们来说变的更加简单了

### 4.软件配置

* 配置jetty作为web容器

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <!-- 排除默认的 Tomcat 依赖 -->
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-tomcat</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jetty</artifactId>
  </dependency>
  ```

### 5.配置文件优先级

优先级由高到底顺序，高优先级会覆盖底优先级（注意还可以通过spring.config.location的启动参数来指定）：

* 项目目录下config目录下的配置文件
* 项目目录下配置文件
* classpath下config目录下配置文件
* classpath下配置文件

### 6.定时任务

使用@Scheduled注解实现，且springboot启动类需要加上@EnableScheduling
