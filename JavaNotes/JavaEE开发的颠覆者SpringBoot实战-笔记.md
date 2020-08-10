# 基本配置
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
![](https://github.com/JianMin-Xie/Learning-Note/pic/spring.factories.jpg)

