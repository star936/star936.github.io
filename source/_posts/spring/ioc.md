> SpringBoot: 2.1.1

### 1. IoC简介

**IoC容器是Spring的核心,可以说Spring是一种基于IoC容器编程的框架,Spring Boot是基于注解的开发Spring IoC.**

**IoC是一种通过描述来生成或者获取对象的技术,该技术不是Spring设置也不是Java所独有的.**

**在Spring中把每一个需要管理的对象称为Spring Bean,而Spring管理这些Bean的容器称之为Spring IoC容器,所有的IoC容器都需要实现接口`BeanFactory`.它具有两个基本的功能:**

* 通过描述管理Bean,包括发布和获取Bean
* 通过描述完成Bean之间的依赖关系

### 2. 装配Bean

#### 2.1 通过扫描装配Bean

使用注解`@Component`和`@ComponentScan`进行扫描装配,其中`@Component`表明标明哪个类被扫描进入Spring IoC容器,`@ComponentScan`标明采用何种策略去扫描装配Bean. 

**注解@SpringBootApplication注入了@ComponentScan**

```java
# 注解@Component表明User类将被IoC容器扫描装配,"user"作为Bean的名称.
@Component("user")  
public class User {
    private Long id;
    private String userName;
    ...
}

# @Cofiguration代表这是一个Java配置文件,Spring的容器会根据它来生成IoC容器去装配Bean
# @ComponentScan默认对AppConfig的当前包及其子包进行扫描去装配Bean 
@Configuration
@ComponentScan
public class AppConfig {
}
```

**@ComponentScan的常用属性:**

* basePackages: 定义扫描的包名
* basePackageClasses: 定义扫描的类
* includeFilters: 定义满足过滤器条件的Bean才去扫描
* excludeFilters: 排除满足过滤器条件的Bean

**`includeFilters`和`excludeFilters`都需要通过一个注解`@Filter`去定义,`@Filter`有一个`type`类型,可以定义为注解或者正则式等类型,classes定义注解类,pattern定义正则式类.**

*例如: 将标注了`@Service`的类不被IoC容器扫描注入*

```java
@ComponentScan(basePackages="com.morven.rest.*", excludeFilters = (@Filter(classes = {Service.class})))
```

#### 2.2 自定义第三方Bean

**如果希望将第三方包的类对象也注入到Spring IoC容器中,使用@Bean注解.**

```java
@Configuration
public class AppConfig {
    # 通过@Bean会将该函数返回的对象用名称"dataSource"保存在IoC容器中
    @Bean(name="dataSource")
    public DataSource getDataSource() {
        ...
    }
}
```

### 3. 依赖注入(Dependency Injection, DI)

**描述的是Bean之间的依赖关系**

*例如:*

```java
public interface Person {
    public void service();
}
public interface Animal {
    public void use();
}
@Component
public class BussinessPerson implements Person {
    @Autowired
    private Anminal animal = null;
    @Override
    public void service() {
        this.animal.use();
    }
}

@Component
public class Dog implements Animal {
    @Override
    public void use() {
        System.out.println("Dog...");
    }
}
```

#### 3.1 注解@Autowired

**使用的最多的注解之一,它注入的机制最基本的一条是根据类型.它既可以标注属性,也可以标注方法.**

*如果我们再添加一个Animal的实现类Cat*

```java
@Component
public class Cat implements Animal {
    @Override
    public void use() {
        System.out.println("Cat...")
    }
}
```

*抛出异常:*

```bash
expected single matching bean but found 2: cat, dog
```

*修改:*

```java 
@Autowired
private Animal dog = null;
```

**原因: @Autowired首先根据类型找到对应的Bean,如果对应类型的Bean不是唯一的,那么它会根据其属性名称和Bean的名称进行匹配;如果匹配上就使用该Bean,否则抛出异常.**

**另外: @Autowired是一个默认必须找到对应Bean的注解,如果不确定其标注属性一定存在并且允许被标注的属性为null,那么可以如下配置**:

```java
@Autowired(required = false)
```

#### 3.2 使用@Primary和@Qualifier消除歧义性

*由于接口Animal有两个实现类,因此造成@Autowired装配的歧义性,`3.1`中给出了一种解决方法,但不是最好的方案;而@Primary和@Qualifier给出了更好的解决方案.*

* @Primary: 是一个修改优先权的注解,通过优先级的变换使得IoC容器知道注入哪个具体的实例来满足依赖注入.如下:

  ```java
  @Component
  # 优先使用该类进行注入
  @Primary
  public class Cat implements Animal {
      ...
  }
  ```

* @Qualifier: 有时候Dog和Cat class都带有@Primary注解,那么@Autowired和@Qualifier组合在一起,通过类型和名称一起找到Bean,以此来消除歧义性.如下:

  ```java
  @Autowired
  @Qualifier("dog")
  private Anminal animal = null;
  ```

#### 3.3 带有参数的构造方法类的装配

*如果有些类只有带有参数的构造方法,则可以使用如下方法::*

```java
@Component
public class BussinessPerson implements Person {
    private Anminal animal = null;
    
    public BussinessPerson(@Autowired @Qualifier("dog") Animal animal) {
        this.animal = animal;
    }
    
    @Override
    public void service() {
        this.animal.use();
    }
}
```

