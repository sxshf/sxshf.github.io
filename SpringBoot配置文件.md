---
title: SpringBoot配置文件
date: 2019-12-17
tags:
	- Java
	- SpringBoot
categories: 专业知识
---

# SpringBoot配置文件

## 1、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的。

- application.properties
- **application.yml**

配置文件的作用：修改SpringBoot的自动配置的默认值；SpringBoot在底层都给我们自动配置好

YAML(YAML Ain't Markup Language)

- YAML A Markup Language：是一个标记语言

- YAML isn‘t Markup Language：不是一个标记语言

标记语言：

- 以前的配置文件大多是xml文件

- YAML：**以数据为中心**，比json、xml等更适合做配置文件

  YAML配置例子

```yaml
server:
  port: 8081
```

​		XML：

```xml
<server>
	<port>8081</port>
</server>
```

## 2、YAML语法

### 1、基本语法

k:(**空格**)v：表示一对键值对（空格必须有）；

以空格的缩进控制层级关系；只要是左对齐的一列数据，都是同一个层级的

### 2、值的写法

**字面量：普通的值（数字、字符串、布尔）**

​	k:v ：字面直接来写

​	  		字符串默认不用加上单引号或者双引号

​				""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思

​							name："zhangsan \n lisi"： 输出，zhangsan 换行 lisi

​				''：单引号；会转义特殊字符，特殊字符最终只是个普通的字符串数据

​							name：’zhangsan \n lisi‘：输出，zhangsan \n lisi

**对象、Map**

​	k:v：

```yaml
friends:
	lastName: zhangsan
	age: 18
```

行内写法

```yaml
friends: {lastName: zhangsan,age: 18}
```

**数组**

```yaml
pets
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

## 3、配置文件值注入

### **1、yaml配置文件**

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2019/11/11
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

**JavaBean**

```Java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中（默认从全局配置文件中获取值）
 * @ConfigurationProperties: 告诉springboot将本类中的所有属性和配置文件中的相关的配置进行绑定
 *      prefix = "person"：配置文件中在哪个下面的属性进行一一映射
 *
 *只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties的功能
 */

@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private  Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

### **2、properties配置文件**

```properties
#idea使用的是utf-8
#file-encoding

person.last-name=张三
person.age=18
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=dog
person.dog.age=15

```

会有乱码问题，ctrl + alt + s 快捷键去setting界面 输入 file-encoding

![image-20191217182412029](/SpringBoot配置文件/image-20191217182412029.png)

### 3、@Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value。

如果说，我们专门编写了一个JavaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties

### 4、配置文件注入值数据校验

```Java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值"/#						{SpEL}>
     </property>
     * </bean>
     */
    @Email
  //@Value("${person.last-name}")
    private String lastName;
    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private  Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;

```

### 5、@PropertySource&@ImportResource

@PropertySource：加载指定的配置文件；

```Java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties: 告诉springboot将本类中的所有属性和配置文件中的相关的配置进行绑定
 *      prefix = "person"：配置文件中在哪个下面的属性进行一一映射
 *
 *只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties的功能
 */

@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值"/#{SpEL}></property>
     * </bean>
     */
//    @Email
  //@Value("${person.last-name}")
    private String lastName;
//    @Value("#{11*2}")
    private Integer age;
//    @Value("true")
    private  Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

@ImportResource：导入spring的配置文件，让配置文件里面的内容生效；

SpringBoot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上

```Java
@ImportResource(locations = {"classpath:beans.xml"})//导入Spring的配置文件让其生效
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

不来编写Spring的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.syj.service.HelloService"></bean>
</beans>
```

SpringBoot推荐给容器中添加组件的方式：使用全注解方式

1、配置类======Spring配置文件

2、使用@Bean给容器中添加组件

​	 **TestApplication.class必须和@Configuration注解的配置类在同一包中，不然就加载不了这个配置类**

```Java
/**
 * @Configuration 指明当前类就是个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean></bean>标签添加组件
 */

@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中默认的id就是方法名
    @Bean
    public HelloService helloService() {
        System.out.println("配置类给容器中添加组件了");
        return new HelloService();
    }
}
```

## 4、配置文件占位符

### 1、随机数

```Java
${random.value}
```

### 2、占位符获取之前配置的值，如果没有可以用:指定默认值

```pro
#idea使用的是utf-8
#file-encoding

person.last-name=张三${random.uuid}
person.age=${random.int}
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```

## 5、Profile

### 1、多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties.yml

默认使用application.properties的配置；

### 2、yml支持多文档块方式

```yml
server:
  port: 8081
spring:
  profiles:
    active: dev
---
server:
  port: 8083
spring:
  profiles: dev
---
server:
  port: 8084
spring:
  profiles: prod
---
```



### 3、激活指定profile

​	1、在配置文件中指定 spring.profiles.active=dev

​	2、命令行

​			--spring.profiles.active=dev

​	3、虚拟机参数

​			-Dspring.profiles.active=dev