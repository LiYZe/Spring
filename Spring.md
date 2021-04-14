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

（后续详细介绍）

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


## BeanFactory
BeanFactory ，顾名思义，就是生产Bean的工厂.作为Spring提供的基本的IoC容器，BeanFactory 可以完成作为IoC Service Provider的所有职责，包括业务对象的注册和对象间依赖关系的绑定。

针对系统和业务逻辑，该如何设计和实现当前系统不受是否引入轻量级容器的影响。前后唯一的不同，就是对象之间依赖关系的解决方式改变了。之前我们的系统业务对象需要自己去“拉”（Pull）所依赖的业务对象，有了 BeanFactory 之类的IoC容器之后，需要依赖什么让 BeanFactory 为我们推过来（Push）就行了。

### 对象注册与依赖绑定方式

#### 直接编码方式

<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114403610-88c7c700-9bd7-11eb-976b-c7563d7ad966.png"/></div> 
```bash
接口：BeanFactory（图书馆）：只定义如何访问容器内管理的Bean的方法
      BeanDefinitionRegistry（图书馆的书架）：定义抽象了Bean的注册逻辑，在BeanFactory 的实现中担当Bean注册管理的角色
      BeanDefinition：负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。
实现类：Default-ListableBeanFactory：负责具体Bean的注册以及管理工作。
        RootBeanDefinition：BeanDefinition的主要实现类
        ChildBeanDefinition：BeanDefinition的主要实现类
```

#### 外部配置文件方式
Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。

```bash
1.根据不同的外部配置文件格式，给出相应的 BeanDefinitionReader 实现类（完成大部分工作，包括解析文件格式、装配BeanDefinition等）
2.由 BeanDefinitionReader 的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition 
3.将映射后的 BeanDefinition注册到一个BeanDefinitionRegistry（负责保存）
4. BeanDefinitionRegistry即完成Bean的注册和加载。
```

##### Properties

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

#### XML

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
##### \<beans>和\<bean>

\<bean>    

```bash
所有注册到容器的业务对象，在Spring中称之为Bean。所以，每一个对象在XML中的映射也自然而然地对应一个叫做 <bean> 的元素。
```

\<beans>

```bash
<bean> 的元素组织起来的，就叫做 <beans>,多个 <bean> 组成一个 <beans>。
```

属性（attribute）：

default-lazy-init：
```bash
其值可以指定为 true 或者 false ，默认值为 false 。用来标志是否对所有的 <bean> 进行延迟初始化。
```
default-autowire：
```bash
可以取值为no、byName、byType、constructor以及autodetect。默认值为no，如果使用自动绑定的话，用来标志全体bean使用哪一种默认
绑定方式。
```
default-dependency-check：
```bash
可以取值 none、objects、simple以及all，默认值为none，即不做依赖检查。
```
default-init-method:
```bash
如果所管辖的 <bean> 按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用在每一个<bean>
上都重复单独指定。
```
default-destroy-method:
```bash
与 default-init-method相对应，如果所管辖的bean有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定。
```
\<beans> 是XML配置文件中最顶层的元素，它下面可以包含0或者1个\<description> 和多个\<bean>以及 \<import> 或者\<alias>。
<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114528318-7b641880-9c7b-11eb-971f-5554115805a0.png"/></div> 

\<description>
```bash
可以通过 <description> 在配置的文件中指定一些描述性的信息。通常情况下，该元素是省略的。
```     
\<import>
```bash
通常情况下，可以根据模块功能或者层次关系，将配置信息分门别类地放到多个配置文件中。在想加载主要配置文件，并将主要配置
文件所依赖的配置文件同时加载时，可以在这个主要的配置文件中通过 <import> 元素对其所依赖的配置文件进行引用。
```
\<alias>
```bash
可以通过 <alias> 为某些 <bean> 起一些“外号”（别名），通常情况下是为了减少输入。
```

###### 配置方式

```bash
<bean id="djNewsListener" class="..impl.DowJonesNewsListener"></bean>
```
id：

```bash
id属性来指定当前注册对象的beanName是什么。这里，通过id指定beanName为djNewsListener。实际上，并非任何情况下都需要
指定每个<bean>的id，有些情况下，id可以省略
```
name:

除了可以使用 id 来指定\<bean>在容器中的标志，还可以使用 name 属性来指定\<bean> 的别名（alia）。
```bash
<bean id="djNewsListener"
      name="/news/djNewsListener,dowJonesNewsListener"
      class="..impl.DowJonesNewsListener">
</bean>
```

id和name区别：

```bash
与 id 属性相比， name 属性的灵活之处在于， name 可以使用 id 不能使用的一些字符，比如/。而且还可以通过逗号、空格
或者冒号分割指定多个 name 。 name 的作用跟使用 <alias> 为 id 指定多个别名基本相同。
```

class：
```bash
每个注册到容器的对象都需要通过 <bean> 元素的 class 属性指定其类型，否则，容器可不知道这个对象到底是何方神圣。
在大部分情况下，该属性是必须的,仅在少数情况下不需要指定。
```

###### 表达依赖

1.构造方法注入XML：

按照Spring的IoC容器配置格式，要通过构造方法注入方式，为当前业务对象注入其所依赖的对象，需要使用 \<constructor-arg>。
      
正常情况下，如以下代码所示：

```bash
<bean id="djNewsProvider" class="..FXNewsProvider">
      <constructor-arg>
            <ref bean="djNewsListener"/>
      </constructor-arg>
      <constructor-arg>
            <ref bean="djNewsPersister"/>
      </constructor-arg>
</bean>
      
//简化形式 
<bean id="djNewsProvider" class="..FXNewsProvider">
      <constructor-arg ref="djNewsListener"/>
      <constructor-arg ref="djNewsPersister"/>
</bean>
```
```bash
type属性:使用type属性指定构造方法的参数类型。
index属性：使用index属性指定构造方法的某个位置的参数类型，取值从0开始。
```

2.setter方法注入的XML：

Spring为setter方法注入提供了\<property>元素。\<property>有一个name属性（attribute），用来指定该\<property>将会注入的对象所对应的实例变量名称。之后通过value或者ref属性或者内嵌的其他元素来指定具体的依赖对象引用或者值。如果只是使用 <property> 进行依赖注入的话，请确保你的对象提供了默认的构造方法（无参构造函数）。

如以下代码所示：
```bash
<bean id="djNewsProvider" class="..FXNewsProvider">
      <property name="newsListener">
            <ref bean="djNewsListener"/>
      </property>
      <property name="newPersistener">
            <ref bean="djNewsPersister"/>
      </property>
</bean>

//简化形式
<bean id="djNewsProvider" class="..FXNewsProvider">
      <property name="newsListener" ref="djNewsListener"/>
      <property name="newPersistener" ref="djNewsPersister"/>
<
/bean>
```
\<property>和\<constructor-arg> 中可用的配置项:

3.Spring提供了元素供我们使用，这包括bean、ref、idref、value、null、list、set、map、props.

\<value>:
```bash
可以通过 value 为主体对象注入简单的数据类型，不但可以指定 String 类型的数据，而且可以指定其他Java语言中的原始类型以及
它们的包装器（wrapper）类型，比如 int、Integer 等。容器在注入的时候，会做适当的转换工作。
```

\<ref>
```bash
使用ref来引用容器中其他的对象实例，可以通过ref的local、parent和bean属性来指定引用
的对象的beanName 是什么。下面没有其他子元素可用。

local：只能指定与当前配置的对象在同一个配置文件的对象定义的名称；
parent：则只能指定位于当前容器的父容器中定义的对象引用；
bean：则基本上通吃，所以，通常情况下，直接使用bean来指定对象引用就可以了。
```

\<idref>
```bash
如果要为当前对象注入所依赖的对象的名称，而不是引用，那么通常情况下，可以使用 <value> 来达到这个目的。
使用idref，容器在解析配置时就可以检查这个beanName是否存在，而不用等到运行时才发现这个beanName对应的对象实例不存在。
```

内部\<bean>
```bash
使用 <ref>可以引用容器中独立定义的对象定义。但有，可能我们所依赖的对象只有当前一个对象引用，或者某个对象定义我们不想其他对象通过<ref>引用
到它。这时，可使用内嵌的<bean> ，将这个私有的对象定义仅局限在当前对象。内部 <bean>的配置只是在位置上有所差异，但配置项上与其他的<bean>是
没有任何差别的。也就是，<bean>内嵌的所有元素，内部<bean>的<bean>同样可以使用。如果内部<bean>对应的对象还依赖于其他对象，你完全可以像其他
独立的<bean>定义一样为其配置相关依赖，没有任何差别。
```

\<list>
```bash
<list>对应注入对象类型为java.util.List 及其子类或者数组类型的依赖对象。通过<list>可以有序地为当前对象注入以collection形式声明的依赖。
```

\<set>
```bash
如果说<list>可以帮你有序地注入一系列依赖的话，那么<set> 就是无序的，而且，对于 set 来说，元素的顺序本来就是无关紧要的。
<set>对应注入Java Collection中类型为java.util.Set 或者其子类的依赖对象。
```

\<map>
```bash
映射（map）可以通过指定的键（key）来获取相应的值。 <map> 与 <list> 和 <set> 的相同点在于，都是为主体对象注入Collection类型的依赖，
不同点在于它对应注入java.util.Map 或者其子类类型的依赖对象。

对于 <map> 来说，它可以内嵌任意多个 <entry> ，每一个 <entry> 都需要为其指定一个键和一个值，就跟真正的 java.util.Map 所要求的一样。

指定entry的键。可以使用 <entry> 的属性key或者key-ref来指定键，也可以使用<entry>的内嵌元素 <key> 来指定键,但两种方式可以达到相同的
效果。在<key>内部，可以使用以上提到的任何元素来指定键，从简单的<value>到复杂的Collection。

指定 entry 对应的值。 <entry> 内部可以使用的元素，除了 <key> 是用来指定键的，其他元素可以任意使用，来指定 entry 对应的值。
除了之前提到的那些元素，还包括马上就要谈到的<props> 。如果对应的值只是简单的原始类型或者单一的对象引用，也可以直接使用
<entry>的 value 或者 value-ref 这两个属性来指定，从而省却多敲入几个字符的工作量。
```
\<props>
```bash
<props>是简化后了的 <map> ，或者说是特殊化的 map ，该元素对应配置类型为java.util.Properties 的对象依赖。因为 Properties 
只能指定 String 类型的键（key）和值，所以，<props> 的配置简化很多，只有固定的格式。每个<props> 可以嵌套多个<prop>，
每个<prop>通过其 key 属性来指定键，在<prop>内部直接指定其所对应的值。<prop> 内部没有任何元素可以使用，只能指定字符串，
这个是由 java.util.Properties 的语意决定的。
```

\<null/>
```bash
<null/> 是最简单的一个元素，因为它只是一个空元素，而且通常使用到它的场景也不是很多。对于String 类型来说，如果通过value
以这样的方式指定注入，即<value></value>得到的结果是"" ，而不是null。所以，如果需要为这个string对应的值注入null的话，请使用<null/>。
```

4.depends-on

系统中所有需要日志记录的类，都需要在这些类使用之前首先初始化log4j。那么，就会非显式地依赖于 SystemConfigurationSetup 的静态初始化块。如果ClassA 需要使用log4j，那么就必须在bean定义中使用 depends-on 来要求容器在初始化自身实例之前首先实例化SystemConfigurationSetup，以保证日志系统的可用，如下代码演示的正是这种情况：
```bash
<bean id="classAInstance" class="...ClassA"depends-on="configSetup"/>
<bean id="configSetup" class="SystemConfigurationSetup"/>
```
你可以在ClassA的 depends-on 中通过逗号分割各个 beanName。

5.autowire

通过 <bean> 的autowire属性，可以指定当前bean定义采用某种类型的自动绑定模式。这样，无需手工明确指定该bean定义相关的依赖关系，从而也可以免去一些手工输入的工作量。Spring提供了5种自动绑定模式，即no、byName 、byType、constructor和autodetect，下面是它们的具体介绍：
     
byName
```bash
按照类中声明的实例变量的名称，与XML配置文件中声明的bean定义的 beanName 的值进行匹配，相匹配的bean定义将被自动绑定到当前实例变量上。
这种方式对类定义和配置的bean定义有一定的限制。

<bean id="fooBean" class="...Foo" autowire="byName">
</bean>
<bean id="emphasisAttribute" class="...Bar">
</bean>

需要注意两点：
第一，我们并没有明确指定 fooBean 的依赖关系，而仅指定了它的 autowire 属性为byName；
第二，第二个bean定义的 id 为 emphasisAttribute ，与 Foo 类中的实例变量名称相同。
```

byType
```bash
如果指定当前bean定义的 autowire 模式为 byType ，那么，容器会根据当前bean定义类型，分析其相应的依赖对象类型，然后到容器所管理的所有
bean定义中寻找与依赖对象类型相同的bean定义，然后将找到的符合条件的bean自动绑定到当前bean定义。如果没有找到，则不做设置。但如果找到
多个，容器会告诉你它解决不了“该选用哪一个”的问题，你只好自己查找原因，并自己修正该问题。所以，byType只能保证，在容器中只存在一个符
合条件的依赖对象的时候才会发挥最大的作用，如果容器中存在多个相同类型的bean定义，那么，不好意思，采用手动明确配置吧！

指定 byType 类型的 autowire 模式与 byName 没什么差别，只是 autowire 的值换成 byType 而已。
```

constructor
```bash
constructor 类型则针对构造方法参数的类型而进行的自动绑定，它同样是 byType 类型的绑定模式。不过，constructor是匹配构造方法的参数类型，
而不是实例属性的类型。与 byType 模式类似，如果找到不止一个符合条件的bean定义，那么，容器会返回错误。使用上也与 byType 没有太大差别，
只不过是应用到需要使用构造方法注入的bean定义之上。

<bean id="foo" class="...Foo" autowire="constructor"/>
<bean id="bar" class="...Bar">
</bean>
```

autodetect
```bash
这种模式是 byType 和 constructor 模式的结合体，如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，
会使用constructor模式。当然，如果通过构造方法注入绑定后还有其他属性没有绑定，容器也会使用byType对剩余的对象属性进行自动绑定。
```

6.dependency-check

使用每个\<bean> 的 dependency-check 属性对其所依赖的对象进行最终检查。该功能主要与自动绑定结合使用，可以保证当自动绑定完成后，最终确认每个对象所依赖的对象是否按照所预期的那样被注入。该功能可以帮我们检查每个对象某种类型的所有依赖是否全部已经注入完成，不过可能无法细化到具体的类型检查。但某些时候，使用setter方法注入就是为了拥有某种可以设置也可以不设置的灵活性所以，这种依赖检查并非十分有用，尤其是在手动明确绑定依赖关系的情况下。
      
基本上有如下4种类型的依赖检查:
none 
```bash
不做依赖检查。将 dependency-check 指定为 none 跟不指定这个属性等效，默认情况下，容器以此为默认值。
```

simple 
```bash
如果将 dependency-check 的值指定为 simple ，那么容器会对简单属性类型以及相关的collection进行依赖检查，对象引用类型的依赖除外。
```

object 
```bash
只对对象引用类型依赖进行检查。
```

all 
```bash
将 simple 和 object 相结合，也就是说会对简单属性类型以及相应的collection和所有对象引用类型的依赖进行检查。
```
7.lazy-init

延迟初始化（lazy-init）这个特性的作用，主要是可以针对 ApplicationContext 容器的bean初始化行为施以更多控制。我们不想某些bean定义在容器启动后就直接实例化，想改变某个或者某些bean定义在ApplicationContext 容器中的默认实例化时机。这时，就可以通过\<bean>的lazy-init属性来控制这种初始化行为。
```bash
<bean id="lazy-init-bean" class="..." lazy-init="true"/>
<bean id="not-lazy-init-bean" class="..."/>
```

###### 继承
```bash
<bean id="newsProviderTemplate" abstract="true">
      <property name="newPersistener">
            <ref bean="djNewsPersister"/>
      </property>
</bean>
<bean id="superNewsProvider" parent="newsProviderTemplate"class="..FXNewsProvider">
      <property name="newsListener">
            <ref bean="djNewsListener"/>
      </property>
</bean>
<bean id="subNewsProvider" parent="newsProviderTemplate"class="..SpecificFXNewsProvider">
      <property name="newsListener">
            <ref bean="specificNewsListener"/>
      </property>
</bean>
```
我们在声明 subNewsProvider 的时候，使用了 parent 属性，将其值指定为 superNewsProvider ，这样就继承了 superNewsProvider 定义的默认值，只需要将特定的属性进行更改，而不要全部又重新定义一遍。parent 属性还可以与 abstract 属性结合使用，达到将相应bean定义模板化的目的。

abstract
```bash
newsProviderTemplate 的 bean 定义通过 abstract 属性声明为 true ，说明这个bean定义不需要实例化。实际上，这就是之前提到的可以不
指定class属性的少数场景之一。容器在初始化对象实例的时候，不会关注将abstract 属性声明为 true 的bean定义。如果你不想容器在初始化
的时候实例化某些对象，那么可以将其 abstract 属性赋值true ，以避免容器将其实例化。
```

###### scope

scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的scope之前，生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁Spring的IoC容器这些对象。

Spring容器最初提供了两种bean的scope类型：singleton和prototype

singleton
```bash
标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，
并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
```

prototype
```bash
针对声明为拥有prototype scope的bean定义，容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。只要准
备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理
工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了
```

自定义scope
```bash
首先需要给出一个 Scope 接口的实现类，接口定义中的4个方法并非都是必须的，但 get 和 remove 方法必须实现。

public interface Scope {
      Object get(String name, ObjectFactory objectFactory);
      Object remove(String name);
      void registerDestructionCallback(String name, Runnable callback);
      String getConversationId();
}
```

###### 工厂方法与 FactoryBean

通过使用工厂方法（Factory Method）模式，提供一个工厂类来实例化具体的接口实现类，这样，主体对象只需要依赖工厂类，具体使用的实现类有变更的话，只是变更工厂类，而主体对象不需要做任何变动。

1.静态工厂方法（Static Factory Method）
```bash
<bean id="foo" class="...Foo">
      <property name="barInterface">
            <ref bean="bar"/>
      </property>
</bean>
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```
class 指定静态方法工厂类， factory-method 指定工厂方法名称，然后，容器调用该静态方法工厂类的指定工厂方法（getInstance），并返回方法调用后的结果，即BarInterfaceImpl的实例。也就是说，为 foo 注入的 bar 实际上是 BarInterfaceImpl 的实例，即方法调用后的结果，而不是静态工厂方法类（StaticBarInterfaceFactory）。

2.非静态工厂方法（Instance Factory Method）
```bash
<bean id="foo" class="...Foo">
      <property name="barInterface">
            <ref bean="bar"/>
      </property>
</bean>
<bean id="barFactory" class="...NonStaticBarInterfaceFactory"/>
<bean id="bar" factory-bean="barFactory" factory-method="getInstance"/>
```
NonStaticBarInterfaceFactory 是作为正常的bean注册到容器的，而 bar 的定义则与静态工厂方法的定义有些不同。现在使用 factory-bean 属性来指定工厂方法所在的工厂类实例，而不是通过class 属性来指定工厂方法所在类的类型。指定工厂方法名则相同，都是通过 factory-method 属性进行的。

3. FactoryBean

FactoryBean 是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口，请不要将其与容器名称BeanFactory相混淆。它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂（Factory）。

以下两种情况可以实现 org.spring-framework.beans.factory.FactoryBean 接口，给出自己的对象实例化逻辑代码。

1.XML配置过于复杂，使我们宁愿使用Java代码来完成这个实例化过程的时候

2.某些第三方库不能直接注册到Spring容器的时候

```bash
public interface FactoryBean {
      Object getObject() throws Exception;
      Class getObjectType();
      boolean isSingleton();
}

getObject()方法会返回该 FactoryBean “生产”的对象实例，我们需要实现该方法以给出自己的对象实例化逻辑； 

getObjectType()方法仅返回 getObject() 方法所返回的对象的类型，如果预先无法确定，则返回 null；

isSingleton()方法返回结果用于表明，工厂方法(getObject())所“生产”的对象是否要以singleton形式存在于容器中。
如果以singleton形式存在，则返回 true ，否则返回 false；
```

###### 方法注入（Method-Injection）以及方法替换（Method-Replacement）

bean的scope的使用“陷阱”，特别是prototype在容器中的使用。当使用prototype时，每个对象都应该是新的独立个体，但是当容器注入一个实例的时候，类有可能会一直持有这个实例的引用，虽然每次都返回了实例，但是都是第一次所注入的实例，没有重新注入实例。

为此Spring使用了两个方法：方法注入（Method-Injection）以及方法替换（Method-Replacement）

1.方法注入

让 getNewsBean 方法声明符合规定的格式，并在配置文件中通知容器，当该方法被调用的时候，每次返回指定类型的对象实例即可。方法声明需要符合的规格定义如下：
```bash
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```
该方法必须能够被子类实现或者覆写，因为容器会为我们要进行方法注入的对象使用Cglib动态生成一个子类实现，从而替代当前对象。

2.方法替换

（1）给出org.springframework.beans.factory.support.MethodReplacer的实现类，在这个类中实现将要替换的方法逻辑。
（2）有了要替换的逻辑之后，我们就可以把这个逻辑通过 <replaced-method> 配置到 FXNewsProvider的bean定义中，使其生效。

3.殊途同归

除了使用方法注入来达到“每次调用都让容器返回新的对象实例”的目的，还可以使用其他方式达到相同的目的。下面给出其他两种解决类似问题的方法

（1）使用 BeanFactoryAware 接口
```bash
Spring框架提供了一个 BeanFactoryAware 接口，容器在实例化实现了该接口的bean定义的过程中，会自动将容器本身注入该bean。这样，该bean就
持有了它所处的 BeanFactory 的引用。 BeanFactory-Aware 的定义如下代码所示：

public interface BeanFactoryAware {
void setBeanFactory(BeanFactory beanFactory) throws BeansException;
}
```
（2）使用 ObjectFactoryCreatingFactoryBean
```bash
ObjectFactoryCreatingFactoryBean是Spring提供的一个FactoryBean实现，返回一个ObjectFactory实例。从ObjectFactoryCreatingFactoryBean 
返回的这个ObjectFactory实例可以为 我们返回容器管理的相关对象。 实际上，ObjectFactoryCreatingFactoryBean实现了BeanFactoryAware接口，
它返回的ObjectFactory实例只是特定于与Spring容器进行交互的一个实现而已。使用它的好处就是，隔离了客户端对象对BeanFactory的直接引用。
```

#### IoC容器功能实现

Spring的IoC容器所起的作用：以某种方式加载Configuration Metadata（通常也就是XML格式的配置信息），根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。

实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段：1.容器启动阶段 2.Bean实例化阶段
<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114668130-f0def000-9d32-11eb-975c-2f18b8b16f75.png"/></div> 

##### 容器启动阶段：

（只是根据图纸装配生产线）

首先通过某种途径加载Configuration MetaData，随后依赖某些工具类（ BeanDefinitionReader ）对加载的进行解析和分析，并将分析后的信息编组为相应的 BeanDefinition ，最后把这些保存了bean定义必要信息的BeanDefinition ，注册到相应的 BeanDefinitionRegistry ，容器启动工作完成。

<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114668657-9003e780-9d33-11eb-87f2-a6edc9b934a6.png"/></div>

######  BeanFactoryPostProcessor 

容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的 BeanDefinition 所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的 BeanDefinition 做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。

自定义：
```bash
自定义实现 BeanFactoryPostProcessor，通常我们需要实现org.springframework.beans.factory.config.BeanFactoryPostProcessor接口。
同时，因为一个容器可能拥有多个Bean FactoryPostProcessor，这个时候可能需要实现类同时实现Spring的org.springframework.core.Ordered
接口，以保证各个 BeanFactoryPostProcessor 可以按照预先设定的顺序执行
```

常用的BeanFactoryPostProcessor实现类：
```bash
1.org.springframework.beans.factory.config.PropertyPlaceholderConfigurer:
PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder），并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。

2.org.springframework.beans.factory.config.PropertyOverrideConfigurer
可以通过占位符，来明确表明bean定义中的property与properties文件中的各配置项之间的对应关系。通过PropertyOverrideConfigurer
可以对容器中配置的任何你想处理的bean定义的property信息进行覆盖替换。

beanName.propertyName=value
```

数据类型转换的补充：

通过org.springframework.beans.factory.config.CustomEditorConfigurer 来注册自定义的 PropertyEditor以补助容器中默认的PropertyEditor。

CustomEditorConfigurer 是另一种类型的 BeanFactoryPostProcessor 实现，它只是辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动。

两种应用：
```bash
基本的IoC容器BeanFactory:手动方式应用所有的BeanFactoryPostProcessor
较为先进的容器 ApplicationContext:将相应 BeanFactoryPostProcessor 实现类添加到配置文件，ApplicationContext 将自动识别并应用它
```
##### Bean实例化阶段：

（使用装配好的生产线来生产具体的产品）

当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据注册的BeanDefinition 所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。

BeanFactory的getBeean法可以被客户端对象隐式地调用：
```bash
1.当对象A被请求而需要第一次实例化的时候，如果它所依赖的对象B之前同样没有被实例化，那么容器会先实例化对象A所依赖的对象。
这时容器内部就会首先实例化对象B，以及对象A依赖的其他还没有实例化的对象。
（B被隐式调用）

2.ApplicationContext会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean() 。
```

之所以说 getBean() 方法是有可能触发Bean实例化阶段的活动，是因为只有当对应某个bean定义的 getBean() 方法第一次被调用时，不管是显式的还是隐式的，Bean实例化阶段的活动才会被触发，第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例（prototype类型bean除外）。当getBean() 方法内部发现该bean定义之前还没有被实例化之后，会通过 createBean() 方法来进行具体的对象实例化，实例化过程如图所示：

<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114673249-9052b180-9d38-11eb-8c10-23e7a5add8d9.png"/></div>

1.Bean的实例化、设置对象属性与BeanWrapper

容器在内部实现的时候，采用“策略模式（Strategy Pattern）”来决定采用何种方式初始化bean实例。通常，可以通过反射或者CGLIB动态字节码生成来初始化相应的bean实例或者动态生成其子类。

CglibSubclassingInstantiationStrategy 继承了SimpleInstantiationStrategy的以反射方式实例化对象的功能，并且通过CGLIB的动态字节码生成功能，该策略实现类可以动态生成某个类的子类，进而满足了方法注入所需的对象实例化需求。默认情况下，容器内部采用的是 CglibSubclassingInstantiationStrategy。

图中的第一步“实例化bean对象”：

容器只要根据相应bean定义的 BeanDefintion 取得实例化信息，结合 CglibSubclassingInstantiationStrategy 及不同的bean定义类型，就可以返回实例化完成的对象实例。但是，返回方式上有些“点缀”。不是直接返回构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例进行包裹，返回相应的 BeanWrapper 实例。

图中的第二步“设置对象属性”：

BeanWrapper 接口通常在Spring框架内部使用，它有一个实现类 org.springframework.beans.BeanWrapperImpl 。其作用是对某个bean进行“包裹”，然后对这个“包裹”的bean进行操作，比如设置或者获取bean的相应属性值。而在第一步结束后返回 BeanWrapper 实例而不是原先的对象实例。

BeanWrapper
```bash
BeanWrapper 定义继承了 org.springframework.beans.PropertyAccessor 接口，可以以统一的方式对对象属性进行访问； BeanWrapper定义同时
又直接或者间接继承了 PropertyEditorRegistry和 TypeConverter 接口。Spring会根据对象实例构造一个 BeanWrapperImpl 实例，然后将之前 
CustomEditor-Configurer 注册的PropertyEditor 复制一份给BeanWrapperImpl实例（这就是BeanWrapper同时又是 PropertyEditorRegistry的
原因）。这样，当BeanWrapper转换类型、设置对象属性值时，就不会无从下手了。
```

2.Aware接口

当对象实例化完成并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实现了一系列的以 Aware 命名结尾的接口定义。如果是，则将这些 Aware 接口定义中规定的依赖注入给当前对象实例。

BeanFactory类型的容器的Aware接口：
 
org.springframework.beans.factory.BeanNameAware
```bash
如果Spring容器检测到当前对象实例实现了该接口，会将该对象实例的bean定义对应的 beanName 设置到当前对象实例。
```
org.springframework.beans.factory.BeanClassLoaderAware 
```bash
如果容器检测到当前对象实例实现了该接口，会将 对应加载当前 bean的Classloader注入当前对象实例。
默认会使用加载org.springframework.util.ClassUtils类的Classloader。
```
org.springframework.beans.factory.BeanFactoryAware 
```bash
在介绍方法注入的时候，我们提到过使用该接口以便每次获取prototype类型bean的不同实例。如果对象声明实现了BeanFactoryAware接口，
BeanFactory容器会将自身设置到当前对象实例。这样，当前对象实例就拥有了一个BeanFactory容器的引用，并且可以对这个容器内
允许访问的对象按照需要进行访问。
```

3.BeanPostProcessor

BeanPostProcessor会处理容器内所有符合条件的实例化后的对象实例。该接口声明了两个方法，分别在两个不同的时机执行
```bash
public interface BeanPostProcessor
{
Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
//执行于“ BeanPostProcessor 前置处理”

Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
//执行于“  BeanPostProcessor 后置处理”
}
```
通常比较常见的使用 BeanPostProcessor 的场景，是处理标记接口实现类，或者为当前对象提供代理实现.

自定义：

假设系统中所有的 IFXNewsListener 实现类需要从某个位置取得相应的服务器连接密码，而且系统中保存的密码是加密的，那么在 IFXNewsListener 发送这个密码给新闻服务器进行连接验证的时候，首先需要对系统中取得的密码进行解密，然后才能发送。

（1）标注需要进行解密的实现类
（2）实现相应的 BeanPostProcessor 对符合条件的Bean实例进行处理
（3） 将自定义的 BeanPostProcessor 注册到容器

4. InitializingBean和init-method

InitializingBean 和 init-method 用于对象的自定义初始化

InitializingBean：

org.springframework.beans.factory.InitializingBean 是容器内部广泛使用的一个对象生命周期标识接口，其定义如下：
```bash
public interface InitializingBean {
void afterPropertiesSet() throws Exception;
}
```
其作用在于，在对象实例化过程调用过“ BeanPostProcessor 的前置处理”之后，会接着检测当前对象是否实现了 InitializingBean 接口，如果是，则会调用其 afterPropertiesSet() 方法进一步调整对象实例的状态。

init-method：

通过 init-method，系统中业务对象的自定义初始化操作可以以任何方式命名，而不再受制于InitializingBean的 afterPropertiesSet() 。

5. DisposableBean 与 destroy-method

容器在最后将检查singleton类型的bean实例，看其是否实现了 org.springframework.beans.factory.DisposableBean 接口。或者其对应的bean定义是否通过 <bean> 的 destroy-method 属性指定了自定义的对象销毁方法。如果是，就会为该实例注册一个用于对象销毁的回调（Callback），以便在这些singleton类型的对象实例销毁之前，执行销毁逻辑。

 DisposableBean 和destroy-method 为对象提供了执行自定义销毁逻辑的机会。

这些自定义的对象销毁逻辑，在对象实例初始化完成并注册了相关的回调方法之后，并不会马上执行。回调方法注册后，返回的对象实例即处于使用状态，只有该对象实例不再被使用的时候，才会执行相关的自定义销毁逻辑，此时通常也就是Spring容器关闭的时候。但Spring容器在关闭之前，不会聪明到自动调用这些回调方法。所以，需要我们告知容器，在哪个时间点来执行对象的自定义销毁方法。

对于 BeanFactory 容器来说。我们需要在独立应用程序的主程序退出之前，或者其他被认为是合适的情况下（依照应用场景而定）销毁。如果不能在合适的时机调用 destroySingletons() ，那么所有实现了 DisposableBean 接口的对象实例或者声明了 destroy-method 的bean定义对应的对象实例，它们的自定义对象销毁逻辑就形同虚设，因为根本就不会被执行。

对于 ApplicationContext 容器来说。道理是一样的。但 AbstractApplicationContext 为我们提供了 registerShutdownHook() 方法，该方法底层使用标准的 Runtime 类的 addShutdownHook() 方式来调用相应bean对象的销毁逻辑，从而保证在Java虚拟机退出之前，这些singtleton类型的bean对象实例的自定义销毁逻辑会被执行。当然 AbstractApplicationContext 注册的 shutdownHook 不只是调用对象实例的自定义销毁逻辑，也包括 ApplicationContext 相关的事件发布等。



### 注解方式

通过注解标注的方式为 FXNewsProvider 注入所需要的依赖，现在可以使用 @Autowired 以及 @Component 对相关类进行标记。再向Spring的配置文件中增加一个“触发器”，使用 @Autowired 和 @Component 标注的类就能获得依赖对象的注入了。

#### @Autowired

它的存在将告知Spring容器需要为当前对象注入哪些依赖对象。

#### @Component

配合Spring 2.5中新的classpath-scanning功能使用的。







