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

