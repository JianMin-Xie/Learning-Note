实战项目地址：https://github.com/JianMin-Xie/spring-boot-learning  
# Spring 常用配置
## Bean Scope
通过 @Scope 来实现，有以下几种：
* Singleton：spring 的默认配置，一个spring容器中只有一个Bean实例。
* Prototype：每次调用都新建一个Bean实例。
* Request：Web项目中，给每一个http Request新建一个Bean实例。
* Session：Web项目中，给每一个http Session新建一个Bean实例。
* GlobalSesson：这个只在protal应用中有用，给每一个global http Session新建一个Bean实例。
## SpEL表达式
Spring Expression Language (SpEL)是一种功能非常强大的表达式语言，可用于在运行时查询和操作对象。 SpEL书写在XML配置文件或者Annotation注解上，在Spring Bean的创建过程中生效。 
## Bean 的初始化与销毁
在 Bean 使用之前或者之后做些必要的操作。  
(1) Java 配置方式：使用 @Bean 的 initMethod 和 destoryMethod   
(2) 注解方式：利用 JSR-250 的 @PostContruct 和 @PreDestory  
## Profile
Profile 为在不同环境下使用不同的配置提供了支持，在开发中使用 @Profile 注解类或者方法，达到在不同情况下选择实例化不同的 Bean。
## 事件（Application Event）
Spring 的事件为 Bean 与 Bean 之间的消息通信提供了支持。  
Spring 的事件需要遵循如下流程：  
(1) 自定义事件，集成 ApplicationEvent  
(2) 定义事件监听器，实现 ApplicationListener  
(3) 使用容器发布事件  
## Spring Aware
在实际项目中，不可避免要用到 Spring 容器本身的功能资源，此时 Bean 必须要意识到 Spring 容器的存在，才能调用 Spring 所提供的资源，这就是所谓的 Spring Aware。使用 Spring Aware，Bean 将会和 Spring 框架耦合。Spring Aware 的目的是为了让 Bean 获得 Spring 容器的服务。
## 多线程
Spring 通过任务执行器（TaskExecutor）来实现多线程和并发编程。实际开发任务中一般是非阻碍的，即异步的，所以需要在配置类中通过 @EnableAsync 开启对异步任务的支持，在实际执行的 Bean 方法中使用 @Async 注解声明其实一个异步任务。
## 计划任务
配置类注解 @EnableScheduling 开启对计划任务的支持，要执行计划任务的方法上注解 @Scheduling 声明这是一个计划任务。
## 条件注解 @Conditional
@Conditional 根据满足某一个特定条件创建一个特定的 Bean。总的来说，就是根据特定条件来控制 Bean 的创建行为，这样我们可以利用这个特性进行一些自动的配置。
## 组合注解与元注解
元注解就是可以注解到别的注解上的注解，被注解的注解称为组合注解。组合注解具备元注解的功能。
## 测试
单元测试只针对当前开发的类和方法进行测试，可以简单通过模拟依赖来实现，对运行环境没有依赖。

# Spring MVC 基础
## Spring MVC 常用注解
### @Controller 注解
`@Controller`注解是开发中最常使用的注解，它的作用有两层含义：  
* 一是声明这个类是Spring MVC 的里的 Controller，告诉 Spring，被该注解标注的类是一个 Spring 的 Bean，需要被注入到Spring的上下文环境中。
* 二是该类里面所有被RequestMapping标注的注解都是HTTP服务端点
### @RequestMapping 注解
`@RequestMapping`注解是用来映射 Web 请求、处理类和方法的，可以注解在类或方法上。
### @ResponseBody 注解
* `@ResponseBody`修饰返回值，注解用于在 HTTP 的 body 中携带响应数据，默认是使用 JSON 的格式。
* 如果不加该注解，spring 响应字符串类型，是跳转到模板页面或 jsp 页面的开发模式。
* 说白了：加上这个注解你开发的是一个数据接口，不加这个注解你开发的是一个页面跳转控制器。
### @RequestBody 注解
* 用于接收和相应序列化数据(JSON)
* 可以支持嵌套
### @PathVariable 注解
`@PathVariable` 用来接收路径参数。
### @RestController 注解
`@RestController`相当于 `@Controller`和`@ResponseBody`结合。它有两层含义：
* 一是作为Controller的作用，将控制器类注入到Spring上下文环境，该类RequestMapping标注方法为HTTP服务端点。
* 二是作为ResponseBody的作用，请求响应默认使用的序列化方式是JSON，而不是跳转到jsp或模板页面。
## Spring MVC 基本配置
### 静态资源映射
程序的静态文件（js、css、图片）等需要直接访问，这时我们需要在配置里重写 addResourceHandlers 方法来实现。
### 拦截器
拦截器实现对每一个请求处理前后进行相关的业务处理，类似于 Servlet 的 Filter。可以让普通的 Bean 实现 HandlerInterceptor 接口或者继承 HandlerInterceptorAdapter 类来实现自定义拦截器。  
通过重写 WebMvcConfigurerAdapter 的 addInterceptors 方法来注册自定义拦截器。
### 文件上传
Spring MVC通过配置 MultipartResolver 来上传文件。
### 自定义 HttpMessageConvertor
HttpMessageConvertor 是用来处理 request 和 response 里的数据的。通常来说，继承 AbstractHttpMessageConverter<> 实现。
### 服务器推送技术
服务器推送的方案都是基于：当客户端向服务端放请求，服务端会抓住这个请求不放，等有数据更新的时候才返回客户端，当客户端接收到消息后，再向服务端发送请求，周而复始。  
(1) 基于 SSE（Server Send Event 服务端发送事件）的服务器端推送  
(2) 基于 Servlet 3.0+ 的异步方法特性  

# Spring Boot基本配置 
## 入口类和 @SpringBootApplication
SpirngBoot 通常有一个名为 xxxApplication 的入口类，入口类里有一个 main 方法，这个 main 方法就是一个标准的 Java 应用程序的入口方法。
## 核心注解 @SpringBootApplication
@SpringBootApplication 是 SpringBoot 的核心注解，它是一个组合注解，源码如下：  
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};
}
```
若不使用 @SpringBootApplication 注解，可以替换为 @SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan  

* @EnableAutoConfiguration 会根据类路径中的 jar 包依赖为当前项目进行自动配置

核心功能由 @EnableAutoConfiguration 注解提供，源码为：  
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};
}
```
关键功能是 @Import 注解导入的配置功能，AutoConfigurationImportSelector 使用 SpringFactoryLoader.loadFactoryNames 方法来扫描具有 META-INF/spring.factories 文件的 jar 包，在 spring-boot-autoconfigure-2.1.0.x.jar 中有一个 spring.factories 文件，此文件声明了有哪些自动配置，如下图所示：  
![](https://github.com/JianMin-Xie/Learning-Note/blob/master/pic/spring.factories文件.jpg)  

每一个自动配置类进行自动配置功能（spring.factories中的每一行对应的类）,以 HttpEncodingAutoConfiguration 为例讲解一下：  
```
//加载application全局配置文件内的部分配置到HttpEncodingProperties里面
@Configuration
@EnableConfigurationProperties({HttpEncodingProperties.class}) 
//当web容器类型是servlet的时候执行本类中的自动装配代码
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
//当有一个CharacterEncodingFilter的这样一个类的字节码文件时时执行本类中的自动装配代码
@ConditionalOnClass({CharacterEncodingFilter.class})
//当spring.http.encoding配置值为enabled的时候执行本类中的自动装配代码
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true   //如果application配置文件里面不配置，默认为true
)
public class HttpEncodingAutoConfiguration {
    private final HttpEncodingProperties properties;

    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
        this.properties = properties;
    }

    @Bean
    //当没有CharacterEncodingFilter这个Bean就实例化CharacterEncodingFilter为一个bean
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpEncodingProperties.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpEncodingProperties.Type.RESPONSE));
        return filter;
    }

    @Bean
    public HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
        return new HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer(this.properties);
    }
    
   //此处省略与自动加载无关的代码：HttpEncode的逻辑及其他
}
```
在配置类加载过程中，大量的使用到了条件加载注解：  
| 条件注解                            | 使用说明                                           |
|---------------------------------|------------------------------------------------|
| @ConditionalOnClass             | classpath中存在该类字节码文件时，才执行实例化方法或将类实例化            |
| @ConditionalOnMissingClass      | classpath中不存在该类字节码文件时，才执行实例化方法。（不存在A的时候去初始化B）  |
| @ConditionalOnBean              | DI容器中存在该类型Bean时，才执行实例化方法或将类实例化                 |
| @ConditionalOnMissingBean       | DI容器中不存在该类型Bean时，才执行实例化方法或将类实例化                |
| @ConditionalOnSingleCandidate   | DI容器中该类型Bean只有一个或@Primary的只有一个时，才执行实例化方法或将类实例化 |
| @ConditionalOnExpression        | SpEL表达式结果为true时，才执行实例化方法或将类实例化                 |
| @ConditionalOnProperty          | 参数设置或者值一致时，才执行实例化方法或将类实例化                      |
| @ConditionalOnResource          | 指定的文件存在时，才执行实例化方法或将类实例化                        |
| @ConditionalOnJndi              | 指定的JNDI存在时，才执行实例化方法或将类实例化                      |
| @ConditionalOnJava              | 指定的Java版本存在时，才执行实例化方法或将类实例化                    |
| @ConditionalOnWebApplication    | Web应用环境下，才执行实例化方法或将类实例化                        |
| @ConditionalOnNotWebApplication | 非Web应用环境下，才执行实例化方法或将类实例化                       |






