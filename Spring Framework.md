# Spring Framework

## 1. The IOC Container

> Spring中关于IOC的实现在包*org.springframework.beans*,*org.springframework.context*

![image-20200607220453355](./images/IOC%20Container.jpg)

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

`Lifecycle`接口

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

`Lifecycle`接口为**具有生命周期的任意对象**提供一致的接口

> 容器本身也可以实现该接口，当容器接收到启动/停止信号时，会级联调用容器内所有实现该接口的对象，具体的调用工作是由`LifecycleProcessor`完成的



> `Lifecycle`接口只是代表了实现了该接口的对象具有生命周期，**不代表在上下文(context)刷新时==自动启动==** ，如果需要自动启动的功能，可以考虑实现`SmartLifecycle`接口

如果有 ==对象A== ---依赖-->==对象B== 那么对象A则应该在对象B之后启动，在对象B之前停止​​，`SmartLifecycle`的父接口`Phased`的`getPhase()`方法的返回值代表了**具有生命周期对象的自启动的优先级**，值越小，越早启动/越晚停止

> 只实现了`Lifecycle`而未实现`SmartLifecycle`的对象，**其Phased值相当于0**

> `Lifecycle`和`SmartLifecycle`的区别在于实现了前者的对象不会自动起/停，需要显式调用start/stop方法，而实现了后者的对象则会自动随着容器起/停，phased值定义其自动工作的优先级

##### 在非web应用中关闭容器(*待补充*)

#### 1.5.2 `Aware`系列接口

`Aware`接口为一个顶级的空接口，子类是各种组件的aware即`xxxAware`，当类实现了`xxxAware`时，该类中会持有一个访问xxx的引用，如果实现了`BeanFactoryAware`接口，那么类中就会持有一个能够访问`BeanFactory`的引用

> 实现该类接口会将代码耦合到Spring框架中，所以不推荐使用，主要应用在基建类Bean中

### 1.6 Bean定义继承(Bean Definition Inheritance)

Bean在容器中以`BeanDefinition`的形式存储，Bean的定义采用父子继承的模型

子类Bean继承父类Bean的：

- 作用域scope
- 构造函数的参数值
- 属性值
- 方法重载

子类自己单独定义的：

- 依赖项
- 自动装配模式
- 依赖项检查
- 是否为单例
- 惰性初始化

### 1.7 容器拓展(Container Extension Points)

> 如果要对容器进行一些定制化，不需要实现`ApplicationContext`，可以实现Spring提供的各类功能的接口来进行拓展

#### 1.7.1 `BeanPostProcessor`

`BeanPostProcessor`负责在bean实例化之后、调用初始化方法前后添加自定义的逻辑，可以有多个`BeanPostProcessor`，可通过实现`Ordered`接口来定义多个`BeanPostProcessor`的优先级

> 由于AOP本身是通过`BeanPostProcessor`来实现的，所以`BeanPostProcessor`类不能用作AOP

####  1.7.2 `BeanFactoryPostProcessor`

`BeanFactoryPostProcessor`和`BeanPostProcessor`类似，只不过`BeanFactoryPostProcessor`是用来处理**构建容器的元数据**的

BeanPostProcessor是在Bean实例化后做的追加操作，BeanFactoryPostProcessor是在Bean放入容器之前就进行的追加操作，可以修改Bean的属性、生命周期等。

`BeanFactoryPostProcessor`的一个比较典型的应用就是将bean的属性配置放在外部的配置文件中，比如数据库连接的四个属性

#### 1.7.3 `FactoryBean`

如果某个Bean是工厂类，那么它就可以实现这个接口。在Spring容器中，调用`getBean("id")`会得到名为`id`的`Bean实例`，而`getBean("&id")`则会得到生产名为id的Bean的`工厂Bean`

### 1.8 基于注解的配置方式

> - 基于注解的配置优先于xml方式，也就是说同一属性的配置，xml方式会覆盖掉注解的方式

#### `@Autowired`

- ##### 四种自动装配的模式

  - 无
  - byName
  - byType

  如果一个bean中只有一个构造函数，那么这个构造函数上就不需要加该注解也可以生效。

  `@Autowired`是由`BeanPostProcessor`实现的，所以在自定义的`BeanPostProcessor`中不能加`@Autowired`

###  1.9使用 JSR 330 规范注解


### 1.x Bean构造流程

容器启动后，将非延迟加载的Bean注册到容器中

1. 调用Bean的构造函数初始化Bean
2. 若实现`BeanPostProcessor`接口，调用`postProcessBeforeInitialization`方法
3. 若指定`init-method`/实现`InitializingBean`接口/`@PostStruct`注解，那么执行对应的方法
4. 若实现`BeanPostProcessor`接口，调用`postProcessAfterInitialization`方法
5. 使用Bean
6. 卸载时，若指定`destroy-method`/实现`DisposableBean`接口/`@PreDestroy`注解，那么执行对应的方法
7. 卸载





## 2. Resources

## x. Spring相关注解的使用

> 本章节主要介绍一些在学习Spring的过程中了解到的一些注解的使用方法

### 定时任务

- `@Scheduled` 将所标注的方法设置为定时方法，定时参数可通过该注解的参数设置

- `@Schedules`其参数为`@Scheduled`数组，用来指定一个方法可以有多种定时方式
> 启动类要加`@EnableScheduling`注解，表明启动定时功能






