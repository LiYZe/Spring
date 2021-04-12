# Spring 

## 依赖
```bash
A a = new b;
```
a：被注入对象 b：被依赖对象
## IoC

```bash
Inversion of Control，中文通常翻译为“控制反转”，它还有一个别名叫做依赖注入（Dependency Injection）。
```
<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114373277-062e1000-9bb5-11eb-98b4-4d3e4283ce1b.png"/></div> 

通常情况下，被注入对象会直接依赖于被依赖对象。但是，在IoC的场景中，二者之间通过IoC ServiceProvider来打交道，所有的被注入对象和依赖对象现在由IoC Service Provider统一管理。


## IoC Service Provider

### 注入方式
被注入对象又是通过三种方式来通知IoC Service Provider为其提供适当服务。
#### 构造方法注入
被注入对象可以通过在其构造方法中声明依赖对象的参数列表，让外部（通常是IoC容器）知道它需要哪些依赖对象。

#### setter方法注入
通过setter方法，可以更改相应的对象属性，通过getter方法，可以获得相应属性的状态。所以，当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。

#### 接口注入
被注入对象如果想要IoC ServiceProvider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。。

(1.创建一个接口，包含一个方法A，该方法的参数为被依赖对象； 2. 实现该接口； 3.通过A方法注入对象）


### 职责

#### 业务对象的构建管理
在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作始终需要有人来做。所以，IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现。

#### 业务对象间的依赖绑定
IoC Service Provider通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。

#### 管理
IoC Service Provider实现职责的方式。

##### 直接编码方式
在容器启动之前，就可以通过程序编码的方式将被注入对象和依赖对象注册到容器中，并明确它们相互之间的依赖注入关系。
```bash
IoContainer container = ...;
container.register(FXNewsProvider.class,new FXNewsProvider());
container.register(IFXNewsListener.class,new DowJonesNewsListener());
...
FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);
newProvider.getAndPersistNews();
```
通过为相应的类指定对应的具体实例，可以告知IoC容器，当我们要这种类型的对象实例时，请将容器中注册的、对应的那个具体实例返回。

使用接口注入，除了注册相应对象，还要将“注入标志接口”与相应的依赖对象绑定一下，才能让容器最终知道是一个什么样的对应关系。
```bash
IoContainer container = ...;
container.register(FXNewsProvider.class,new FXNewsProvider());
container.register(IFXNewsListener.class,new DowJonesNewsListener());
...
container.bind(IFXNewsListenerCallable.class, container.get(IFXNewsListener.class));
...
FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);
newProvider.getAndPersistNews();
```
通过 bind 方法将“被注入对象”（由 IFXNewsListenerCallable 接口添加标志）所依赖的对象，绑定为容器中注册过的 IFXNewsListener 类型的对象实例。容器在返回 FXNewsProvider 对象实例之前，会根据这个绑定信息，将 IFXNewsListener 注册到容器中的对象实例注入到“被注入对象”——FXNewsProvider 中，并最终返回已经组装完毕的 FXNewsProvider 对象。
（bind所实现的是接口注入中的第一步：将接口中的方法参数设置为被依赖对象。）

##### 配置文件方式
这是一种较为普遍的依赖注入关系管理方式。最为常见的，还是通过XML文件来管理对象注册和对象间依赖关系。

（后续详细介绍）

##### 元数据方式（注解）
直接在类中使用元数据信息来标注各个对象之间的依赖关系，然后由Guice框架根据这些注解所提供的信息将这些对象组装后，交给客户端对象使用。

（后续详细介绍）

## IoC容器
Spring的IoC容器是一个IoC Service Provider，是一个提供IoC支持的轻量级容器。除了基本的IoC支持，它作为轻量级容器还提供了IoC之外的支持。
<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114380593-d4b94280-9bbc-11eb-902f-511f9ca9ffcc.png"/></div> 
Spring提供了两种容器类型：BeanFactory 和 ApplicationContext。

BeanFactory：
```bash
基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（lazy-load）。只有当客户端对象需要访问容器中的某个受管
对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求
不是很严格的场景， BeanFactory 是比较合适的IoC容器选择。
```

ApplicationContext：
```bash
ApplicationContext 在 BeanFactory 的基础上构建，是相对比较高级的容器实现，除了拥有 BeanFactory 的所有支持， ApplicationContext 还提供
其他高级特性，比如事件发布、国际化信息支持等， ApplicationContext 所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，
相对于 BeanFactory 来说， ApplicationContext 要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之 BeanFactory 也会
长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext 类型的容器是比较合适的选择。
```
BeanFactory 和 ApplicationContext的关系:ApplicationContext 间接继承自 BeanFactory ，所以说它是构建于 BeanFactory 之上
的IoC容器。
<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114400742-dbec4a80-9bd4-11eb-9693-c5764464e4fb.png"/></div> 


### BeanFactory
BeanFactory ，顾名思义，就是生产Bean的工厂.作为Spring提供的基本的IoC容器，BeanFactory 可以完成作为IoC Service Provider的所有职责，包括业务对象的注册和对象间依赖关系的绑定。

针对系统和业务逻辑，该如何设计和实现当前系统不受是否引入轻量级容器的影响。前后唯一的不同，就是对象之间依赖关系的解决方式改变了。之前我们的系统业务对象需要自己去“拉”（Pull）所依赖的业务对象，有了 BeanFactory 之类的IoC容器之后，需要依赖什么让 BeanFactory 为我们推过来（Push）就行了。

#### 对象注册与依赖绑定方式

##### 直接编码方式

<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114403610-88c7c700-9bd7-11eb-976b-c7563d7ad966.png"/></div> 
```bash
接口：BeanFactory（图书馆）：只定义如何访问容器内管理的Bean的方法
      BeanDefinitionRegistry（图书馆的书架）：定义抽象了Bean的注册逻辑，在BeanFactory 的实现中担当Bean注册管理的角色
      BeanDefinition：负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。
实现类：Default-ListableBeanFactory：负责具体Bean的注册以及管理工作。
        RootBeanDefinition：BeanDefinition的主要实现类
        ChildBeanDefinition：BeanDefinition的主要实现类
```

##### 外部配置文件方式
Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。

```bash
1.根据不同的外部配置文件格式，给出相应的 BeanDefinitionReader 实现类（完成大部分工作，包括解析文件格式、装配BeanDefinition等）
2.由 BeanDefinitionReader 的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition 
3.将映射后的 BeanDefinition注册到一个BeanDefinitionRegistry（负责保存）
4. BeanDefinitionRegistry即完成Bean的注册和加载。
```

###### Properties

Spring提供了 org.springframework.beans.factory.support.PropertiesBeanDefinitionReader 类用于Properties格式配置文件的加载。
```bash
djNewsProvider.(class)=..FXNewsProvider
# ----------通过构造方法注入的时候-------------
djNewsProvider.$0(ref)=djListener
djNewsProvider.$1(ref)=djPersister
# ----------通过setter方法注入的时候---------
# djNewsProvider.newsListener(ref)=djListener
# djNewsProvider.newPersistener(ref)=djPersister
djListener.(class)=..impl.DowJonesNewsListener
djPersister.(class)=..impl.DowJon
esNewsPersister
```
djNewsProvider 作为 beanName ，后面通过 .(class) 表明对应的实现类是什么。通过在表示 beanName 的名称后添加 .$[number] 后缀的形式，来表示当前 beanName 对应的对象需要通过构造方法注入的方式注入相应依赖对象。在这里，我们分别将构造方法的第一个参数和第二个参数对应到djListener 和 djPersister。

（注意： (ref) 用来表示所依赖的是引用对象，而不是普通的类型。如果不加 (ref)，PropertiesBeanDefinitionReader 会将 djListener 和 djPersister 作为简单的String类型进行注入，产生异常。）

setter方法与构造方法的区别：
```bash
构造方法注入无法通过参数名称来标识注入的确切位置，而setter方法注入则可以通过属性名称来明确标识注入。
```

##### XML

XML配置格式是Spring支持最完整，功能最强大的表达方式。大部分讲解还会沿用DTD的方式，只有必要时才会给出特殊说明。

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" ➥
"http://www.springframework.org/dtd/spring-beans.dtd">

<beans>
  <bean id="djNewsProvider" class="..FXNewsProvider">
    <constructor-arg index="0">
      <ref bean="djNewsListener"/>
    </constructor-arg>
    <constructor-arg index="1">
      <ref bean="djNewsPersister"/>
    </constructor-arg>
  </bean>

  <bean id="djNewsListener" class="..impl.DowJonesNewsListener">
  </bean>
  <bean id="djNewsPersister" class="..impl.DowJonesNewsPersister">
  </bean>
</beans>
```

#### 注解方式

通过注解标注的方式为 FXNewsProvider 注入所需要的依赖，现在可以使用 @Autowired 以及 @Component 对相关类进行标记。再向Spring的配置文件中增加一个“触发器”，使用 @Autowired 和 @Component 标注的类就能获得依赖对象的注入了。

##### @Autowired

它的存在将告知Spring容器需要为当前对象注入哪些依赖对象。

##### @Component

配合Spring 2.5中新的classpath-scanning功能使用的。







