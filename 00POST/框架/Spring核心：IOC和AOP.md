# IOC
## IOC的概念和原理
IOC（Inversion of Control），即控制反转，把对象的创建、初始化、销毁（即对象的生命周期，称为bean，参考[Bean的生命周期](https://www.xlxq.fun/posts/48c92662.html) ）交给 Spring 来管理，而不是由开发者控制，实现控制反转。IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂（BeanFactory 接口）。IOC的原理是基于xml解析、工厂设计模式、反射实现的，通过将控制权从程序内部转移到外部来降低组件之间的耦合度，提高代码的可维护性和可测试性。具体来说，IOC 的原理包括以下几个方面：
1. **控制反转**：传统的程序中，对象的创建和管理由程序内部控制，而采用IOC 的方式，控制权被反转到外部，由框架或容器来管理对象的创建和生命周期。这样做可以减少组件之间的直接依赖关系，提高灵活性和可扩展性。
2. **依赖注入**：依赖注入是IOC 的一种实现方式，它通过将对象的依赖关系从对象内部移动到外部容器中，在对象创建时将依赖关系注入到对象中。这样做可以使得对象更加灵活，易于测试和替换。
3. **解耦**：IOC 的主要目的之一是降低组件之间的耦合度，使得组件之间的依赖关系更加松散。这样做可以提高系统的灵活性和可维护性，降低修改一个组件对其他组件的影响。
4. **配置化**：采用IOC 的方式可以将程序的配置信息从代码中抽离出来，以配置文件或注解的方式进行管理。这样做可以使得系统的配置更加灵活，方便进行修改和扩展。
## IOC 容器实现的两种方式
### BeanFactory
1. BeanFactory 是 Spring 框架的核心接口之一，它是一个工厂模式的实现，负责管理和创建 Bean 实例。
2. BeanFactory 采用延迟初始化策略，也就是说，在**调用 getBean() 方法获取 Bean 实例时**才会进行实例化。
3. BeanFactory 提供了基本的 Bean 容器功能，包括 Bean 的实例化、依赖注入、生命周期管理等，但是它的功能相对较少，主要用于低层次的 Bean 容器功能。
### ApplicationContext
1. ApplicationContext 是 BeanFactory 的子接口之一，它在 BeanFactory 的基础上进行了扩展，提供了更多的功能，比如**事件发布、AOP 支持、资源加载**等。
3. ApplicationContext 采用预先实例化的策略，也就是说，在**容器启动时**就会预先实例化所有的 Bean，而不是等到调用 getBean() 方法时才进行实例化。
3. ApplicationContext 是一个更加高级的容器，通常在实际项目中更为常用。
## IOC 操作 Bean 管理
IOC操作Bean管理有两种形式，一种是基于xml方式，另一种是基于注解方式。
1. **基于 XML 配置**：
    - 在基于 XML 的方式中，我们通过在 Spring 的配置文件（通常是applicationContext.xml）中使用 `<bean>`、`<property>` 等标签来定义和配置 Bean。
    - 这种方式需要在 XML 文件中显式地配置每一个 Bean 及其属性，较为传统且繁琐，但是在某些情况下更加直观和可控。
```xml
<bean id="userService" class="com.example.UserService">     
	<property name="userDao" ref="userDao"/> 
</bean>  
```
2. **基于注解方式**：
    - 在基于注解的方式中，我们可以使用 `@Component`、`@Service`、`@Controller`、`@Repository` 等注解来标识一个类为 Bean，并通过 `@Autowired` 注解来实现依赖注入。     
    - 这种方式使得 Bean 的配置更加简洁和灵活，同时也更加符合现代开发的趋势。Spring 在启动时会自动扫描带有这些注解的类，并将其注册为 Bean。
```java
@Service public class UserService {     
	@Autowired     
	private UserDao userDao;     
	// 其他方法 
}
```
# AOP
AOP（Aspect-Oriented Programming，面向切面编程）是一种编程范式，旨在通过将横切关注点（cross-cutting concerns）与核心业务逻辑分离，来提高代码的模块化和可维护性。
在传统的面向对象编程中，我们将功能按照业务逻辑划分为不同的对象和方法，但是有些功能并不属于特定的业务逻辑，而是横跨多个对象和方法的，比如日志、事务管理、安全性控制等。这些横切关注点会使得代码中充斥着重复的代码和混乱的逻辑，导致代码难以理解、维护和测试。AOP 的核心思想是将这些横切关注点抽象成一个单独的模块，称为切面（Aspect），然后通过在特定的切点（Join Point）上织入这些切面，从而将横切关注点与核心业务逻辑分离开来。在 AOP 中，切面可以理解为横跨多个对象和方法的代码片段，而切点则是确定在哪些位置应用这些切面的规则。
## AOP实现方式
AOP底层是使用动态代理来实现的，这里有两种情况的动态代理：
1. 有接口的情况，使用 JDK 动态代理，即创建接口实现类代理对象，增强类的方法。
2. 没有接口的情况，使用 CGLIB 动态代理，即创建子类的代理对象，增强类的方法。
3. 详细的关于代理模式的分析和介绍，请转  [代理模式](https://www.xlxq.fun/posts/7b510e10.html)
## AOP相关概念
1. 连接点：类里面可以被增强的方法，这些方法被称为连接点。
2. 切入点：实际被真正增强的方法，称为切入点。
3. 通知（增强）：
	- 实际增强的逻辑部分称为通知（增强）
	- 通知有多种类型：前置通知、后置通知、环绕通知、异常通知、最终通知
4. 切面：把通知应用到切入点的过程，称为切面。
## AOP使用示例
以下是一个简单的 Spring AOP 示例，演示了如何使用 AOP 实现日志记录功能：
假设有一个接口 `Calculator`，其中包含了两个方法 `add()` 和 `subtract()`：
```java
public interface Calculator {     
	int add(int a, int b);     
	int subtract(int a, int b); 
}
```
然后有一个实现类 `CalculatorImpl`：
```java
public class CalculatorImpl implements Calculator {     
	@Override     public int add(int a, int b) {   return a + b;   }     
	@Override     public int subtract(int a, int b) {  return a - b;   }
}
```
想要在调用这些方法时记录日志，可以使用 AOP 实现。首先定义一个切面类 ：
```java
import org.aspectj.lang.annotation.Aspect; 
import org.aspectj.lang.annotation.Before; 
import org.springframework.stereotype.Component;  
@Aspect 
@Component 
public class LoggingAspect {      
	@Before("execution(* com.example.Calculator.*(..))")     
	public void logBeforeMethodExecution() {  
		System.out.println("LoggingAspect: Method is being executed.");     
	} 
}
```
在上面的切面中，使用了 `@Aspect` 注解来标识这是一个切面类，然后使用 `@Before` 注解来定义切点表达式，表示在执行 `Calculator` 接口中的任何方法之前都会执行 `logBeforeMethodExecution()` 方法。
接下来，我们需要配置 Spring 容器以启用 AOP，可以创建一个配置类：
```java
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.ComponentScan; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.context.annotation.EnableAspectJAutoProxy;  @Configuration 
@ComponentScan("com.example") 
@EnableAspectJAutoProxy 
public class AppConfig {      
	@Bean     
	public Calculator calculator() {         
		return new CalculatorImpl();     
	}      
	@Bean     
	public LoggingAspect loggingAspect() {         
		return new LoggingAspect();     
	} 
}
```
在上面的配置类中，使用 `@EnableAspectJAutoProxy` 注解启用了 Spring 的 AspectJ 自动代理功能，从而使得切面能够生效。

