---
title: "Spring Beans的装配规则总结"
author: "Exfly"
cover: "/media/img/Java/beans/springLogo.png"
tags: ["Spring", "SSM"]
date: 2018-05-20T16:49:06+08:00
---

文章简介：spring中bean的装配有一定规则，在这里进行总结。本文主要讲解一些概念和java配置方法。

<!--more--> 

# 目录
* 手动装配
	* 使用@Bean
* 自动装配 使用@ComponentScan
* 自动装配的歧义性
* 条件生效 Bean
* profile
* bean作用域

## 手动装配
手动装配可以通过声明xml文件和java配置文件两种手段。在这两种方式中，更加推荐使用java配置的方式。两种配置不是相互替代的关系，一般将业务相关的配置放到java配置中，对于非业务，如数据库等，可以放到xml中。
通过使用@Bean注解方法，可以声明并注册一个以方法名为name的bean。@Bean(name= {"sayAndPlayServiceNewName", "sayAndPlayService"})
```java
public interface SayAndPlayService {
	String say();
	String play();
}
public class PeopleSayAndPlayServiceImpl implements SayAndPlayService {
	@Override
	public String say() {
		return "People Service implements say.";
	}
	@Override
	public String play() {
		return "People Service implements play.";
	}
}

@Configuration
public class SimpleManualwireConfig {
	@Bean
	public SayAndPlayService sayAndPlayService() {
		return new PeopleSayAndPlayServiceImpl();
	}
}

@RunWith(SpringRunner.class)
@ContextConfiguration(classes= {org.exfly.demo.config.SimpleManualwireConfig.class})
public class SimpleBeanManualwireTest {
	@Autowired
	private SayAndPlayService service;
	@Test
	public void testTestOneAutowiredService() {
		Assert.assertEquals("People Service implements say.", service.say());
	}
	@Test
	public void testAnnotationConfigAppContext() {
		ApplicationContext context = new AnnotationConfigApplicationContext(org.exfly.demo.config.SimpleManualwireConfig.class);
		SayAndPlayService service = (SayAndPlayService) context.getBean("sayAndPlayService");
		Assert.assertEquals("People Service implements say.", service.say());
	}
}
```
## 自动装配 
使用@ComponentScan可以自动扫描，@ComponentScan告诉Spring 哪个packages 的用注解标识的类 会被spring自动扫描并且装入bean容器。自动扫描，会扫描相应包以及子包，并为所有bean生成name，name命名规则为其类首字母变小写，如interface UserService被唯一的UserServiceImpl实现，则经过扫描，bean被声明为name为userServiceImpl。如果接口被多各类实现，需要转到下文**消除歧义**部分进行了解。
```java
public interface SpeakService {
	String speak();
}
@Service
public class PeopleSpeakServiceImpl implements SpeakService {
	@Override
	public String speak() {
		return "People speak";
	}
}

@Configuration
@ComponentScan(basePackageClasses={org.exfly.demo.service.SpeakService.class})
public class SimpleConfigScanConfig {}

@RunWith(SpringRunner.class)
@ContextConfiguration(classes= {org.exfly.demo.config.SimpleManualwireConfig.class})
public class SimpleBeanManualwireTest {
	@Test
	public void testAnnotationConfigAppContextAutoScan() {
		ApplicationContext context = new AnnotationConfigApplicationContext(org.exfly.demo.config.SimpleConfigScanConfig.class);
		SpeakService service = (SpeakService) context.getBean("peopleSpeakServiceImpl");
		Assert.assertEquals("People speak", service.speak());
	}
}

//如果希望使用@Autowired自动装配，
@RunWith(SpringRunner.class)
@ContextConfiguration(classes= {org.exfly.demo.config.SimpleConfigScanConfig.class})
public class SimpleAutoScanTest {
	@Autowired //根据类型进行自动注入
	private SpeakService sservice;
	
	@Test
	public void testAnnotationConfigAppContextAutoScanAutoWire() {
		Assert.assertEquals("People speak", sservice.speak());
	}
}
```
### @Autowired
可以在属性、构造方法、set函数中进行自动注入

## 消除歧义
通过使用@Bean注解方法，可以声明并注册一个以方法名为name的bean。如果使用@Bean(name= {"sayAndPlayServiceNewName", "sayAndPlayService"})对bean进行命名，可以用不同的名字取用（在@Autowired处再加一个@Qualifier("sayAndPlayServiceNewName")）;
如果使用@ComponentScan，相应的Bean定义需要使用Component等进行注解，同时使用@Qualifier("BeanId")限定符，如下
```java
@Configuration
public class SimpleManualwireConfig {
	@Bean(name={"sayAndPlayServiceNewName", "sayAndPlayService"})
	public SayAndPlayService sayAndPlayService() {
		return new PeopleSayAndPlayServiceImpl();
	}	
}

@Service
@Qualifier("peopleSayServ")
public class PeopleSayServiceImpl implements SayService {}

//or
@Service("peopleSayServ")
public class PeopleSayServiceImpl implements SayService {}

//如何使用
@Autowired
@Qualifier("peopleSayServ")
SayService service;
```

## 条件生效 
如下的解释：在当前上下文中，如果Conditional注解中的MagitExistsCondition.matches方法返回true，则当前bean：magicBean生效。@Profile和springboot自动配置都是基于此种原理实现的。
```java
@Bean
@Conditional(MagitExistsCondition.class)
public MagicBean magicBean(){
	return new MagitBean();
}
public class MagitExistsCondition  implements Condition {
	boolean matches(ConditionContext ctxt, AnnotatedTypeMetadat metadate){
		Environment env = ctxt.getEnvironment();
		return env.containsProperty("magic");
	}
}
```

## @Profile
## bean作用域
* Singleton 默认 只创建一个实例
* Prototype    每次创建新的实例
* Session  每个会话创建一个实例
* Request  每个请求创建一个实例

使用@Scope进行配置即可
```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad{}
```

## 运行时值注入
以后补充
