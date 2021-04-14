# 依赖

```bash
A a = new b;
```
a：被注入对象 b：被依赖对象

# IoC

```bash
Inversion of Control，中文通常翻译为“控制反转”，它还有一个别名叫做依赖注入（Dependency Injection）。
```
<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114373277-062e1000-9bb5-11eb-98b4-4d3e4283ce1b.png"/></div> 

通常情况下，被注入对象会直接依赖于被依赖对象。但是，在IoC的场景中，二者之间通过IoC ServiceProvider来打交道，所有的被注入对象和依赖对象现在由IoC Service Provider统一管理。


# IoC Service Provider

## 注入方式
被注入对象又是通过三种方式来通知IoC Service Provider为其提供适当服务。

### 构造方法注入
被注入对象可以通过在其构造方法中声明依赖对象的参数列表，让外部（通常是IoC容器）知道它需要哪些依赖对象。

### setter方法注入
通过setter方法，可以更改相应的对象属性，通过getter方法，可以获得相应属性的状态。所以，当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。

### 接口注入
被注入对象如果想要IoC ServiceProvider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。。

(1.创建一个接口，包含一个方法A，该方法的参数为被依赖对象； 2. 实现该接口； 3.通过A方法注入对象）

## 职责

### 业务对象的构建管理
在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作始终需要有人来做。所以，IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现。

### 业务对象间的依赖绑定
IoC Service Provider通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。

### 管理
IoC Service Provider实现职责的方式。

#### 直接编码方式
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

#### 配置文件方式
这是一种较为普遍的依赖注入关系管理方式。最为常见的，还是通过XML文件来管理对象注册和对象间依赖关系。

- [XML详细介绍](BeanFactory.md#XML)

#### 元数据方式（注解）
直接在类中使用元数据信息来标注各个对象之间的依赖关系，然后由Guice框架根据这些注解所提供的信息将这些对象组装后，交给客户端对象使用。

（后续详细介绍）

# IoC容器
Spring的IoC容器是一个IoC Service Provider，是一个提供IoC支持的轻量级容器。除了基本的IoC支持，它作为轻量级容器还提供了IoC之外的支持。
<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114380593-d4b94280-9bbc-11eb-902f-511f9ca9ffcc.png"/></div> 
Spring提供了两种容器类型：BeanFactory 和 ApplicationContext。

BeanFactory：
```bash
基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（lazy-load）。只有当客户端对象需要访问容器中的某个受管
对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求
不是很严格的场景， BeanFactory 是比较合适的IoC容器选择。
```

- [详细介绍](BeanFactory.md#BeanFactory)

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

# IoC容器功能实现

Spring的IoC容器所起的作用：以某种方式加载Configuration Metadata（通常也就是XML格式的配置信息），根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。

实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段：1.容器启动阶段 2.Bean实例化阶段

<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114668130-f0def000-9d32-11eb-975c-2f18b8b16f75.png"/></div> 

## 容器启动阶段：

（只是根据图纸装配生产线）

首先通过某种途径加载Configuration MetaData，随后依赖某些工具类（BeanDefinitionReader）对加载的进行解析和分析，并将分析后的信息编组为相应的BeanDefinition，最后把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，容器启动工作完成。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/114668657-9003e780-9d33-11eb-87f2-a6edc9b934a6.png"/></div>

###  BeanFactoryPostProcessor 

容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的 BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。.

### 实现方式

基本的IoC容器BeanFactory:手动方式应用所有的BeanFactoryPostProcessor；

较为先进的容器 ApplicationContext:将相应 BeanFactoryPostProcessor 实现类添加到配置文件，ApplicationContext 将自动识别并应用它。


#### 自定义：
```bash
自定义实现 BeanFactoryPostProcessor，通常我们需要实现org.springframework.beans.factory.config.BeanFactoryPostProcessor接口。
同时，因为一个容器可能拥有多个Bean FactoryPostProcessor，这个时候可能需要实现类同时实现Spring的org.springframework.core.Ordered
接口，以保证各个 BeanFactoryPostProcessor 可以按照预先设定的顺序执行
```

#### 常用的BeanFactoryPostProcessor实现类：

##### PropertyPlaceholderConfigurer:

PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder），并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。

##### PropertyOverrideConfigurer

可以通过占位符，来明确表明bean定义中的property与properties文件中的各配置项之间的对应关系。通过PropertyOverrideConfigurer可以对容器中配置的任何你想处理的bean定义的property信息进行覆盖替换。

```bash
beanName.propertyName=value
```
##### CustomEditorConfigurer

为了处理配置文件中的数据类型与真正的业务对象所定义的数据类型转换，Spring还允许我们通过org.springframework.beans.factory.config.CustomEditorConfigurer来注册自定义的PropertyEditor以补助容器中默认的PropertyEditor。XML所记载的，都是String类型，即容器从XML格式的文件中读取的都是字符串形式，最终应用程序却是由各种类型的对象所构成。要想完成这种由字符串到具体对象的转换，都需要这种转换规则相关的信息，而CustomEditorConfigurer就是帮助我们传达类似信息的

CustomEditorConfigurer是另一种类型的BeanFactoryPostProcessor实现，它只是辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动。

## Bean实例化阶段：

（使用装配好的生产线来生产具体的产品）

当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。

之所以说getBean()方法是有可能触发Bean实例化阶段的活动，是因为只有当对应某个bean定义的getBean()方法第一次被调用时，不管是显式的还是隐式的，Bean实例化阶段的活动才会被触发，第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例（prototype类型bean除外）。当getBean()方法内部发现该bean定义之前还没有被实例化之后，会通过createBean()方法来进行具体的对象实例化，实例化过程如图所示：

<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114673249-9052b180-9d38-11eb-8c10-23e7a5add8d9.png"/></div>

### 隐式调用：

两种情况下，BeanFactory的getBeean法可以被客户端对象隐式地调用

1.当对象A被请求而需要第一次实例化的时候，如果它所依赖的对象B之前同样没有被实例化，那么容器会先实例化对象A所依赖的对象。
这时容器内部就会首先实例化对象B，以及对象A依赖的其他还没有实例化的对象。
（B被隐式调用）

2.ApplicationContext会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean() 。

### Bean的实例化、设置对象属性与BeanWrapper

容器在内部实现的时候，采用“策略模式（Strategy Pattern）”来决定采用何种方式初始化bean实例。通常，可以通过反射或者CGLIB动态字节码生成来初始化相应的bean实例或者动态生成其子类。

#### 第一步“实例化bean对象”

##### InstantiationStrategy

org.springframework.beans.factory.support.InstantiationStrategy定义是实例化策略的抽象接口，其直接子类SimpleInstantiationStrategy实现了简单的对象实例化功能，可以通过反射来实例化对象实例，但不支持方法注入方式的对象实例化

##### CglibSubclassingInstantiationStrategy

CglibSubclassingInstantiationStrategy继承了SimpleInstantiationStrategy的以反射方式实例化对象的功能，并且通过CGLIB的动态字节码生成功能，该策略实现类可以动态生成某个类的子类，进而满足了方法注入所需的对象实例化需求。默认情况下，容器内部采用的是CglibSubclassingInstantiationStrategy。

##### 过程

容器只要根据相应bean定义的BeanDefintion 取得实例化信息，结合CglibSubclassingInstantiationStrategy及不同的bean定义类型，就可以返回实例化完成的对象实例。但是，返回方式上有些“点缀”。不是直接返回构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例进行包裹，返回相应的 BeanWrapper 实例。

#### 第二步“设置对象属性”

BeanWrapper接口通常在Spring框架内部使用，它有一个实现类org.springframework.beans.BeanWrapperImpl 。其作用是对某个bean进行“包裹”，然后对这个“包裹”的bean进行操作，比如设置或者获取bean的相应属性值。而在第一步结束后返回BeanWrapper实例而不是原先的对象实例。

##### BeanWrapper

BeanWrapper定义继承了org.springframework.beans.PropertyAccessor接口，可以以统一的方式对对象属性进行访问。BeanWrapper定义同时又直接或者间接继承了PropertyEditorRegistry和 TypeConverter接口。Spring会根据对象实例构造一个BeanWrapperImpl实例，然后将之前CustomEditorConfigurer注册的PropertyEditor复制一份给BeanWrapperImpl实例（这就是BeanWrapper同时又是PropertyEditorRegistry的原因）。这样，当BeanWrapper转换类型、设置对象属性值时，就不会无从下手了。

### Aware接口

当对象实例化完成并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实现了一系列的以Aware命名结尾的接口定义。如果是，则将这些Aware接口定义中规定的依赖注入给当前对象实例。

#### BeanFactory类型的容器的Aware接口：
 
##### org.springframework.beans.factory.BeanNameAware
如果Spring容器检测到当前对象实例实现了该接口，会将该对象实例的bean定义对应的 beanName 设置到当前对象实例。

##### org.springframework.beans.factory.BeanClassLoaderAware 
如果容器检测到当前对象实例实现了该接口，会将 对应加载当前 bean的Classloader注入当前对象实例。默认会使用加载org.springframework.util.ClassUtils类的Classloader。

###### org.springframework.beans.factory.BeanFactoryAware 
在介绍方法注入的时候，我们提到过使用该接口以便每次获取prototype类型bean的不同实例。如果对象声明实现了BeanFactoryAware接口，BeanFactory容器会将自身设置到当前对象实例。这样，当前对象实例就拥有了一个BeanFactory容器的引用，并且可以对这个容器内允许访问的对象按照需要进行访问。

### BeanPostProcessor

BeanPostProcessor会处理容器内所有符合条件的实例化后的对象实例。

该接口声明了两个方法，分别在两个不同的时机执行：
```bash
public interface BeanPostProcessor
{
      //执行于“ BeanPostProcessor 前置处理”
      Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
      
      //执行于“  BeanPostProcessor 后置处理”
      Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```
通常比较常见的使用 BeanPostProcessor 的场景，是处理标记接口实现类，或者为当前对象提供代理实现.

#### 自定义：

假设系统中所有的 IFXNewsListener 实现类需要从某个位置取得相应的服务器连接密码，而且系统中保存的密码是加密的，那么在 IFXNewsListener 发送这个密码给新闻服务器进行连接验证的时候，首先需要对系统中取得的密码进行解密，然后才能发送。

（1）标注需要进行解密的实现类；
（2）实现相应的BeanPostProcessor对符合条件的Bean实例进行处理；
（3）将自定义的BeanPostProcessor注册到容器；

### InitializingBean和init-method

InitializingBean和init-method用于对象的自定义初始化。

#### InitializingBean：

org.springframework.beans.factory.InitializingBean 是容器内部广泛使用的一个对象生命周期标识接口，其定义如下：

```bash
public interface InitializingBean {
      void afterPropertiesSet() throws Exception;
}
```

其作用在于，在对象实例化过程调用过“BeanPostProcessor的前置处理”之后，会接着检测当前对象是否实现了InitializingBean接口，如果是，则会调用其afterPropertiesSet()方法进一步调整对象实例的状态。

#### init-method：

通过init-method，系统中业务对象的自定义初始化操作可以以任何方式命名，而不再受制于InitializingBean的afterPropertiesSet() 。

### DisposableBean与destroy-method

DisposableBean和destroy-method为对象提供了执行自定义销毁逻辑的机会。

容器在最后将检查singleton类型的bean实例，看其是否实现 org.springframework.beans.factory.DisposableBean接口。或者其对应的bean定义是否通过\<bean>的destroy-method属性指定了自定义的对象销毁方法。如果是，就会为该实例注册一个用于对象销毁的回调（Callback），以便在这些singleton类型的对象实例销毁之前，执行销毁逻辑。

这些自定义的对象销毁逻辑，在对象实例初始化完成并注册了相关的回调方法之后，并不会马上执行。回调方法注册后，返回的对象实例即处于使用状态，只有该对象实例不再被使用的时候，才会执行相关的自定义销毁逻辑，此时通常也就是Spring容器关闭的时候。但Spring容器在关闭之前，不会聪明到自动调用这些回调方法。所以，需要我们告知容器，在哪个时间点来执行对象的自定义销毁方法。

#### 对于BeanFactory容器
我们需要在独立应用程序的主程序退出之前，或者其他被认为是合适的情况下（依照应用场景而定）销毁。如果不能在合适的时机调用destroySingletons()，那么所有实现了DisposableBean接口的对象实例或者声明了destroy-method的bean定义对应的对象实例，它们的自定义对象销毁逻辑就形同虚设，因为根本就不会被执行。

#### 对于ApplicationContext容器
与BeanFactory道理相同。但是，AbstractApplicationContext为我们提供了registerShutdownHook()方法，该方法底层使用标准的Runtime类的addShutdownHook()方式来调用相应bean对象的销毁逻辑，从而保证在Java虚拟机退出之前，这些singtleton类型的bean对象实例的自定义销毁逻辑会被执行。当然AbstractApplicationContext注册的shutdownHook不只是调用对象实例的自定义销毁逻辑，也包括ApplicationContext相关的事件发布等。





