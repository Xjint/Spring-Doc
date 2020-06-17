# Spring Framework

## 1. The IOC Container

> Spring中关于IOC的实现在包*org.springframework.beans*,*org.springframework.context*

![image-20200607220453355](../images/IOC%20Container.jpg)

### 1.1 容器概述

Spring中的容器一般是指`ApplicationContext`接口/实现类，关于容器一般有三个步骤：

1. 构建容器
2. 实例化容器
3. 使用容器

容器的**构建**需要`元数据`，可通过xml文件/注解/java或混合的方式进行。*相当于提供一张容器的设计图*

实例化容器，直接`new`容器的实现类即可，要与上一步构建容器的方式相对应，比如用xml配置文件，那么实例化容器就需要

```java
ApplicationContext ac = new ClassPathXmlApplicationContext(String[]);
```

> 参数为元数据的位置

使用容器，可直接`ac.getBean(beanName)`等方式获取Bean

### 1.2 Bean概述

Bean即为普通Java对象(POJO)

```java
public class Person{
    private String name;
    private Integer age;
	public String getName(){
        return name;
    }
    public Integer getAge(){
        return age;
    }
    public void setName(String name){
        this.name = name;
    }
    public void setAge(Integer age){
		this.age = age;        
    }
}
```

Bean中的属性都为private,且对外提供get/set方法，**还要有无参构造器**

Bean在容器中被包装成为`BeanDefinition`类，包含有以下几类信息：

- 该Bean所在的全限定类名，通常是这个bean的实现类，如com.demo.dao.userDao
- Bean的行为配置，如**作用域、生命周期**等
- **对于其他Bean的依赖/引用**
- 在新创建的对象中需要设置的配置。比如池的大小限制或管理连接池的Bean中使用的连接数

#### 将Bean注册到容器中

可使用`@Bean`注解，该注解是一个方法级别的注解，将方法的*返回类型*作为Bean的类型注册到容器中，该Bean的名字默认为*该方法的名字*,

```java
@Configuration
public class Config{
    @Bean
    public UserServiceImpl userService(){
        return new UserServiceImpl();
    }
}
```

也可以让方法返回基类/接口

#### 实例化Bean

- 构造器注入
- 静态工厂方法注入
- 实例工厂方法注入(FactoryBean)

### 1.3 依赖

#### 依赖注入

依赖注入可分为以下两种：

- 构造器注入
- set注入

**循环依赖的依赖项可使用set注入解决**

#### 依赖和详细配置

##### `@Autowired`四种自动装配的模式

- 无
- byName
- byType
- 



### 1.4 Bean Scope

1. `singleton`(默认)
2. `prototype`
3. `request`
4. `session`
5. `application`
6. `websocket`

### 1.5 自定义Bean的属性

#### 1.5.1 生命周期回调

##### 初始化回调&销毁回调(Initialization Callbacks & Desctruction Callbacks)

为了能够让容器管理Bean的生命周期，可以让我们的Bean实现`InitializingBean` `DisposableBean`接口，表示Bean的`初始化后`要执行的方法和销毁前执行的方法**

这种实现的方法使得业务代码和Spring耦合在一起，所以更推荐使用注解`@PostConstruct` `@PreDestory`或xml中的`init-method` `destory-method`

> 也可以使用`@Bean(init = "[method-name]")`的形式来指定，不常用，这个和接口/注解 方式不冲突，可以同时生效

当同时使用这三种方法且每种对应的初始化方法都不一样时，可以都生效，作用顺序如下：

**初始化**：

1. `@PostConstruct`
2. `InitializingBean` 接口
3. `init()`方法(xml中定义 <bean init-method=""/>)

销毁：

1. `@PreDestory`
2. `DisposableBean`接口
3. `destory()`方法(xml中定义<bean destory-method=""/>)

> 若采用这三种方式引用的方法为同一个方法，那么就只会调用一次，不会重复调用

---

##### 启动&关机回调(Startup & Shutdown Callbacks)



#### 1.5.2 `Aware`系列接口





## 2. Resources








