# Spring
## 分层解耦（IOC/DI）
内聚：软件中各个功能模块内部的功能练习
耦合：衡量软件中各个层或模块之间的依赖、关联的程度
程序的设计原则：高内聚，低耦合
### 三层架构
controll：控制层，接收前端发送的请求，对请求进行处理，并响应数据
service：业务逻辑层，处理具体的业务逻辑
dao：数据访问层（持久层），负责数据访问操作，包括数据的增删改查
实现三层架构可以创建三个包中分别放对应层的接口，这样可以使代码的复用性高，维护容易，但由于各层之间存在互相创建调用对象，仍避免不了耦合的情况
```
@RestController//包含@Controller和@ResponseBody
public class Empcontroller {
	private Empservice empService=new EmpService() ;
	@RequestMapping("/listEmp")
	public Result list() throws Exception {
	List<Emp> empList = empService.listEmp();
	return Result.success(empList);
```
```
public class EmpServiceA implements Empservice {
	private EmpDao empDao =new EmpDao();
	public List<Emp> listEmp(){
	//调用dao层，查询数据
	List<Emp> empList = empDao.listEmp();
	//···
```
```
public class EmpDaoA implements EmpDao {
	public List<Emp> listEmp(){
		//1.从文件中查询数据
		String file = this.getClass().getclassLoader().getResource("emp.xml").getFile();
		List<Emp> empList = XmlParserUtils.parse(file,Emp.class);
		return empList;

```
### IOC和DI
控制反转IOC：inversion of control，对象的控制权由程序自身转移到外部（容器）
依赖注入DI：dependency injection，容器为应用程序注入运行时所依赖的资源
Bean对象：IOC容器中创建、管理的对象，称之为bean
#### 解耦实现
**service层及dao层的实现类，交给IOC容器管理**
1.在实现类的上面加上注解@Component，不用时将注解注释掉即可；
2.@Component有三个衍生注解@Controller、@Service、@Repository分别标注在控制器类、业务类、数据访问类上，不属于以上三类时，才使用@Component；
3.标记在实现类上方的注解，可用于为IOC容器中对应类的bean赋名字，不赋名字时默认为首字母小写的类名；
4.上述注解要生效，需要被组件扫描注解@ComponentScan扫描，但是在启动类注解@SpringBootApplication中已经包含了，默认扫描的范围是启动类所在包及其子包，如果不在其中需要手动添加组件扫描注解@ConponentScan
**为controll及service注入运行时依赖的对象**
1.声明对象变量时不直接赋值，而是在上面加上注解@Autowired；
2.@Autowired默认按照类型进行，如果存在多个相同类型的bean，会报错；
3.解决方案：@Primary，在实现类上添加，是默认使用的bean
@Qualifier，在声明对象变量时添加，后面指定使用的bean的名字
@Resource，代替@Autowired，默认按照名字进行
**@Autowired和@Resource的区别：**
@Autowired 是spring框架提供的注解，而@Resource是JDK提供的注解；@Autowired 默认是按照类型注入，而@Resource默认是按照名称注入。
**运行测试**

## 事务
### 概念和基本使用
事务是一组操作的集合，它时一个不可分割的工作单位，这些操作要么同时成功，要么同时失败
开启事务（一组操作开始前，开启事务）：start transaction / begin;
提交事务（这组操作全部成功后，提交事务）：commit;
回滚事务（中间任何一个操作出现异常，回滚事务）：rollback;
### spring提供的方法
在业务(service）层的方法上、类上、接口上，添加注解@Transactional
将当前方法交给spring进行事务管理，方法执行前，开启事务;成功执行完毕，提交事务;出现异常，回滚事务

```
#开启事务管理日志
logging:
	level:
	org.springframework.jdbc.support.JdbcTransactionManager: debug

```
#### 事务属性
##### rollbackFor
默认情况下，只有出现RuntimeException才回滚异常，rollbackFor属性用于控制出现何种异常类型，回滚事务
rollbackFor = Exception.class就可以针对所有异常
##### propagation
事务属性-传播行为：当一个事务被另一个事务方法调用时，这个事务应该如何进行事务控制
![[Pasted image 20230804164801.png]]
REQUIRED∶大部分情况下都是用该传播行为即可。
REQUIRES_NEW:当我们不希望事务之间相互影响时，可以使用该传播行为。比如:下订单前需要记录日志，不论订单保存成功与否，都需要保证日志记录能够记录成功。

## AOP
Aspect Oriented Programming（面向切面编程），即面向特定方法编程
动态代理是其最主流的实现，旨在管理bean对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程，可以达到不修改原本代码而增添其他功能的效果
### 快速入门
1.导入依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
2.编写AOP程序，针对于特定方法根据业务需要进行编程
### 核心概念
连接点：JoinPoint，可以被AOP控制的方法，暗含方法执行时的相关信息
通知：Advice，指重复的逻辑，也就是共性的功能（体现为一个方法）
切入点：PointCut，匹配连接点的条件，通知仅会在切入点方法执行时被应用
切面：Aspect，描述通知与切入点的对应关系（通知+切入点)
目标对象：Target，通知所应用的对象
```
//通知
@Aspect
public class TimeAspect {
	//切入点：包com.itheima.service.impl下的方法执行以下逻辑
	@Around("execution(* com.itheima.service.impl.*.*(..))")
	public object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
		long begin = System.currentTimeMillis();
		Object result = joinPoint.proceed();//调用原始操作
		long end = System.currentTimeMillis();
		log.info("执行耗时:ms", (end-begin));
		return result;
	}
}
```
### 通知类型
@Around:环绕通知，此注解标注的通知方法在目标方法前、后都被执行
@Before:前置通知，此注解标注的通知方法在目标方法前被执行
@After:后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行@AfterReturning : 返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行@AfterThrowing :异常后通知，此注解标注的通知方法发生异常后执行

@PointCut：该注解的作用是将公共的切点表达式抽取出来，需要用到时引用该切点表达式即可。
```
@Pointcut ("execution(* com.itheima.service.impl. DeptServiceImpl.* (..)) ")
//private:仅能在当前切面类中引用该表达式
//public:在其他外部的切面类中也可以引用该表达式
public void pt(){}

@Around ("pt()")
public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
```
### 通知顺序
当有多个切面的切入点都匹配到了目标方法，目标方法运行时，多个通知方法都会被执行
此时通知的执行顺序如下，建议以实际运行为准
1.不同切面类中，默认按照切面类的类名排序，在原始方法前运行的通知，类名排名越前，越先运行；在原始方法后运行的通知，类名排名越前，越后运行；
2.用@Order（int）加在切面类上来控制顺序，目标方法前的通知方法，数字小的先执行；目标方法后的通知方法，数字小的后执行
### 切入点表达式
是描述切入点方法的一种表达式，主要用来决定项目中的哪些方法需要加入通知
1.execution（string）：根据方法的签名来匹配
2.@annotation（string）：根据注解匹配
```
@Before ("execution (public void com.itheima.service.impl.DeptServiceImp1.delete(java.lang.Integer))")
public void before (JoinPoint joinPoint) {
--------------------------------------------------------
@Before ( "@annotation (com.itheima.anno.Log)")
public void before() {
```
#### execution
![[Pasted image 20230804210459.png]]
![[Pasted image 20230804210557.png]]
书写建议:
1.所有业务方法名在命名时尽量规范，方便切入点表达式快速匹配。如:查询类方法都是find开头，更新类方法都是update开头。
2.描述切入点方法通常基于接口描述，而不是直接描述实现类，增强拓展性。
3.在满足业务需要的前提下，尽量缩小切入点的匹配范围。如:包名匹配尽量不使用..，使用* 匹配单个包。
#### @annotation
![[Pasted image 20230804210904.png]]
这里所说的特定注解一般是自定义注解，是用作标记的，可以没有属性，但需要@Retention（）和@Target（）两个注解修饰一下
@annotation（）括号中填入这个自定义注解的全名

### 连接点
在Spring中用JoinPoint抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等
对于@Around通知，获取连接点信息只能使用ProceedingJoinPoint；其它四种通知，获取连接点信息只能使用JoinPoint，他是ProceedingJoinPoint的父类型
![[Pasted image 20230807152339.png]]
![[Pasted image 20230807152358.png]]


**梳理bean/IOC/DI
最终目的是实现程序的解耦，
1.将各层的对象封装到bean中并统一交给IOC容器（spring）后台管理（这一步在springboot中只需要通过注解即可实现）
2.需要使用某一层的对象时（一般是需要调用某个方法实现某种功能），实现bean的实例化获取对象,但此时的bean对象不能立即被使用
3.通过某种注入方式为bean的实例化对象注入所需的数据或其它的bean（依赖项），这个过程被称为依赖注入，完成依赖注入后，才能确保bean被正确使用，依赖注入前的bean对象可能是不完整的**
# Bean
用来定义Spring核心容器管理的对象
在spring中，将某个类的对象交由IOC容器统一管理，在需要使用时，可以通过多种方式注入该对象的实例（所以得先实例化），如setter注入，构造器注入，自动装配，集合注入
但需要注意的是，这时的bean一般默认是单例的，这是因为一般bean对象重复使用并不会对业务造成影响
适合交给容器进行管理的bean，表现层对象，业务层对象，数据层对象，工具对象，这些对象即使重复使用也一般不会影响到程序正常运行，但如果是封装实体的域对象，则不适合交给容器管理
![[Pasted image 20230819152308.png]]
## Bean的实例化
### 构造方法
bean本质上就是对象，创建bean使用构造方法完成
![[Pasted image 20230819130147.png]]
### 静态工厂
通过静态工厂实现bean的实例化，一般是为了兼容早期的设计模式，可以在工厂中执行其它语句
![[Pasted image 20230819130443.png]]
### 实例工厂
先造实例工厂对象，再用实例工厂中的方法得到bean的实例化对象
因此在配置时还需要添加一个实例工厂的bean，并且在需要实例化的bean的配置栏中加入实例工厂的bean的id
![[Pasted image 20230819130815.png]]
这种方法由于实例工厂bean的id是自己定义，自己使用的，不是很方便，因此有改良
在创建实例工厂类时，可以直接实现Factory<对象名>接口（对象名指的是你想用这个工厂造什么对象，也就是最开始想要实例化的bean对象），并实现接口中的抽象方法，这几个抽象方法分别替代了原来的构造方法，指定了对象的类型（就是刚才泛型中的对象名），还可以选择是否是单例对象（isSingleton方法可以不重写）
![[Pasted image 20230819131550.png]]
## Bean的生命周期
生命周期︰从创建到消亡的完整过程
bean生命周期: bean从创建到销毁的整体过程
bean生命周期控制︰在bean创建后到销毁前做一些事情
生命周期控制有两种方法，一种是直接在bean的配置中加入init-method和deatory-method
另一种是通过实现接口来控制
![[Pasted image 20230819131913.png]]
![[Pasted image 20230819131921.png]]
bean的生命周期包括
初始化容器（创建对象内存分配，执行构造方法，执行属性注入set操作，执行自定义bean初始化方法）
使用bean（执行业务操作）
关闭或销毁容器（执行bean销毁方法）

补充：容器关闭前触发bean的销毁
其中关闭容器有两种方式
手工关闭容器：通过ComfigurableApplicationContext接口中的close（）操作
注册关闭钩子，在虚拟机退出前先关闭容器再退出虚拟机：ComfigurableApplicationContext接口中的registerShutdownHook（）操作
```
public class AppForLifeCycle {
	public static void main(String[]args) {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext( "applicationContext.xml");
	ctx.close();
	}
}
```
## 依赖注入
bean的本质也是一个类，实例化中有时它也会依赖外部提供的数据运行，而一般向一个类中传递数据有两种方式，普通方法（set方法）和构造方法，传递的数据也可以分为引用数据类型和基本数据类型
![[Pasted image 20230819152400.png]]
### setter注入
基本数据类型
```
//在bean中定义基本类型属性并提供可访问的set方法
public class BookDaoImpl implements BookDao {
	private int connectionNumber;
	public void setConnectionNumber(int connectionNumber) {
		this.connectionNumber = connectionNumber;
	}
}
```
```
//配置中使用property标签value属性注入简单类型数据
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
	<property name="connectionNumber" value="10"/>
</bean>
```
引用数据类型
```
//在bean中定义引用类型属性并提供可访问的set方法
public class BookServiceImpl implements BookService{
	private BookDao bookDao;
	public void setBookDao(BookDao bookDao) {
		this.bookDao = bookDao;
	}
}
```
```
//配置中使用property标签ref属性注入引用类型对象
<bean id="bookService" class="com.itheima.service.impl.BookServiceImpl">
	<property name="bookDao" ref="bookDao" />
</bean>
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl" />
```
### 构造器注入
构造器注入与setter注入基本一致，只是在配置bean时将property标签换成constructor-arg
由此引出的构造方法名紧耦合问题，可通过其它方式解决，如去除方法名，将参数的赋值改为使用type按形参属性注入或使用index按形参顺序注入
#### 注入方式小结
setter注入：property，基本value，引用ref
构造器注入：constructor-arg，基本value，引用ref
#### 注入方式选择
1.强制依赖（bean必须使用的依赖）使用构造器进行，使用setter注入有概率不进行注入导致null对象出现
2.可选依赖使用setter注入进行，灵活性强
3.Spring框架倡导使用构造器，第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
4.如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
5.实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
6.自己开发的模块推荐使用setter注入
### 自动装配
IOC容器根据bean所依赖的资源在容器中自动查找（也就是容器中必须已经有相关的bean）并注入到bean中的过程称为自动装配，它有以下几种方式：按类型，按名称，按构造方法，或不启用自动装配
使用方式是在bean的配置项中添加属性autowire，并指定自动装配方式

特征：
1.自动装配用于引用类型依赖注入，不能对简单类型进行操作（因为IOC容器中一般不存在基本数据类型的bean，这时可以直接通过setter或构造器注入）
2.使用按类型装配时( byType )必须保障容器中相同类型的bean唯一，推荐使用
3.使用按名称装配时( byName )必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐使用4.自动装配优先级低于setter注入与构造器注入，同时出现时自动装配配置失效
### 集合注入
同时注入多个同种类型的bean（一般是基本数据类型）数组，List,Set，Map，Properties
![[Pasted image 20230819144454.png]]
## 加载properties文件
![[Pasted image 20230819150932.png]]
![[Pasted image 20230819151023.png]]
## 容器的一些补充知识
1.创建容器的两种方法
classPathXmlApplicationcontext
```
ApplicationContext ctx = new clalssPath×m1ApplicationContext("applicationContext.xm1");
BookDao bookDao = (BookDao) ctx.getBean("bookDao");
bookDao.save();
```
FileSystemXmlApplicationContext
```
ApplicationContext ctx = new FilePath×m1ApplicationContext(绝对路径);
BookDao bookDao = (BookDao) ctx.getBean("bookDao");
bookDao.save();
```
2．获取bean(3种)
方式一︰使用bean名称获取
BookDao bookDao = (BookDao) ctx.getBean( "bookDao");
方式二︰使用bean名称获取并指定类型
BookDao bookDao = ctx.getBean( "bookDao"，BookDao.class);
方式三︰使用bean类型获取
BookDao bookDao = ctx.getBean(BookDao.class);

3.容器类层次结构
![[Pasted image 20230819151913.png]]
BeanFactory是IOC容器的顶层接口，初始化BeanFactory对象时，加载的bean延迟加载
ApplicationContext接口是spring容器的核心接口，初始化时bean立即加载
ApplicationContext接口提供基础的bean操作相关方法，通过其他接口扩展其功能
ApplicationContext接口常用初始化类包括ClassPathXmlApplicationContext和FileSystemXmlApplicationContext

4.BeanFactory
```
//beanfactory初始化
//类路径加载配置文件
Resource resources = new classPathResource("applicationContext.xml");
BeanFactory bf = new XmlBeanFactory(resources);
BookDao bookDao = bf.getBean("bookDao" , BookDao.class);
bookDao.save();
```
beanfactory创建完毕后，所有的bean均为延迟加载
# 注解开发
## 定义bean
```
//使用@Component定义bean
@Component("bookDao")
public class BookDaoImpl implements BookDao {}
@Component
public class BookServiceImpl implements BookService {}
//@Component提供的三个衍生注解
@Controller:用于表现层bean定义
@Service :用于业务层bean定义
@Repository :用于数据层bean定义


//核心配置文件中通过组件扫描加载bean
< context : component-scan base-package="com.itheima" />
```
## 纯注解开发模式
用一个配置类替代xml文件
1.定义一个配置类（用注解@Configuration声明）
2.用注解@ComponentScan（"包名"）扫描包中所有用@Component声明的bean
3.此时不再加载配置文件，而应该改为加载配置类
```
Applicationcontext ctx = new AnnotationconfigApplicationContext(SpringConfig.class);
```
## 注解开发中bean的作用范围与声明周期
作用范围：@Scope（指定单例还是非单例）
生命周期：
@PostContrust 构造方法后执行的方法
@PreDestory 销毁前执行的方法
## 注解开发中的依赖注入
在声明对象时在上方加上注解@AutoWired，自动装配bean对象
注意︰
1.自动装配基于反射设计创建对象并暴力反射对应属性为私有属性初始化数据，因此无需提供setter方法
2.自动装配建议使用无参构造方法创建对象（默认），如果不提供对应构造方法，请提供唯一的构造方法
3.如果对应类型的bean不是唯一的，会报错，此时通过注解@Qualifier（）来指定名称，它必须配合@AutoWired使用
4.自动装配简单数据类型时，通过注解@Value（）直接为其赋值（可以用${ }从配置文件中传值过来，但要在配置类中加上@PropertySource（配置文件名.properties））
## 注解开发管理第三方bean
```
//以管理druid数据连接池对象为例
//1.定义一个方法获得要管理的对象
//2.添加@Bean，表示当前方法的返回值是一个bean
@Bean//表示当前方法的返回值是一个bean
public DataSource dataSource(){
	DruidDataSource ds = new DruidDataSource();
	//手动写入所需的各项参数
	ds.setDriverclassName("com.mysq1.jdbc.Driver");
	ds.setUrl("jdbc : mysql://localhost: 3306/spring_db");
	ds.setUsername("root");
	ds.setPassword("root");
	return ds;
}
```
以上可以直接放在SpringConfig类中，但一般不这么做，因为当引入的第三方bean越来越多时会使这个配置类中非常杂乱，一般会重新定义一个配置类如jdbcConfig（名字随意，用注解@configurtion标记即可），再在SpringConfig类上用注解@ComponentScan扫描jdbaConfig所在的包（一般就是config包），也可以用注解@import导入该配置类名（@Import（{ jdbcConfig.class } ）），后者更为直观，也更为常用

如果第三方bean也需要依赖注入，如上述例子中name，url等参数应该通过配置文件注入的
基本数据类型，在配置类中声明成员变量 然后用@Vaule注入
引用数据类型，为bean的定义方法（被@bean注解的方法）设置形参，容器会自动根据类型装配对象
## 注解开发总结（xml配置对比注解配置）
![[Pasted image 20230819160430.png]]

# SpringMVC(待补充)
是一种表现层开发技术，是一种基于Java实现MVC模型的轻量级web框架，使用简单，开发便捷，灵活性强
## 入门案例
1.使用SpringMVC技术需要先导入SpringMVC坐标与Servlet坐标
![[Pasted image 20230807153225.png]]
2.创建SpringMVC控制器类
```
@Controller
public class UserController {
	//指定请求路径
	@RequestMapping("/save")
	//设置当前操作的返回值类型就是方法的返回值类型
	@ResponseBody
	public String save(){
		system.out.println( "user save ...");
		return "{ 'info ' : ' springmvc '}";
	}
}
```
3.初始化SpringMVC环境，设定SpringMVC加载对应的bean
```
@Configuration
@ComporntScan("com.itheima.controller"(包名))
public class SpringMvcConfig{
}
```
4.初始化Sevelet容器，加载SpringMVC环境，并设置SpringMVC技术处理的请求
![[Pasted image 20230807153740.png]]
其中第一个方法中初始化了一个web专用的ApplicationContext，将SpringMvcConfig注册进去
第二个方法设置所有的请求都交由springmvc处理
### 注解
#### @Controller
![[Pasted image 20230807154459.png]]
#### @ReqestMapping
![[Pasted image 20230807154512.png]]
#### @ResponseBody
![[Pasted image 20230807154617.png]]
### 总结
一次性工作：
创建工程（Maven），设置服务器（Tomcat插件），加载工程
导入mvc和servlet的坐标
创建web容器的启动类，加载SpringMVC配置，并设置SpringMVC请求拦截路径
SpringMVC核心配置类（设置配置类，扫描controller包，加载Controller控制器bean）
多次工作：
定义处理请求的控制器类
定义处理请求的控制器方法，并配置映射路径（@RequestMapping）与返回json数据（@ResponBody）
### 工作流程分析
启动服务器初始化过程
1．服务器启动，执行ServletContainersInitConfig类，初始化web容器
2．执行createServletApplicationContext方法，创建了webApplicationContext对象
3．加载SpringMvcConfig
4．执行@ComponentScan加载对应的bean
5．加载UserController，每个@RequestMapping的名称对应一个具体的方法
6．执行getServletMappings方法，定义所有的请求都通过SpringMVC
单次请求过程
1.发送请求localhost/save
2.web容器发现所有请求都经过SpringMVC，将请求交给springMVC处理
3．解析请求路径/save
4．由/save匹配执行对应的方法save()
5．执行save()
6．检测到有@ResponseBody直接将save()方法的返回值作为响应求体返回给请求方

## bean的加载控制
由于spring和springmvc各自有自己需要管理的bean，因此常常需要对这些bean的加载进行控制，防止spring加载到springmvc的bean，出现不必要的麻烦（如命名冲突，内存占用，性能降低等）

springmvc相关的bean为表现层bean，通常由注解@Controller修饰；spring控制的bean包括业务层bean（通常由注解@Service修饰）和功能bean（如DataSource等）
常见的加载控制方法有：
方式一: Spring加载的bean设定扫描范围为com.itheima，排除掉controller包内的bean
```
@ComponentScan(value="com.itheima",
	excludeFilters = @ComponentScan.Filter(
		type = FilterType.ANNOTATION,
		classes = controller.class
	)
)

excludeFilters:排除扫描路径中加载的bean，需要指定类别(type)与具体项(classes)
includeFilters:加载指定的bean，需要指定类别(type)与具体项(classes)

```
方式二: Spring加载的bean设定扫描范围为精准范围，例如service包、dao包等
方式三:不区分Spring与springMVvC的环境，加载到同一个环境中

简化开发的方法是设置servlet配置类中时使用简化开发格式
```
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherservletInitializer{
	protected class<?>[] getServletConfigclasses() {
		return new Class[]{SpringMvcConfig.class};
	}
	protected string[] getServletMappings() {
		return new String[]{"/"};
	}
	protected class<?>[] getRootConfigClasses () {
		return new Class[]{SpringConfig.class};
	}
}
```

# Mybatis
MyBayis是一款优秀的持久层框架，用于简化JDBC的开发
## 一些前置知识
### JDBC
是使用java语言操作关系型数据库的一套API,各个数据库厂商去实现这套接口，提供数据库驱动jar包,我们可以使用这套接口(JDBC）编程，真正执行的代码是驱动jar包中的实现类。
```
@Test
public void testJdbc ()throws Exception {
	//1.注册驱动
	Class.forName ("com.mysql.cj.jdbc. Driver");
	//2．获取连接对象
	String url = "jdbc:mysql://localhost:3306/mybatis";
	String username = "root";
	String password = "1234";
	Connection connection = DriverManager. getConnection(url，username，password);
	//3.获取执行SQL的对象Statement,执行SQL,返回结果
	String sql = "select * from user";
	Statement statement = connection.createstatement () ;
	Resultset resultset = statement.executeQuery (sql);
	//4.封装结果数据
	List<User> userList = new ArrayList<> ();
	while (resultset.next()){
		int id = resultset-getInt ("id");
		string name = resultset.getstring ("name");
		short age = resultset.getShort("age") ;
		short gender = resultset.getShort ("gender") ;
		string phone = resultset.getstring ("phone");
		User user = new User(id, name , age, gender,phone);
		userList.add (user);
	}
	//5.释放资源
	statement.close ();
	connection.close ();
	userList.stream ().forEach(user -> {
		System.out.println (user);
	});
}
```
上述代码中含有较多硬编码，且比较繁琐，造成资源浪费和性能降低
MyBatis中将数据库的连接信息专门放在一个文件中，便于修改，并且使用在方法上添加注解的方法使SQL语言便于复用和识别
### 数据库连接池
1.数据库连接池是个容器，负责分配、管理数据库连接(Connection)
2.它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个
3.释放空闲时间超过最大空闲时间的连接，来避免因为没有释放连接而引起的数据库连接遗漏
优点：资源重用、提升系统响应速度、避免数据库连接遗漏
idea自带默认连接池追光者，需要更换时在pom文件中引入相关连接池的依赖即可
### Lombok
Lombok是一个实用的Java类库，能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，并可以自动化生成日志变量，简化java开发、提高效率。
![[Pasted image 20230801134347.png]]
使用方法：引入依赖
```
<dependency>
	<groupld>org.projectlombok</groupld>
	<artifactld>lombok<lartifactld>
</dependency>
```
## 基础操作
### 准备工作
1.准备数据库表emp
2.创建一个新的springboot工程，选择引入对应的起步依赖（ mybatis、mysql)
3.application.properties中引入数据库连接信息
4.创建对应的实体类Emp(实体类属性采用驼峰命名)
5.准备Mapper接口EmpMapper,后续可以在接口中创建sql语句对应的方法
### 删除
```
//如果mapper接口方法形参中只有一个普通类型的参数，#{···}中的属性名随意
//预编译时#{}(参数占位符)会被替换成？，这使得性能更高且可以防止SQL注入
//参数传递时使用#{}，对表明、列表进行动态设置时使用${}
//SQL注入是通过操作输入的数据来修改事先定义好的SQL语句，以达到执行代码对服务器进行攻击的方法。

@Delete("delete from emp where id=#{id}")
public void delete(Integer id)
```
### 添加
```
//参数很多时可以用实体类封装起来传递实体类
//#{}中是属性名，采用驼峰命名，不同于字段命名采用下划线
//true表示需要返回主键值，id表示返回的主键值存入哪个属性

@Options(keyProperty ="id",useGeneratedKeys = true)
@Insert(sql)
public void insert(Emp emp);
```

### 更新
```
@Update(sql)
public void update(Emp emp);
```
### 查询
#### 数据封装
在传递数据时，会传递数据的包装类对象，但在sql语句和实体类中对象的命名规则不同，要对应起来需要数据封装
有三种办法：
1.在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样
```
@Select("select id,username, password,name, gendr, image,jab,entrydate, dept_id deptld, create_time createTime, update_timeupdateTime from emp where id = #{id} ")
public Emp getByld(Integer id);
```
2.通过@Results和@Result进行手动结果映射
```
@Select("select * from emp where id = #{id}")
@Results({
	@Result(column = "dept_id", property = "deptld"),
	@Result(column = "create_time", property = "createTime"),
	@Result(column = "update_time", property = "updateTime")})
public Emp getByld(Integer id);
```
3.如果字段名与属性名严格符合托风格命名和下划线命名，mybatis可以自动通过驼峰命名规则映射
在配置文件中加入：
```
#开启驼峰命名自动映射，即从数据库字段名a_column 映射到Java 属性名acolumn
mybatis.configuration.map-underscore-to-camel-case=true
```
#### 条件查询
//% %表示模糊查找，引号中不能包含预编译的？，因此不能用#，而应该用表拼接的$。但是这样有一定问题，推荐用concat函数
![[Pasted image 20230803123452.png]]

### XML映射文件
简单的sql语句用注解的形式，复杂的sql语句用xml文件的形式，使代码更加简洁；
下述几点规范是为了让接口能准确定位到要用的sql语句：
1.XML映射文件的名称与Mapper接口名称一致，并且将XML映射文件和Mapper接口放置在相同包下(同包同名)
2.XML映射文件的namespace属性为Mapper接口全限定名一致。
3.XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。
![[Pasted image 20230803123919.png]]
#### 动态SQL
< if >:用于判断条件是否成立。使用test属性进行条件判断，如果条件为true，则拼接SQL
< where >: where元素只会在子元素有内容的情况下才插入where子句。而且会自动去除子句的开头的AND或OR
< set >:同where，可以替代sql语句中的set关键字，自动删除多余的逗号
```
<select id="list" resultType="com.itheima.pojo.Emp">
	select id, username,password,name,gender，image，job，
			entrydate,dept_id,create_time,update_time from emp
	<where>
		<if test= "name !=null">
			name like concat ('%',# {name } ,'%')
		</if>
		<if test="gender != null">
			and gender = #{gender}
		</if>
		<if test="begin != null and end != null">
			and entrydate between # {begin] and #{end}
		</if>
	</ where>
	order by update_time desc
</select>
```
< foreach >
collection:集合名称
item:集合遍历出来的元素/项
separator:每一次遍历使用的分隔符
open:遍历开始前拼接的片段
close:遍历结束后拼接的片段
![[Pasted image 20230803124940.png]]
< sql >和< include >
< sql >:定义可重用的SQL片段
< include >：通过属性refid，指定包含的sql片段
```
<sql id="commonSelect">
	select id, username, password, name, gender, image, job, 
			entrydate,dept_id, create_time, update_time from emp
</sql>


<select id="list" resultType="com.itheima.pojo.Emp">
	<include refid="commonSelect"/>
	where
		<if test="name != null">
			name like concat( %',#{name},'%')
		</if>
		<if test="gender != null">
			and gender = #{gender}
		</if>
		<if test="begin != null and end l= null">
			and entrydate between #{begin} and #{end}
		</if>
	order by update_time desc
</select>

<select id="getByld" resultType="com. itheima.pojo.Emp">
	<include refid="commonSelect"/>
	where id = #{id}
</select>
```
