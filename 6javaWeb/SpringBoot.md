# 基础篇
Springboot是用来简化spring应用的初始搭建以及开发过程的全新框架

简单对比一下原生开发springmvc程序和直接使用springboot的过程
**最简化版本的原生开发：**
1.引入依赖（servlet、spring-webmvc）
2.web配置类（简化形式）
3.spring配置类（加注解的空白类）
4.提供controller类
**springboot开发：**
1.在idea中直接创建spring initializr工程，依次完成初始化配置和勾选wen并选定springboot的版本
2.提供controller类
也就是可以仅仅完成业务需要的controller类，其它都靠springboot和idea搞定，极大的简化了开发
springboot能实现这样的功能是因为在它的启动依赖中，已经准备好了绝大部分web开发中可能使用到的依赖的版本号，从而达到了减少依赖冲突的目的，如果不想使用其中的依赖，可在pom文件中用exclusion标签排除相关依赖，再重新导入想用的依赖（如更换服务器）
## Springboot项目快速启动
1.对Springboot项目打包，执行maven构建指令package
2.执行启动指令
```
java -jar springboot.jar
```
jar支持命令行启动需要依赖maven插件支持，请确认打包时是否具有SpringBoot对应的maven插件
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```
### 入门案例解析
#### springboot-parent
1.开发SpringBoot程序要继承spring-boot-starter-parent
2.spring-boot-starter-parent中定义了若干个依赖管理
3.继承parent模块可以避免多个依赖使用相同技术时出现依赖版本冲突
4.继承parent的形式也可以采用引入依赖的形式实现效果
#### springboot-starter
SpringBoot中常见项目名称，定义了当前项目使用的所有依赖坐标，以达到减少依赖配置的目的
#### 引导类
![[Pasted image 20230901154642.png]]
SpringBoot的引导类是Boot工程的执行入口，运行main方法就可以启动项目
SpringBoot工程运行后初始化Spring容器，扫描引导类所在包加载bean
#### 内置服务器
springboot的默认内置服务器是tomcat（parent中引入了依赖），如果想要更换为其他服务器，先引入对应的依赖坐标，然后在pom文件中用exclude标签将tomcat的依赖排除
常见的几种服务器对比：
tomcat(默认)    apache出品，粉丝多，应用面广，负载了若干较重的组件
jetty                  更轻量级，负载性能远不及tomcat
undertow          undertow，负载性能勉强跑赢tomcat
## 基础配置
### 三种配置文件格式
application.properties
application.yml
application.yaml
![[Pasted image 20230821162244.png]]
SpringBoot配置文件加载顺序:
application.properties > application.yml > application.yaml

![[Pasted image 20230821162112.png]]
### yaml
yaml 是一种数据序列化格式，优点是便于阅读，容易与脚本语言交互，并且以数据为核心，重数据轻格式，它有.yml和.yaml两种扩展名，其中.yml更为主流和常用
#### 语法规则
1.大小写敏感
2.属性层级关系使用多行描述，每行结尾使用冒号结束
3.使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用Tab键)
4.属性值前面添加空格（属性名与属性值之间使用冒号+空格0作为分隔)
5.# 表示注释
6.数组格式：-space
#### 数据读取三种方式
1.在controller类中声明变量，通过@Value("${含层级关系的变量名或带下标的数组名}")获取
2.定义Environment类型的变量，一次性获取所有数据，并在方法中通过environment.getProperty("含层级关系的变量名或带下标的数组名") 获取
3.如果想要获取某个属性的数据，先定义它的实体类，并添加@Component和@ConfigurationProperties注解，并为该注解的prefix属性赋值yaml中想要获取的属性名，随后在controller类中声明该类的一个对象并用@Autowired自动装配，最后就可以在方法中使用该对象及它的所有数据

## REST风格
### 介绍
REST (Representational State Transfer)，表现形式状态转换
传统风格资源描述形式
http: / / localhost/user/getById?id=1
http:/ / localhost/user/ saveUser
REST风格描述形式
http: / /localhost/user/1
http: / / localhost/user
优点:
隐藏资源的访问行为，无法通过地址得知资源是何种操作,书写简化
![[Pasted image 20230901155211.png]]
上述行为是约定方式，约定不是规范，可以打破，所以称REST风格，而不是REST规范,但一般就按这个写
**描述模块的名称通常使用复数**，也就是加s的格式描述，表示此类资源，而非单个资源，例如:users、books、account...
**根据REST风格对资源进行访问称为RESTful**
### 入门案例
1.利用注解设定http请求动作
![[Pasted image 20230901155458.png]]
2.设定请求参数（路径变量）
![[Pasted image 20230901155602.png]]
用到的注解：
@ RequestMapping 位于springmvc控制器上方，用于设置当前控制器方法请求访问路径
有两个属性，value用于设置请求访问路径，method用于设置http请求动作（GET/POST等）
@ PathVariable 位于springmvc控制器形参定义前面，绑定路径参数与处理器方法形参间的关系，要求路径参数名与形参名一一对应
![[Pasted image 20230901160010.png]]
### RESTful快速开发
1.如果每个方法都用@RequestBody修饰（一般都是这样的），可以直接在类上用@RequestBody修饰，然后@Controller和@RequestBody合并成@RestController
2.直接在类上用@RequestMapping（value）传递路径（要求类中的所有方法都采用这个路径，如果路径后面还有参数，只能省略在类上定义的内容，后面的参数仍要在对应的方法上用@RequestMapping给出）
3.@RequestMapping中的method属性可以用@PostMapping等注解取代（结合2 方法上可能只要一个@PostMapping就够了，其他定义在类上）
# 运维实用篇
## 工程打包与运行
1.对springboot项目打包（执行maven构建指令package）
2.运行项目（执行启动指令）
```
java -jar 工程名.jar
```
![[Pasted image 20230901161809.png]]
### 打包时的注意点
1.先clean
2.注意编码格式是否为UTF-8
（3.有多种配置时将其他的配置文件找地方备份，不要干扰当前要验证的配置文件）
## 配置高级
### 临时属性配置
1.使用jar命令启动springboot工程时可以使用临时属性替换配置文件中的属性
```
java -jar springboot.jar --server.port=80
```
2.带属性参数启动springboot
在开发环境中，有两种办法：
1.通过idea自带功能为命令行参数赋值，这种赋值是idea提供的，与源码无关
2.在启动类中直接为args赋值，这种赋值是源码级的

3.携带多个属性启动springboot，属性间使用空格分离
4.临时属性必须是当前boot工程支持的属性，否则设置无效
5.有多个地方可以配置临时属性，它们的加载有优先级
### 配置文件分类
1．配置文件分为4种
一级：项目类路径配置文件:服务于开发人员本机开发与测试
classpath: application.yml
二级：项目类路径config目录中配置文件:服务于项目经理整体调控
classpath: config/application.yml
三级：工程路径配置文件:服务于运维人员配置涉密线上环境
file : application.yml
四级：工程路径config目录中配置文件:服务于运维经理整体调控
file : config/application.yml 
作用:
三级与四级留做系统打包后设置通用属性
一级与二级用于系统开发阶段设置通用属性
2．多层级配置文件间的属性采用叠加并覆盖的形式作用于程序

### 自定义配置文件
1.使用idea自带功能在命令行参数中为spring.config.name赋值
2.使用idea自带功能在命令行参数中为spring.config.location赋值（全路径）
说明：
单服务器项目:使用自定义配置文件需求较低
多服务器项目:使用自定义配置文件需求较高，将所有配置放置在一个目录中，统一管理
基于SpringCloud技术，所有的服务器将不再设置配置文件，而是通过配置中心进行设定，动态加载配置信息


## 多环境开发配置
### 两个版本的配置
yaml版：
![[Pasted image 20230821163729.png]]
properties版：
![[Pasted image 20230821163758.png]]

### 多环境命令行启动
多环境命令行启动格式：
```
java -jar springboot.jar --spring.profiles.active=test
```

## maven与springboot的多环境兼容
1.maven中设置多环境属性
![[Pasted image 20230821164405.png]]
2.springboot中引用maven属性
![[Pasted image 20230821164440.png]]
3.执行maven打包指令
注意：maven指令执行完毕后，生成了对应的包，其中类参与编译，但是配置文件并没有编译，而是复制到包中，这时要对资源文件开启对默认占位符的解析，加入一个插件
![[Pasted image 20230821164651.png]]
## 日志

日志用于记录开发调试与运维过程消息
1.创建记录日志的对象（导包slt4j）
```
private static final Logger log = LoggerFactory.getLogger(BookController.class);
```
2.在方法中用log对象调用方法
六种日志级别对应的方法名（小写），一般只用中间四种
![[Pasted image 20230901164437.png]]
3.在yml文件中设置日志输出级别
![[Pasted image 20230901164609.png]]
4.设置日志组，控制指定包对应的日志输出级别，也可以直接控制指定包对应的日志输出级别
![[Pasted image 20230901164750.png]]
5.创建日志对象的代码可以省略，在类上加注解@Slf4j（lombok）
6.设置日志输出格式
```
//%d日期，%m：消息，%n换行
logging:
	pattern:
		console:...
```
7.文件记录日志
```
//设置日志文件
logging:
	file:
		name:...

//详细配置
logging:
	file:
		name:...
	logback：
		rollingpolicy:
			max-file-size:3KB
			file-name-pattern:server.%d{yyyy-MM-dd}.%i.log
```

# 开发实用篇
## 热部署
程序中的代码发生改变时，一般需要重新启动服务器才能将改动的代码应用，启动热部署后，可以省去重启这个步骤，自动应用新的代码而不需要其它操作
### 手工启动热部署
开启开发者工具
```
<dependency>
	<groupId>org. springframework. boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```
激活热部署：ctrl+F9

关于热部署：
重启：自定义开发代码，包含类、页面、配置文件等，加载位置restart类加载器
重载：jar包，加载位置base类加载器
也就是说，**热部署仅仅加载当前开发者自定义开发的资源，不加载jar资源**
### 自动启动热部署
1.idea中setting-build-compiler中自动构建项目勾选上
2.Settings中的Advanced Settings中，勾选Allow auto-make to...即可
当前页面不聚焦在idea五秒后，自动进行热部署
### 热部署范围配置
自定义不参与重启的排除项（文件路径）
```
dectools:
	restart:
		exclude:public/**,static/**
```
### 关闭热部署
设置高优先级属性禁用热部署，如果没有被禁用，是因为在更高优先级的配置中开启了热部署
```
public static void main(String[]args) {
	system.setProperty("spring.devtools.restart.enabled" ,"false");
	SpringApplication.run(SSMPApplication.class);
```
## 配置高级
### @ConfigurationProperties
当配置文件中的属性和实体类均存在时，可以为该注解中的prefix属性赋值（字符串表示的配置属性）将实体类（或第三方bean）中的属性与配置文件中的属性对应起来
例如实体类
```
@Component@Data
@ConfigurationProperties(prefix = "servers")
public class servletconfig {
	private string ipAddress;
	private int port;
	private long timeout;
}
```
 中的各属性与配置文件中
```
servers:
	ipAddress:
	port:8080
	timeout:
```
的属性绑定，可以在配置文件中为属性赋值

@EnableConfigurationProperties（配置类名（实体类）的字节码文件）用于总配置类上，可以指定哪些类从配置文件中获取属性，这个注解会自动将其中的类（必须使用@ConfigurationProperties）加入spring容器，因此该类（使用@ConfigurationProperties的类）不能再使用@Component
（也就是说@EnableConfigurationProperties与@Component不能同时使用）

@ConfigurationProperties用于普通配置类（或第三方bean），指定类中的属性与配置文件中的属性对应
@EnableConfigurationProperties用于spring配置类，用于指定哪些类可以从配置类中获取属性（相当于开启了这个功能）
### 松散绑定
![[Pasted image 20230908173735.png]]
但绑定前缀（perfix的值）只能使用纯小写字母，数字和下划线
注意：@Value也可以用于引用单个属性（集合），但它不支持宽松绑定

### 常用计量单位绑定
springboot支持jdk8提供的时间与空间计量单位，用注解@DurationUnit和@DataSizeUnit来指定单位
### 数据校验
1.添加JSR303规范坐标与Hibernate校验框架对应坐标
```
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
</dependency>

<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
</dependency>
```
2.对Bean开启校验功能
在配置类或bean上添加@Validated注解
3.设置校验规则，可以自定义
使用注解@Max等
属性value给定范围，message添加提示信息
### 关于yaml文件中的数字
yaml文件中对于数组的定义支持进制书写格式，如需使用字符串应使用引号明确标注（如数据库密码），否则可能出现意想不到的错误
## 测试
### 加载测试专用属性
1.在启动测试环境时可以通过properties参数设置测试环境专用的属性
```
@SpringBootTest(properties={"test.prop=testValue"})
```
比多环境开发中的测试环境印象范围更小，仅对当前测试类有效
2.还可以通过args参数设置测试环境专用的传入参数
```
@SpringBootTest(args={"--test.prop=testValue"})
```
这是源码级的传入参数
### 加载测试专用配置
测试所用的bean不能定义在main文件下，而应该定义在test文件下，但@SpringBootTest导入配置时仅导入main文件下的配置类，这时需要用@Import（配置类字节码）来追加配置（也就是当前测试类专用的配置）
### web环境模拟测试
#### 测试类中启动web环境
在测试类中启动web环境，用到@SpringBootTest中的属性webEnvironment=SpringBootTest.WebEnvironment.***
有默认端口（配置中指定的）、随机端口、不启动几种选择
#### 发送虚拟请求
![[Pasted image 20230908180707.png]]
1.使用@AutoConfigureMockMvc开启虚拟mvc调用
2.使用@Autowired在形参上注入虚拟调用对象
3.创建虚拟请求
4.执行请求
#### 匹配响应执行状态
![[Pasted image 20230908183643.png]]
1.设定预期值，与真实值进行比较，成功测试通过，失败测试失败
2.定义本次调用的预期值
3.预计本次调用时成功的状态
4.添加预计值到本次调用过程中进行匹配
#### 匹配响应体
![[Pasted image 20230908183908.png]]
基本与上面匹配响应执行状态相同，
执行结果匹配器修改为ContentResultMatchers
预期执行结果修改
#### 匹配虚拟请求体（json）
![[Pasted image 20230908184108.png]]
### 匹配虚拟请求头
![[Pasted image 20230908184210.png]]
### 数据层测试回滚
为测试用例添加事务（@Transctional），springboot会对测试用例对应的事务提交操作进行回滚
如果想在测试用例中提交事务，可以通过@Rollback注解设置为true
### 测试用例数据随机设定
![[Pasted image 20230908184552.png]]
## springboot内置
### 数据源
HikariCP:默认内置数据源对象
Tomcat提供DataSource: HikariCP不可用的情况下，且在web环境中，将使用tomcat服务器配置的数据源对象
Commons DBCP: Hikari不可用，tomcat数据源也不可用，将使用dbcp数据源
配置文件中对数据源进行基本配置后（启动类，url，用户名，密码），可以对指定的数据源进行进一步的配置（通用配置无法设置具体的数据源配置信息，仅提供基本的连接相关配置，如需配置，在下一级配置中设置具体设定）
### 数据层内置持久化解决方案（jdbcTemplate）
![[Pasted image 20230908185257.png]]
需要导入依赖(mybatis-plus包含它)
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</ artifactId>
</ dependency>
```
### H2数据库
导入依赖
```
<dependency>
	<groupId>com.h2databasec/groupId>
	<artifactId>h2</artifactId>
</dependency>

```
设置当前项目为web工程，并配置H2管理控制台参数
```
spring:
	h2:
		console:
			path:/h2
			enabled:true
```
H2的操作界面在浏览器中输入指定路径localhost/path指定的路径
用户名sa，默认密码123456
H2数据库控制台仅用于开发阶段，线上项目务必关闭控制台功能（enable赋值为false）


# springboot整合
## juint
![[Pasted image 20230821165148.png]]
只需要在测试类上添加注解@SpringBootTest，相关属性classes可以设置SpringBoot启动类，如果测试类在SpringBoot启动类的包或子包中，可以省略启动类的设置，也就是省略classes的设定，一般都可以省略
## Mybatis
![[Pasted image 20230821165436.png]]
1.创建新模块，选择Spring初始化，并配置模块相关基础信息（idea）
2.选择当前模块需要使用的技术集（Mybatis、MySQL）（idea）
3.设置数据源参数
![[Pasted image 20230821165635.png]]
4.定义数据层接口与映射配置
![[Pasted image 20230821165711.png]]
5.测试类中注入dao接口，测试功能
![[Pasted image 20230821165734.png]]
# 原理篇
## bean的加载方式总结
### XML方式声明bean
![[Pasted image 20230908193112.png]]
所有bean定义一目了然，但声明麻烦
### XML+注解定义bean
注解：
1.使用@Component及其衍生注解@Controller、@Service、@Repository，如果是在配置类上声明@Component，更建议使用@Configuration
2.使用@Bean定义第三方bean，并将所在类定义为配置类或Bean

在xml文件中声明要扫描的包：
![[Pasted image 20230908194422.png]]
说明：
要使用标签<context:component-scan>，必须先添加一个命名空间，如如红色字体context的五个地方要修改

由此可以得出@Component和@Bean虽然都是用于定义bean，但仍有区别：
1.用途不同
@Component用于标识普通的类
@Bean是在配置类中声明和配置Bean对象
2.使用方式不同
@Component是一个类级别的注解，spring通过@Component注解扫描并注册为Bean
@Bean通过方法级别的注解使用，在配置类中手动声明和配置Bean
3.控制权不同
@Component注解修饰的类是由spring来创建和初始化的
@Bean注解允许开发人员手动控制Bean的创建和配置过程
### 配置类（纯注解）
将上述xml文件中的扫描语句简化掉（不需要xml文件了）
xml+注解将bean的定义简化为注解
定义配置类加上@ComponentScan（要扫描的包名）
自定义的bean放在要扫描的包中，而第三方bean则可以在这个配置类中直接用@Bean定义
![[Pasted image 20230908195051.png]]
#### FactoryBean
初始化实现FactoryBean接口的类，实现对bean加载到容器之前的批处理操作
![[Pasted image 20230908195309.png]]
注意下面的@Bean虽然返回值是BookFactoryBean，但仍然是将book对象作为bean返回而不是工厂，工厂类中还有一个函数isSingleton可以用于指定bean对象是否是单例模式
#### ImportResource（....xml）
这个注解可以导入已经用xml定义的bean对象，而不影响现有的程序或xml文件，是用来适配早期bean声明方法的
直接在配置类中上使用这个注解，可以实现纯xml文件与纯注解的兼容
#### proxyBeanMethod
是@Configuration的一个属性，默认值是true，此时相当于该类下的bean是单例的
这个单例对象是指bean对象的创建是从容器中获取的一个代理对象，无论调用多少次方法，获取的都是同一个对象
而false则是通过@Bean修饰的方法每次都重新获取一个新的对象
### @Import（....class数组）
使用@Import注解在配置类上导入要注入的bean对应的字节码
被导入的bean无需使用注解声明为bean，也就是任意想要声明为bean的类不需要任何操作，仅在spring配置类上添加注解即可声明为bean
这样声明的bean的名字是类的全类名
此方式可以有效的降低源代码与spring技术的耦合度

注意，当某个配置类按此方式声明为bean时（一般都是为了将其内部用@Bean修饰的方法的返回值声明为bean），即使没有@Configuration注解，仍会将改配置类连同其内部的@Bean方法的返回值一并声明为bean（配置类本身也会声明成bean）
### 编程声明bean
使用上下文对象在容器初始化完毕后注入bean
只能用AnnotationConfigApplicationContext容器调用registerBean方法（形参是所要声明的bean的实体类的字节码）直接声明
可以指定该bean的名称，类别（已经有对应的实体类），可变参数（bean对象的属性）（后声明的覆盖前声明的）
如果使用register方法，则是仅添加了一个对应类的bean，而没有为属性赋值
![[Pasted image 20230912112440.png]]
### ImportSelctor
通过@Import在配置类上导入实现了ImportSelector接口的类实现对**导入源**（前面所指的配置类）的编程式处理
实现了ImportSelector接口的类中要实现一个方法，通过这个方法的形参可以获取关于配置类（实体类）的许多信息，通过这些信息可以选择性的加载某些bean
![[Pasted image 20230912120045.png]]
谁导入这个类（MyImportSelector）就可以通过metadata获取谁的一系列信息并进行后续操作，进行判断以选择要不要将它作为bean加载
### ImportBeanDefinitionRegistrar
通过@Import在配置类上导入实现了ImportBeanDefinitionRegistrar接口的类，通过BeanDefinition的注册器注册实名bean，实现对容器中bean的裁定，例如对现有bean的覆盖，进而达成不修改源代码的情况下更换实现的效果
类似于ImportSelect，但是增加了一些对bean的修改功能，通过beandefintion可以实现对bean的操作，操作完成后直接将这个bean加载到容器中
![[Pasted image 20230912120745.png]]
### BeanDefinitionRegistryPostProcessor
通过@Import在配置类上导入实现了BeanDefinitionRegistryPostProcessor接口的类，通过BeanDefinition的注册器注册实名bean,实现对容器中bean的最终裁定
同名bean不一定是同一个bean，bean还有类型
这个接口是为了解决容器中存在多个同名bean先后加载到底最终用谁的，一般来说，后加载的会覆盖前加载的
但这个接口的实现类的机制是最后加载，因此会覆盖前面所有的同名bean，起到一锤定音的作用
![[Pasted image 20230912121629.png]]
### bean加载方式总结
![[Pasted image 20230912121743.png]]
## bean加载控制
### 编程式控制
在以上八种bean的加载方式中，后四种bean加载时都是通过registry调用方法在程序中显式加载的（人为加载的），因此可以进行加载控制，以下是一个实例，如果容器中有老鼠，则加载猫
![[Pasted image 20230912122420.png]]
### 注解式控制
上述方法每有一个bean需要加载控制都需要一些代码，比较麻烦
spring提供了注解@Conditional来帮助实现bean的加载控制，但是需要自己实现condition接口
springboot帮我们解决了这些问题，提前写好了一大堆condition接口的实现类
我们使用时直接在bean上添加注解即可，一般注解的形式是@ConditionalOn...
根据要求选择合适的注解，并在注解后面添加合适的参数（如字节码后字符串等）
这些注解可以同时使用，满足所有条件bean才会被加载
通过这种方式，可以利用一些环境上的特征去加载某个bean，当不需要时就不加载
例如，当程序中没有数据库驱动类时，数据源bean也就不需要加载
![[Pasted image 20230912123453.png]]
## bean依赖属性配置
将配置类的属性抽取成配置文件，需要修改时直接在配置文件中修改，而不需要修改任何代码
这个过程中包含以下组件：
配置文件：为实体类中的各属性赋值
实体(配置)类：通过@ComfigurationProperties（prefix）获取配置文件中的值
业务逻辑类：通过@EnableComfigurationProperties（实体类字节码）基于实体类的值实现业务逻辑，利用三目运算符为实体类中未赋值的变量给定默认值
引用业务逻辑的类：使用@Import导入业务逻辑类字节码，以引用业务逻辑
上述流程实现了在不引用业务逻辑时实体类，业务逻辑类均不会注册为bean，只有在被import导入时，才会一并加载为bean，同时，只需在配置文件中修改变量的值，而不需要修改任何代码，就能实现不同的功能（业务逻辑类中需提前对该值有相应的功能方法）
## 自动配置原理
![[Pasted image 20230912124901.png]]
当我们的SpringBoot项目启动的时候，会先导入AutoConfigurationImportSelector，这个类会帮我们选择所有候选的配置，我们需要导入的配置都是SpringBoot帮我们写好的一个一个的配置类，那么这些配置类的位置，存在于META-INF/spring.factories文件中，通过这个文件，Spring可以找到这些配置类的位置，于是去加载其中的配置。
但是不是所有的配置都会被加载，而是通过注解@ConditionalOnClass的判断，如果注解中的类都存在，才会进行加载。
![[Pasted image 20230912125808.png]]
基于以上原理，我们可以自定义自动配置
要删除某些文件的自动配置，一可以通过配置文件中使用\<exclude>标签（pom）或者spring.autoconfig.exclude（yml），二可以通过注解@EnableAutoConfiguration的属性排除自动配置项
要添加自定义的自动配置，在spring.factories中添加需要自动配置的类（即使不improt也能直接使用业务逻辑）