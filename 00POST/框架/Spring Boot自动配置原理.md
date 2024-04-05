Spring Boot自动配置是它的一个核心特性，它能够根据应用程序的依赖、类路径、配置和其他因素来自动配置Spring应用程序。
# @SpringBootApplication
Spring Boot的启动类上的注解@SpringBootApplication是实现自动配置的关键，跟踪可以看到@SpringBootApplication包含许多注解![[Pasted image 20240303201036.png]]
下面依次说明各注解的作用
1. `@Target({ElementType.TYPE})`: 这个注解指定了该自定义注解可以应用的目标元素类型。在这里，它指定了这个注解只能应用于类（Type）。
2. `@Retention(RetentionPolicy.RUNTIME)`: 这个注解指定了该自定义注解在运行时可见。在这里，它表示这个注解在运行时会保留，因此可以通过**反射**来访问它。
3. `@Documented`: 这个注解指定了当Java文档工具（如Javadoc）生成文档时，是否将这个注解包含在文档中。在这里，它表示这个注解会被文档化。
4. `@Inherited`: 这个注解指定了子类是否会继承父类的注解。在这里，它表示子类会继承这个注解。
5. `@SpringBootConfiguration`: 这个注解是Spring Boot提供的一个特殊的配置类注解，它等同于`@Configuration`。用于标识一个类是Spring Boot应用程序的配置类，通常用于定义Bean。
6. `@EnableAutoConfiguration`: 这个注解启用了Spring Boot的自动配置功能。它会根据类路径中的情况自动配置Spring应用程序。这是Spring Boot自动配置的核心注解之一。
7. `@ComponentScan`: 这个注解指示Spring在指定的包及其子包中搜索带有注解的组件。它可以用于指定要扫描的包路径。
# @EnableAutoConfiguration
@EnableAutoConfiguration是Spring Boot自动配置的核心注解之一，跟踪可以发现它同样包含许多注解，其中@AutoConfigurationPackage和@Import是自动配置的关键
![[Pasted image 20240303201722.png]]
1. `@AutoConfigurationPackage`: 这个注解用于指示Spring Boot应该自动配置位于指定包及其子包中的Bean。它会将该包及其子包下的所有组件加入到Spring Boot的自动配置中。
2. `@Import`: 这个注解是Spring的核心注解之一，它用于导入其他配置类或者配置项。在这里，通过导入`AutoConfigurationImportSelector.class`，实现了自动配置的导入。
# 类AutoConfigurationImportSelector
@Import导入了类AutoConfigurationImportSelector，这个类实现了DeferredImportSelector，再跟进可以发现DeferredImportSelector继承自ImportSelector，而ImportSelector中的方法selectImports的返回值就保存着需要自动配置的类的全类名
```java
String[] selectImports(AnnotationMetadata importingClassMetadata);
```
它在AutoConfigurationImportSelector中的实现如下
```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {  
    if (!isEnabled(annotationMetadata)) {  
       return NO_IMPORTS;  
    }  
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);  
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());  
}
```
容易发现，返回值是通过getAutoConfigurationEntry方法获取并通过getConfigurations获取的，继续跟进getAutoConfigurationEntry方法
```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {  
    if (!isEnabled(annotationMetadata)) {  
       return EMPTY_ENTRY;  
    }  
    AnnotationAttributes attributes = getAttributes(annotationMetadata);  
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);  
    configurations = removeDuplicates(configurations);  
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);  
    checkExcludedClasses(configurations, exclusions);  
    configurations.removeAll(exclusions);  
    configurations = getConfigurationClassFilter().filter(configurations);  
    fireAutoConfigurationImportEvents(configurations, exclusions);  
    return new AutoConfigurationEntry(configurations, exclusions);  
}
```
可以看出我们需要的configurations集合来自getCandidateConfigurations方法，继续跟进
```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {  
    List<String> configurations = ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader())  
       .getCandidates();  
    Assert.notEmpty(configurations,  
          "No auto configuration classes found in META-INF/spring.factories nor in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you are using a custom packaging, make sure that file is correct.");  
    return configurations;  
}
```
这里的文本信息很关键，告诉我们auto configuration classes（自动配置类）是在两个文件中获取的（springboot3.0之后已经取消了META-INF/spring.factories），找到这两个文件，可以发现其中确实存在许多全类名，随便选一个跟进，可以发现有@AutoConfiguration，而这个注解又包含@Configuration，也就是说，它是一个配置类，并且提供了被@Bean修饰的方法来返回bean对象，因此，一旦该类被加载，对应的bean就会自动进入IOC容器，我们后续就可以使用，这就是所谓的自动配置
# 流程梳理
经过上述跟进源码，我们可以梳理一下整个流程：
1. Spring Boot实现自动配置的关键注解
	- @SpringBootApplication--->@EnableAutoConfiguration
	- @EnableAutoConfiguration--->@Import({AutoConfigurationImportSelector.class})
2. AutoConfigurationImportSelector类中的方法selectImports返回需要加载的配置类
	- selectImports--->getAutoConfigurationEntry
	- getAutoConfigurationEntry--->getCandidateConfigurations
	- getCandidateConfigurations--->META-INF/spring.factories文件
3. META-INF/spring.factories文件中存放了所有需要进行自动配置的类
# @ConditionalOnXXX
一个配置类中通常会包含许多bean，那么这些bean全部都会随着配置类的加载进入IOC容器吗，当然不会，这就是@ConditionalOnXXX系列注解的作用
它用于配置类或者组件上(@Bean修饰的方法)上，可以根据项目的特定需求和环境来决定是否加载某个组件或配置，从而实现更加灵活和可配置的应用程序开发。
常见的`@ConditionalOnXXX`注解及其作用：
1. `@ConditionalOnClass`: 当指定的类位于类路径中时，才会生效。
2. `@ConditionalOnMissingClass`: 当指定的类不位于类路径中时，才会生效。
3. `@ConditionalOnBean`: 当指定的Bean存在于Spring应用程序上下文中时，才会生效。
4. `@ConditionalOnMissingBean`: 当指定的Bean不存在于Spring应用程序上下文中时，才会生效。
5. `@ConditionalOnProperty`: 当指定的属性满足条件时，才会生效。
6. `@ConditionalOnResource`: 当指定的资源存在时，才会生效。
7. `@ConditionalOnWebApplication`: 当应用程序是一个Web应用程序时，才会生效。
8. `@ConditionalOnNotWebApplication`: 当应用程序不是一个Web应用程序时，才会生效。
9. `@ConditionalOnExpression`: 当指定的SpEL表达式计算结果为true时，才会生效。

# 总结
Spring Boot自动配置原理：
1. @SpringBootApplication包含若干注解，其中比较重要的是@ComponentScan，@EnableAutoConfiguration，@@SpringBootApplication，而@EnableAutoConfiguration是实现自动配置的关键注解
2. @EnableAutoConfiguration包含@AutoConfigurationPackage和@Import两个关键注解，@AutoConfigurationPackage指明Spring Boot应该自动配置位于指定包及其子包中的Bean，而@Import导入的类AutoConfigurationImportSelector中则通过selectImports方法扫描了META-INF/spring.factories文件，并加载了其中的所有配置类
3. 配置类加载后，其中的bean根据其条件注解@ConditionalOnXXX决定是否被放入IOC容器中
4. 可以从IOC容器中获取bean对象，自动配置完成
# 补充
## @ComponentScan的参数
`@ComponentScan`注解可以包含多个参数，用于指定扫描的方式和范围。下面是`@ComponentScan`注解常用的参数：
1. **basePackages**: 指定要扫描的基础包。可以指定一个或多个包路径，多个包路径之间用逗号分隔。如果不指定该参数，默认扫描注解所在类的包及其子包。
2. **basePackageClasses**: 指定要扫描的基础类。通常是一些标志性的类，`@ComponentScan`会扫描这些类所在的包及其子包。如果同时指定了`basePackages`和`basePackageClasses`，则`basePackages`会被忽略。
3. **includeFilters**: 指定包含过滤器，用于指定哪些类应该被包含在扫描范围内。默认为空数组。可以通过`@Filter`注解指定过滤条件。
4. **excludeFilters**: 指定排除过滤器，用于指定哪些类应该被排除在扫描范围外。默认为空数组。可以通过`@Filter`注解指定过滤条件。
5. **useDefaultFilters**: 是否启用默认过滤器。默认值为true，表示启用默认过滤器，会扫描所有的组件。如果设置为false，则需要显式地配置包含和排除过滤器。
6. **lazyInit**: 是否启用延迟初始化。默认值为false，表示不启用延迟初始化。如果设置为true，则会延迟初始化扫描到的所有组件。
7. **resourcePattern**: 指定要扫描的资源模式。默认值为"/\*.class"，表示扫描所有的类文件。可以根据需要指定其他模式，比如"/\*.java"。
```java
@Configuration
@ComponentScan(
    basePackages = {"com.example.services", "com.example.controllers"},
    excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
        @Filter(type = FilterType.REGEX, pattern = ".*Test.*")
    },
    useDefaultFilters = false
)
public class AppConfig {
    // 这是一个配置类，用于指定组件扫描的方式和范围
}

```
## @ComponentScan和@EnableAutoConfiguration
1. 在Spring Boot应用程序中，`@ComponentScan`注解通常用于扫描自定义的配置类（例如包含`@Configuration`注解的类），以及其他自定义的组件（例如包含`@Component`, `@Service`, `@Repository`, `@Controller`等注解的类）。这些自定义的配置类和组件通常位于**自己编写的包**中。
2. 对于**starter中的配置类**，通常是通过`@EnableAutoConfiguration`注解来启用的。starter通常会提供一些自动配置类，这些自动配置类会被Spring Boot的`@EnableAutoConfiguration`注解自动扫描并加载。这些自动配置类中通常包含了一些自动配置的逻辑，用于根据项目的依赖和条件自动配置应用程序的一些功能。
3. 在Spring Boot应用程序中，自定义的配置类和组件可以使用@ComponentScan注解来扫描并加载，而starter中的配置类通常是通过@EnableAutoConfiguration注解来启用的，然后通过SpringFactoriesLoader加载自动配置类。这样就能够实现自定义配置和自动配置的组合使用，从而灵活地定制和配置应用程序的功能。
## 自定义starter
1. SpringBoot Starter 类似于一种插件机制，抛弃了之前繁琐的配置，将复杂依赖统一集成进 Starter。所有依赖模块都遵循着约定成俗的默认配置，并允许我们调整这些配置，即遵循“约定大于配置”的理念 。Starter 的出现极大的帮助开发者们从繁琐的框架配置中解放出来，从而更专注于业务代码。
2. Spring 官方提供 Starter 通常命名为 spring-boot-starter-{name} 如：spring-boot-starter-web，spring-boot-starter-activemq 等；Spring 官方建议非官方提供的 Starter 命名应遵守 {name}-spring-boot-starter 的格式：如mybatis-spring-boot-starter。
3. Starter 也是基于 SpringBoot 项目创建的，所以第一步应该先创建 SpringBoot 项目，编写pom文件和需要自动配置的类（用@Configuration和@Bean），Resources 目录下新建 META-INF 文件夹，然后创建 spring.factories 文件
	- 为什么要指定 resources/META-INF 下写 spring.factories？后续补充
4. 完成后通过maven打包测试（可以获取到bean就算成功了）并保存到本地仓库中，后续就可以供自己使用了
5. 可插拔starter虽然引入了Starter Jar 包，但是可以通过条件判断是否加载满足条件的话加载此 Jar 相关配置，不满足就不加载，这可以通过自定义注解+条件注解实现，例如
```java
//自定义注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface EnableAutoConfigTest { }
```
```java
//AutoConfigurationTest 类中添加条件注解，然后重新打包至本地仓库
@Configuration
@ConditionalOnBean(annotation = EnableAutoConfigTest.class)
public class AutoConfigurationTest {
    @Bean
    public ServiceBean getServiceBean() {
        return new ServiceBean();
    }
}
```
```java
//在主程序引用 @EnableAutoConfigTest 注解
@EnableAutoConfigTest
@SpringBootApplication
public class DemoTestSpringBootStarterApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoTestSpringBootStarterApplication.class, args);
    }
}
```
## 为什么一定是spring.factories
`@EnableAutoConfiguration`注解通过读取`META-INF/spring.factories`文件的内容来加载自动配置类。这个过程是由Spring框架的`SpringFactoriesLoader`类实现的。
具体来说，`SpringFactoriesLoader`类提供了一个静态方法`loadFactoryNames()`，这个方法接收一个ClassLoader和一个要加载的工厂类型作为参数，然后返回一个包含工厂名称的列表。在Spring Boot中，`@EnableAutoConfiguration`注解内部就是通过调用`SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, classLoader)`方法来加载自动配置类的。
`SpringFactoriesLoader`在加载`META-INF/spring.factories`文件时，会使用给定的ClassLoader来查找这个文件。一旦找到了`spring.factories`文件，它会读取文件中每个工厂类型对应的配置，并将这些工厂名称加载到一个列表中返回。
总而言之`SpringFactoriesLoader#loadFactories` 负责完成自动装配类的加载，扫描的就是这个变量文件，因此它的名称不可随意更改