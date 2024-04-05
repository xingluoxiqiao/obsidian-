# 微服务
## 单体架构
将业务的所有功能集中在一个项目中开发，打成一个包部署，架构简单，部署成本低但耦合度高
## 分布式架构
根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务
降低了服务耦合，有利于服务升级拓展
## 微服务
微服务是一种经过良好架构设计的**分布式架构**方案,微服务架构特征:
1.单一职责:微服务拆分粒度更小，每-一个服务都对应唯- -的业务能力，做到单- -职责,避免重复业务开发
2.面向服务:微服务对外暴露业务接口
3.自治:团队独立、技术独立、数据独立、部署独立
4.隔离性强:服务调用做好隔离、容错、降级，避免出现级联问题
# SpringCloud
SpringCloud集成了各种微服务功能组件，并基于springboot实现了这些组件的自动装配，从而提供了良好的开箱即用体验
![[Pasted image 20230915143539.png]]
![[Pasted image 20231008111713.png]]
## 服务拆分
### 注意事项
1．不同微服务，不要重复开发相同业务
2．微服务数据独立，不要访问其它微服务的数据库
3．微服务可以将自己的业务暴露为接口，供其它微服务调用
### 远程调用
各模块之间互相可以调用其它模块的信息，通过远程调用实现
通过java代码向其它模块发起请求并获取到相应的信息，得到信息后可以做其它处理
![[Pasted image 20230914112041.png]]
### RestTemplate
是java代码用来发送http请求的对象，要将其加载进容器中使用
使用方法：
1.在order-service的OrderApplication中注册RestTemplate
![[Pasted image 20230914112156.png]]
2.服务远程调用RestTemplate，并通过代码将两组信息组合起来
![[Pasted image 20230914112247.png]]
## Eureka
### 提供者和消费者
服务提供者:一次业务中，被其它微服务调用的服务。(提供接口给其它微服务)
服务消费者:一次业务中，调用其它微服务的服务。(调用其它微服务提供的接口)
提供者与消费者的角色是相对的，一个服务既可以是消费者，也可以是提供者
### 作用和架构
**1.消费者该如何获取服务提供者具体信息?**
服务提供者启动时向eureka注册自己的信息eureka保存这些信息
消费者根据服务名称向eureka拉取提供者信息
**2.如果有多个服务提供者，消费者该如何选择?**
服务消费者利用负载均衡算法，从服务列表中挑选一个
**3.消费者如何感知服务提供者健康状态?**
服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态
eureka会更新记录服务列表信息，心跳不正常会被剔除
消费者就可以拉取到最新的信息
![[Pasted image 20230914112716.png]]
![[Pasted image 20230914112634.png]]
### 搭建
1.引入eureka-server依赖
```
<dependency>
	<groupid>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
2.添加@EnableEurekaServer注解
3.在配置文件中配置eureka地址
![[Pasted image 20230914113209.png]]
### 服务注册
1.在user-service项目引入client依赖
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
2.在配置文件中编写配置
![[Pasted image 20230914113444.png]]
### 服务发现
服务拉去是基于服务名称获取服务列表，然后再对服务列表做负载均衡
1.修改orderservice的代码，修改访问的url路径，用服务名代替ip、端口
2.在orderservice的启动类orderapplication中的resttemplate添加负载均衡注解@LoadBalanced
### Eureka使用小结
![[Pasted image 20230914113823.png]]
### 负载均衡原理（Ribbon）
![[Pasted image 20230914113932.png]]
根据上述流程可以发现，实现负载均衡的方式是通过IRule来指定的,默认实现是ZoneAvoidanceRule，根据zone选择服务列表，然后轮询
ribbon已经提供了很多IRule的实现了来指定不同的方式实现负载均衡
我们可以切换实现负载均衡的方式，有两种方法
1.在提供者的application类中定义一个新的IRule并加载为bean(代码方法)
**这种方法提供者访问任何微服务时都生效**
```
@Bean
public IRule randomRule(){
	return new RandomRule();
}
```
2.在提供者的配置文件中，添加新的配置（配置文件法）
```
userservice:
	ribbon:
	NFLoadBalancerRuleclassName: com.netflix.loadbalancer.RandomRule #负载均衡规则
```
**这种方法只有在访问特定微服务时生效**
### 饥饿加载
Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长。而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载:
```
ribbon :
	eager-load:
	enabled: true #开启饥饿加载
	clients: userservice #指定对userservice这个服务饥饿加载
```
## Nacos
可以实现eureka的类似功能，并且功能更加丰富
### 快速入门
#### 服务注册和发现
单机模式运行，命令行指令
```
startup.cmd -m standealone
```
![[Pasted image 20230914121200.png]]
![[Pasted image 20230915143821.png]]
#### nacos服务分级存储模型
分为三层：服务、集群、实例
![[Pasted image 20230915143957.png]]
服务调用尽可能选择本地集群的服务，跨集群调用延迟较高本地集群不可访问时，再去访问其它集群
#### 服务集群属性配置
在yml文件中添加以下配置(添加spring.cloud.nacos.discovery.cluster-name)
```
spring:
	cloud :
		nacos:
			server-addr: localhost:8848 # nacos服务端地址
			discovery :
				cluster-name: HZ # 配置集群名称，也就是机房位置，例如:HZ，杭州
```

#### 服务实例权重设置
![[Pasted image 20230915144834.png]]
#### nacosRule负载均衡
可以实现优先访问本集群中的实例
1.修改yml设置集群
2.在userservice.ribbon.NFLoadBalanceRuleClassName中设置负载均衡的IRule为NacosRule，这个规则会优先寻找与自己同集群的服务
```
userservice:
	ribbon:
	NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule #负载均衡规则

```
3.将user-service的权重都设置为1，集群内部是随机访问
#### nacos和eureka对比
共同点：
1.都支持服务注册和服务拉取
2.都支持服务提供者心跳方式做健康检测
区别：
1.Nacos支持服务端主动检测提供者状态:临时实例采用心跳模式，非临时实例采用主动检测模式；临时实例心跳不正常会被剔除，非临时实例则不会被
剔除
```
//服务注册到nacos时，可以选择注册为临时或非临时实例
配置如下
spring：
	cloud:
		nacos:
			discovery:
				ephemeral:false#设置为非临时实例
```
2.Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
3.Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP式;Eureka采用AP方式
### nacos配置管理
在nacos网站上可以添加配置
![[Pasted image 20230915145645.png]]
#### 统一配置管理
nacos网站上添加的配置要被读取，需要知道配置的信息和名称，这需要在读取yml文件之前完成，因此需要定义一个比application优先级更高的配置文件，这就是bootstrap.yml
1.引入nacos的配置管理客户端依赖
```
<!--nacoS配置管理依赖-->
<dependency>
	<groupid>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```
2． 在userservice中的resource目录添加一个bootsto无识时磨集，这个文件是引导文件，优先级高于application.yml
```
spring:
	application:
		name: userservice #服务名称
		profiles:
			active: dev #开发环境，这里是dev
		cloud:
			nacos:
				server-addr: localhost:8848 #Nacos地址
				config:
					file-extension: yaml#文件后缀名
```
#### 配置热更新
nacos中的配置文件变更后，微服务无需重启就可以感知
方法一：在@Value注入的变量所在类上添加注解@RefreshScope
方法二：定义配置类，使用@ConfigurationProperties注入，自动刷新
```
@Component
@Data
@configurationProperties(prefix ="pattern")
public class PatternProperties {
	private string dateformat;
}
```
#### 注意事项
1.不是所有的配置都适合放到配置中心，维护起来比较麻烦
2.建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置
#### 多环境共享配置
![[Pasted image 20230915150935.png]]
![[Pasted image 20230915151057.png]]
因此可以将多环境共享的配置写入服务名称.yaml文件中
## Feign
### RestTemplate存在的问题
代码可读性差，编程体验不统一（代码中用string来表示url）
由于url通过代码写死，遇到参数复杂的url时难以维护
### 定义和使用Feign客户端
1.引入Feign的依赖
```
<dependency>
	<groupid>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
2.在order-service的启动类添加注解@EnableFeignClients开启Feign的功能
3.编写Feign客户端接口
```
@Feignclient("userservice")
public interface UserClient {
	@GetMapping("/user/iid}")
	User findById(@PathVariable(id") Long id);
}
```
![[Pasted image 20230918124923.png]]
4.使用客户端
![[Pasted image 20230918125101.png]]
客户端的代码中的变量是通过注解传参，url有多个参数时，只需要添加对应的传递变量即可，同时直接通过函数调用获取url，解决了RestTemplate的问题
### Feign的自定义配置
![[Pasted image 20230918131053.png]]
#### 配置文件方式配置日志级别
1.全局配置
```
feign:
	client:
		config:
			default: #这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
				loggerLevel: FULL#日志级别
```
2.局部配置
```
feign:
	client:
		config:
			userservice: #这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
				loggerLevel: FULL#日志级别
```
#### java代码方式配置日志级别
1.声明一个bean
```
public class FeignclientConfiguration {
	@Bean
	public Logger.Level feignLogLevel(){
		return Logger. Level.BASIC;
}
```
2.如果是全局配置，则把它放到@EnableFeignclients这个注解中:
@EnableFeignClients(defaultConfiguration = FeignclientConfiguration.class)
如果是局部配置，则把它放到@Feignclient这个注解中:
@FeignClient(value = "userservice"，configuration = FeignclientConfiguration.class)
### 性能优化
Feign底层的客户端实现:
URLConnection:默认实现，不支持连接池
Apache HttpClient:支持连接池
OKHttp:支持连接池

因此优化Feign的性能主要包括:
使用连接池代替默认的URLConnection
日志级别，最好用basic或none
![[Pasted image 20230918131650.png]]
### Feign的最佳实践
#### 定义统一父接口
![[Pasted image 20230918131909.png]]
#### 抽取独立模块
![[Pasted image 20230918131945.png]]
#### 方式二代码实现
1.首先创建一个module，命名为feign-api，然后引入feign的starter依赖
2.将order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中
3.在order-service中引入feign-api的依赖
4.修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包
5.重启测试
此时feignclient定义在feign-api中，不在要使用的类中，无法将其注入bean中，需要导入包，有两种方式
不同包的Feignclient的导入有两种方式:
在@EnableFeignclients注解中添加basePackages，指定FeignClient所在的包
在@EnableFeignclients注解中添加clients，指定具体FeignClient的字节码
## Gateway
### 网关的功能和种类
功能：
身份认证和权限校验
服务路由、负载均衡
请求限流

在Springcloud中网关的实现包括两种:gateway和zuul
Zuul是基于Servlet的实现，属于阻塞式编程。而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程的实现，具备更好的性能。
### 搭建网关服务
1．创建新的module，引入SpringCloudGateway的依赖和nacos的服务发现依赖:
```
<!--网关依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</ dependency>
```
2.编写路由配置及nacos地址
网关路由可以配置的内容包括:
路由id:路由唯一标示
uri:路由目的地，支持lb和http两种
predicates:路由断言，判断请求是否符合要求，符合则转发到路由目的地
filters:路由过滤器，处理请求或响应
```
server:
	port: 10010#网关端口
spring:
	application:
		name: gateway #服务名称
	cloud :
		nacos:
			server-addr: localhost:8848 # nacos地址
		gateway :
			routes: #网关路由配置
				- id: user-service #路由id，自定义，只要唯─即可
				# url: http://127.0.0.1:8081 #路由的目标地址 http就是固定地址
				url: lb://userservice #路由的目标地址 lb就是负载均衡，后面跟服务名称
				predicates: #路由断言，也就是判断请求是否符合路由规则的条件
				 -Path=luser/** #这个是按照路径匹配，只要以user/开头就符合要求
```
![[Pasted image 20230918133529.png]]
### 路由断言工厂
我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件
例如Path=/user/\*\*是按照路径匹配，这个规则是由
org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory类来处理的
spring提供了11中这样类似的断言工厂类，具体可以查看官网
在配置文件中的断言（predicate）下按照特定断言工厂的规则编写断言语句，可以使网关对请求进行判断后进行后续处理
#### 过滤器
网关中还有一个结构可以对进入的请求进行预处理，就是过滤器
可以在配置文件中用default-filters或filters指定
其中，default-filters写在gateway下，表示对所有的请求都执行的操作
filters则是写在routes下，表示对特定请求路径进行操作
#### 全局过滤器（自定义过滤器）
对所有路由都生效的过滤器，并且可以自定义处理逻辑
实现GloalFilter接口(定义处理逻辑)，添加@order注解指定优先级（或实现Oredered接口重写getorder方法）
```
@Order(int)//越小，越优先处理
@Component
public class AuthorizeFilter implements GlobalFilter {
@0verride
	public Mono<Void> filter(ServerWebExchange exchange，GatewayFilterChain chain){
		//1.获取请求参数
		ServerHttpRequest request = exchange.getRequest();
		MultiValueMap<String，String> params = request.getQueryParams();
		//2.获取参数中的authorization参数
		String auth = params.getFirst("authorization");
		//3.判断参数值是否等于 admin
		if ( "admin".equals(auth)) {
			//4.是，放行
			return chain.filter(exchange);
		}
		//5.否，拦截
		return exchange. getResponse(.setcompleteo;
	}
}
```
#### 过滤器执行顺序
1.每一个过滤器都必须指定一个int类型的order值,order值越小，优先级越高，执行顺序越靠前。
2.GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定。
3.路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
4.当过滤器的order值一样时，会按照defaultFilter >路由过滤器>GlobalFilter的顺序执行。
总结：order值越小，优先级越高；
当order值一样时，顺序是defaultFilter最先，然后是局部的路由过滤器，最后是全局过滤器
### 网关的cors跨域配置
![[Pasted image 20230918135405.png]]
## Docker
大型项目组件较多，运行环境也较为复杂，部署时会碰到一些问题：
依赖关系复杂，容易出现兼容性问题；开发、测试、生产环境有差异 
Docker允许开发中将应用、依赖、函数库、配置一起打包，形成可移植镜像. Docker应用运行在容器中，使用沙箱机制，相互隔离，以此解决了兼容性问题
Docker镜像中包含完整运行环境，包括系统函数库，仅依赖系统的Linux内核，因此可以在任意Linux操作系统上运行，解决了开发、测试、生产环境有差异的问题

Docker是一个快速交付应用、运行应用的技术:
1.可以将程序及其依赖、运行环境一起打包为一个镜像，可以迁移到任意Linux操作系统
2.运行时利用沙箱机制形成隔离容器，各个应用互不干扰
3.启动、移除都可以通过一行命令完成，方便快捷
### Docker与虚拟机的区别
![[Pasted image 20230922144620.png]]
Docker和虚拟机的差异:
docker是一个系统进程;虚拟机是在操作系统中的操作系统. 
docker体积小、启动速度快、性能好;虚拟机体积大、启动速度慢、性能一般
### Docker架构
镜像（lmage） : Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。
容器（Container）∶镜像中的应用程序运行后形成的进程就是容器，只是Docker会给容器做隔离，对外不可见。
DockerHub: DockerHub是一个Docker镜像的托管平台。这样的平台称为Docker Registry,国内网易云镜像服务，阿里云镜像库等

Docker是一个CS架构的程序，由两部分组成:
服务端(server): Docker守护进程，负责处理Docker指令，管理镜像、容器等
客户端(client):通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令。
yum remove docker \
docker-client \
docker-client-latest \docker-common \
docker-latest \
docker-latest-logrotate ldocker-logrotate \
docker-selinux \
docker-engine-selinux \docker-engine \
docker-ce
