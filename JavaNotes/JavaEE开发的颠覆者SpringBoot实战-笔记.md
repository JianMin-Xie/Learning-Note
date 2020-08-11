# Spring 常用配置
## Bean Scope
通过 @Scope 来实现，有以下几种：
* Singleton：spring 的默认配置，一个spring容器中只有一个Bean实例。
* Prototype：每次调用都新建一个Bean实例。
* Request：Web项目中，给每一个http Request新建一个Bean实例。
* Session：Web项目中，给每一个http Session新建一个Bean实例。
* GlobalSesson：这个只在protal应用中有用，给每一个global http Session新建一个Bean实例。


# Spring Boot基本配置
## 入口类和 @SpringBootApplication
SpirngBoot 通常有一个名为 xxxApplication 的入口类，入口类里有一个 main 方法，这个 main 方法就是一个标准的 Java 应用程序的入口方法。

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






