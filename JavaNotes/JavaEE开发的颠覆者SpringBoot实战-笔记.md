# Spring 基础配置
## 依赖注入
依赖注入是指容器负责创建对象和维护对象间的依赖关系，而不是通过对象本身负责自己的创建和解决自己的依赖。
## Java 配置
Java 配置是通过 @Configuration 和 @Bean 来实现的。  
* @Configuration 声明当前类是一个配制类，相当于一个 Spring 配置的 xml 文件
* @Bean 注解在方法上，声明当前方法的返回值是一个 Bean

