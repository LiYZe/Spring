# BeanFactory

BeanFactory ，顾名思义，就是生产Bean的工厂.作为Spring提供的基本的IoC容器，BeanFactory 可以完成作为IoC Service Provider的所有职责，包括业务对象的注册和对象间依赖关系的绑定。

针对系统和业务逻辑，该如何设计和实现当前系统不受是否引入轻量级容器的影响。前后唯一的不同，就是对象之间依赖关系的解决方式改变了。之前我们的系统业务对象需要自己去“拉”（Pull）所依赖的业务对象，有了 BeanFactory 之类的IoC容器之后，需要依赖什么让 BeanFactory 为我们推过来（Push）就行了。

# 对象注册与依赖绑定方式

## 直接编码方式

<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/114403610-88c7c700-9bd7-11eb-976b-c7563d7ad966.png"/></div> 

```bash
接口：BeanFactory（图书馆）：只定义如何访问容器内管理的Bean的方法，保存实例化阶段将要用的必要信息。

     BeanDefinitionRegistry（图书馆的书架）：定义抽象了Bean的注册逻辑，在BeanFactory的实现中担当Bean注册管理的角色。
      
     BeanDefinition：负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。
      
实现类：DefaultListableBeanFactory：负责具体Bean的注册以及管理工作。

        RootBeanDefinition：BeanDefinition的主要实现类
        
        ChildBeanDefinition：BeanDefinition的主要实现类
```

## 外部配置文件方式

Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。

```bash
1.根据不同的外部配置文件格式，给出相应的 BeanDefinitionReader 实现类（完成大部分工作，包括解析文件格式、装配BeanDefinition等）
2.由 BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition
3.将映射后的BeanDefinition注册到一个BeanDefinitionRegistry（负责保存）
4. BeanDefinitionRegistry即完成Bean的注册和加载。
```

### Properties

Spring提供了org.springframework.beans.factory.support.PropertiesBeanDefinitionReader 类用于Properties格式配置文件的加载。

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
djNewsProvider作为beanName，后面通过.(class)表明对应的实现类是什么。通过在表示beanName的名称后添加.$[number]后缀的形式，来表示当前beanName对应的对象需要通过构造方法注入的方式注入相应依赖对象。在这里，我们分别将构造方法的第一个参数和第二个参数对应到djListener和djPersister。

（注意：(ref)用来表示所依赖的是引用对象，而不是普通的类型。如果不加(ref)，PropertiesBeanDefinitionReader会将djListener和djPersiste 作为简单的String类型进行注入，产生异常。）

setter方法与构造方法的区别：
```bash
构造方法注入无法通过参数名称来标识注入的确切位置，而setter方法注入则可以通过属性名称来明确标识注入。
```

### XML

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
Spring同样为XML格式的配置文件提供了现成的BeanDefinitionReader实现，即XmlBeanDefinitionReader。XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition ，并加载到相应的BeanDefinitionRegistry中（在这里是 DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了。

#### \<beans>和\<bean>

##### \<bean>    

```bash
所有注册到容器的业务对象，在Spring中称之为Bean。所以，每一个对象在XML中的映射也自然而然地对应一个叫做<bean>的元素。
```

##### \<beans>

```bash
<bean>的元素组织起来的，就叫做<beans>,多个<bean>组成一个<beans>。
```

###### 属性（attribute）：

对所辖的<bean>进行统一的默认行为设置。

default-lazy-init：
```bash
其值可以指定为true或者false，默认值为 false。用来标志是否对所有的 <bean> 进行延迟初始化。
```
default-autowire：
```bash
可以取值为no、byName、byType、constructor以及autodetect。默认值为no，如果使用自动绑定的话，用来标志全体bean使用哪一种默认
绑定方式。
```
default-dependency-check：
```bash
可以取值none、objects、simple以及all，默认值为none，即不做依赖检查。
```
default-init-method:
```bash
如果所管辖的<bean> 按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用在每一个<bean>
上都重复单独指定。
```
default-destroy-method:
```bash
与 default-init-method相对应，如果所管辖的bean有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定。
```
###### \<description>、\<import>、\<alias>

\<beans>是XML配置文件中最顶层的元素，它下面可以包含0或者1个\<description>和多个\<bean>以及\<import>或者\<alias>。

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

##### 对象配置方式

```bash
<bean id="djNewsListener" class="..impl.DowJonesNewsListener"></bean>
```
###### id：

```bash
id属性来指定当前注册对象的beanName是什么。这里，通过id指定beanName为djNewsListener。实际上，并非任何情况下都需要
指定每个<bean>的id，有些情况下，id可以省略。
```
###### name:

除了可以使用id来指定\<bean>在容器中的标志，还可以使用name属性来指定\<bean>的别名（alia）。
```bash
<bean id="djNewsListener"
      name="/news/djNewsListener,dowJonesNewsListener"
      class="..impl.DowJonesNewsListener">
</bean>
```

###### id和name区别：

```bash
与id属性相比，name属性的灵活之处在于，name可以使用id不能使用的一些字符，比如/。而且还可以通过逗号、空格
或者冒号分割指定多个name。name的作用跟使用<alias>为id指定多个别名基本相同。
```

###### class：
```bash
每个注册到容器的对象都需要通过<bean>元素的 class 属性指定其类型，否则，容器可不知道这个对象到底是何方神圣。
在大部分情况下，该属性是必须的,仅在少数情况下不需要指定。
```

##### 表达依赖（传参）

###### 构造方法注入XML：

按照Spring的IoC容器配置格式，要通过构造方法注入方式，为当前业务对象注入其所依赖的对象，需要使用\<constructor-arg>。
      
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
type属性：使用type属性指定构造方法的参数类型。

index属性：使用index属性指定构造方法的某个位置的参数类型，取值从0开始。
```

###### setter方法注入的XML：

Spring为setter方法注入提供了\<property>元素。\<property>有一个name属性（attribute），用来指定该\<property>将会注入的对象所对应的实例变量名称。之后通过value或者ref属性或者内嵌的其他元素来指定具体的依赖对象引用或者值。如果只是使用\<property> 进行依赖注入的话，请确保你的对象提供了默认的构造方法（无参构造函数）。

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
###### \<property>和\<constructor-arg> 中可用的配置项:

Spring提供了元素供我们使用，这包括bean、ref、idref、value、null、list、set、map、props。

\<value>:
```bash
可以通过value为主体对象注入简单的数据类型，不但可以指定String类型的数据，而且可以指定其他Java语言中的原始类型以及
它们的包装器（wrapper）类型，比如int、Integer等。容器在注入的时候，会做适当的转换工作。
```

\<ref>
```bash
使用ref来引用容器中其他的对象实例，可以通过ref的local、parent和bean属性来指定引用的对象的beanName是什么。
下面没有其他子元素可用。

local：只能指定与当前配置的对象在同一个配置文件的对象定义的名称；
parent：则只能指定位于当前容器的父容器中定义的对象引用；
bean：则基本上通吃，所以，通常情况下，直接使用bean来指定对象引用就可以了。
```

\<idref>
```bash
如果要为当前对象注入所依赖的对象的名称，而不是引用，那么通常情况下，可以使<value>来达到这个目的。
使用idref，容器在解析配置时就可以检查这个beanName是否存在，而不用等到运行时才发现这个beanName对应的对象实例不存在。
```

内部\<bean>
```bash
使用 <ref>可以引用容器中独立定义的对象定义。但有，可能我们所依赖的对象只有当前一个对象引用，或者某个对象定义我们不想其他对象通过<ref>引用
到它。这时，可使用内嵌的<bean>，将这个私有的对象定义仅局限在当前对象。内部<bean>的配置只是在位置上有所差异，但配置项上与其他的<bean>是
没有任何差别的。也就是，<bean>内嵌的所有元素，内部<bean>的<bean>同样可以使用。如果内部<bean>对应的对象还依赖于其他对象，你完全可以像其他
独立的<bean>定义一样为其配置相关依赖，没有任何差别。
```

\<list>
```bash
<list>对应注入对象类型为java.util.List 及其子类或者数组类型的依赖对象。通过<list>可以有序地为当前对象注入以collection形式声明的依赖。
```

\<set>
```bash
如果说<list>可以帮你有序地注入一系列依赖的话，那么<set> 就是无序的，而且，对于set来说，元素的顺序本来就是无关紧要的。
<set>对应注入Java Collection中类型为java.util.Set 或者其子类的依赖对象。
```

\<map>
```bash
映射（map）可以通过指定的键（key）来获取相应的值。<map>与<list>和<set>的相同点在于，都是为主体对象注入Collection类型的依赖，
不同点在于它对应注入java.util.Map 或者其子类类型的依赖对象。

对于<map>来说，它可以内嵌任意多个<entry>，每一个<entry>都需要为其指定一个键和一个值，就跟真正的java.util.Map所要求的一样。

指定entry的键。可以使用<entry>的属性key或者key-ref来指定键，也可以使用<entry>的内嵌元素<key>来指定键,但两种方式可以达到相同的
效果。在<key>内部，可以使用以上提到的任何元素来指定键，从简单的<value>到复杂的Collection。

指定entry对应的值。<entry>内部可以使用的元素，除了 <key>是用来指定键的，其他元素可以任意使用，来指定entry对应的值。
除了之前提到的那些元素，还包括马上就要谈到的<props>。如果对应的值只是简单的原始类型或者单一的对象引用，也可以直接使用
<entry>的value或value-ref这两个属性来指定，从而省却多敲入几个字符的工作量。
```
\<props>
```bash
<props>是简化后了的<map>，或者说是特殊化的map，该元素对应配置类型为java.util.Properties的对象依赖。因为Properties 
只能指定String类型的键（key）和值，所以，<props>的配置简化很多，只有固定的格式。每个<props>可以嵌套多个<prop>，
每个<prop>通过其key属性来指定键，在<prop>内部直接指定其所对应的值。<prop>内部没有任何元素可以使用，只能指定字符串，
这个是由java.util.Properties 的语意决定的。
```

\<null/>
```bash
<null/>是最简单的一个元素，因为它只是一个空元素，而且通常使用到它的场景也不是很多。对于String类型来说，如果通过value
以这样的方式指定注入，即<value></value>得到的结果是"" ，而不是null。所以，如果需要为这个string对应的值注入null的话，请使用<null/>。
```

###### depends-on

系统中所有需要日志记录的类，都需要在这些类使用之前首先初始化log4j。那么，就会非显式地依赖于SystemConfigurationSetup的静态初始化块。如果ClassA需要使用log4j，那么就必须在bean定义中使用depends-on来要求容器在初始化自身实例之前首先实例化SystemConfigurationSetup，以保证日志系统的可用，如下代码演示的正是这种情况：

```bash
<bean id="classAInstance" class="...ClassA"depends-on="configSetup"/>
<bean id="configSetup" class="SystemConfigurationSetup"/>
```
可以在ClassA的depends-on中通过逗号分割各个beanName。

###### autowire

通过<bean>的autowire属性，可以指定当前bean定义采用某种类型的自动绑定模式。这样，无需手工明确指定该bean定义相关的依赖关系，从而也可以免去一些手工输入的工作量。
  
 Spring提供了5种自动绑定模式，即no、byName、byType、constructor和autodetect，下面是它们的具体介绍：
     
1.no
```bash
容器默认的自动绑定模式，也就是不采用任何形式的自动绑定，完全依赖手工明确配置各个bean之间的依赖关系。
```

2.byName
```bash
按照类中声明的实例变量的名称，与XML配置文件中声明的bean定义的beanName的值进行匹配，相匹配的bean定义将被自动绑定到当前实例变量上。
这种方式对类定义和配置的bean定义有一定的限制。

<bean id="fooBean" class="...Foo" autowire="byName">
</bean>
<bean id="emphasisAttribute" class="...Bar">
</bean>

需要注意两点：
第一，我们并没有明确指定fooBean的依赖关系，而仅指定了它的autowire属性为byName；
第二，第二个bean定义的id为emphasisAttribute ，与Foo类中的实例变量名称相同。
```

3.byType
```bash
如果指定当前bean定义的autowire模式为byType，那么，容器会根据当前bean定义类型，分析其相应的依赖对象类型，然后到容器所管理的所有
bean定义中寻找与依赖对象类型相同的bean定义，然后将找到的符合条件的bean自动绑定到当前bean定义。如果没有找到，则不做设置。但如果找到
多个，容器则无法选择。如果容器中存在多个相同类型的bean定义，采用手动明确配置。

指定 byType 类型的autowire模式与byName没什么差别，只是autowire的值换成 byType 而已。
```

4.constructor
```bash
constructor类型则针对构造方法参数的类型而进行的自动绑定，它同样是byType类型的绑定模式。不过，constructor是匹配构造方法的参数类型，
而不是实例属性的类型。与byType模式类似，如果找到不止一个符合条件的bean定义，那么，容器会返回错误。使用上也与 byType 没有太大差别，
只不过是应用到需要使用构造方法注入的bean定义之上。

<bean id="foo" class="...Foo" autowire="constructor"/>
<bean id="bar" class="...Bar">
</bean>
```

5.autodetect
```bash
这种模式是byType和constructor模式的结合体，如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，
会使用constructor模式。当然，如果通过构造方法注入绑定后还有其他属性没有绑定，容器也会使用byType对剩余的对象属性进行自动绑定。
```

###### dependency-check

使用每个\<bean>的dependency-check 属性对其所依赖的对象进行最终检查。该功能主要与自动绑定结合使用，可以保证当自动绑定完成后，最终确认每个对象所依赖的对象是否按照所预期的那样被注入。该功能可以帮我们检查每个对象某种类型的所有依赖是否全部已经注入完成，不过可能无法细化到具体的类型检查。但某些时候，使用setter方法注入就是为了拥有某种可以设置也可以不设置的灵活性所以，这种依赖检查并非十分有用，尤其是在手动明确绑定依赖关系的情况下。
      
基本上有如下4种类型的依赖检查:
none 
```bash
不做依赖检查。将dependency-check指定为none跟不指定这个属性等效，默认情况下，容器以此为默认值。
```

simple 
```bash
如果将dependency-check的值指定为simple ，那么容器会对简单属性类型以及相关的collection进行依赖检查，对象引用类型的依赖除外。
```

object 
```bash
只对对象引用类型依赖进行检查。
```

all 
```bash
将simple和object相结合，也就是说会对简单属性类型以及相应的collection和所有对象引用类型的依赖进行检查。
```
###### lazy-init

延迟初始化（lazy-init）这个特性的作用，主要是可以针对ApplicationContext容器的bean初始化行为施以更多控制。我们不想某些bean定义在容器启动后就直接实例化，想改变某个或者某些bean定义在ApplicationContext容器中的默认实例化时机。这时，就可以通过\<bean>的lazy-init属性来控制这种初始化行为。

```bash
<bean id="lazy-init-bean" class="..." lazy-init="true"/>
<bean id="not-lazy-init-bean" class="..."/>
```

##### 继承

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
我们在声明subNewsProvider的时候，使用了parent属性，将其值指定为superNewsProvider，这样就继承了superNewsProvider定义的默认值，只需要将特定的属性进行更改，而不要全部又重新定义一遍。parent属性还可以与abstract属性结合使用，达到将相应bean定义模板化的目的。

abstract：

```bash
newsProviderTemplate的bean定义通过abstract属性声明为true ，说明这个bean定义不需要实例化。实际上，这就是之前提到的可以不
指定class属性的少数场景之一。容器在初始化对象实例的时候，不会关注将abstract属性声明为true的bean定义。如果你不想容器在初始化
的时候实例化某些对象，那么可以将其abstract 属性赋值true ，以避免容器将其实例化。
```

##### scope

scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的scope之前，生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁Spring的IoC容器这些对象。

Spring容器最初提供了两种bean的scope类型：singleton和prototype

###### singleton
```bash
标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，
并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
```
<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114724944-a4b1a100-9d6e-11eb-8a0b-cce713e01b23.png"/></div> 


###### prototype
```bash
针对声明为拥有prototype scope的bean定义，容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。只要准
备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理
工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了
```
<div align="center"> <img src="https://user-images.githubusercontent.com/37955886/114725295-f6f2c200-9d6e-11eb-9732-10dc76b0da07.png"/></div>

###### 自定义scope

首先需要给出一个Scope接口的实现类，接口定义中的4个方法并非都是必须的，但get和remove方法必须实现。

```bash
public interface Scope {
      Object get(String name, ObjectFactory objectFactory);
      Object remove(String name);
      void registerDestructionCallback(String name, Runnable callback);
      String getConversationId();
}
```

##### 工厂方法与 FactoryBean

通过使用工厂方法（Factory Method）模式，提供一个工厂类来实例化具体的接口实现类，这样，主体对象只需要依赖工厂类，具体使用的实现类有变更的话，只是变更工厂类，而主体对象不需要做任何变动。

（[工厂方法](https://www.runoob.com/design-pattern/factory-pattern.html)）

###### 静态工厂方法（Static Factory Method）
```bash
public class StaticBarInterfaceFactory
{
  public static BarInterface getInstance()
  {
    return new BarInterfaceImpl();
  }
}

<bean id="foo" class="...Foo">
      <property name="barInterface">
            <ref bean="bar"/>
      </property>
</bean>
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```
class指定静态方法工厂类，factory-method指定工厂方法名称，然后，容器调用该静态方法工厂类的指定工厂方法（getInstance），并返回方法调用后的结果，即BarInterfaceImpl的实例。也就是说，为 foo注入的bar实际上是BarInterfaceImpl的实例，即方法调用后的结果，而不是静态工厂方法类（StaticBarInterfaceFactory）。

##### 非静态工厂方法（Instance Factory Method）
```bash
<bean id="foo" class="...Foo">
      <property name="barInterface">
            <ref bean="bar"/>
      </property>
</bean>
<bean id="barFactory" class="...NonStaticBarInterfaceFactory"/>
<bean id="bar" factory-bean="barFactory" factory-method="getInstance"/>
```
NonStaticBarInterfaceFactory是作为正常的bean注册到容器的，而bar的定义则与静态工厂方法的定义有些不同。现在使用factory-bean属性来指定工厂方法所在的工厂类实例，而不是通过class属性来指定工厂方法所在类的类型。指定工厂方法名则相同，都是通过 factory-method 属性进行的。

#####  FactoryBean

FactoryBean 是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口，请不要将其与容器名称BeanFactory相混淆。它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂（Factory）。

以下两种情况可以实现org.spring-framework.beans.factory.FactoryBean接口，给出自己的对象实例化逻辑代码。

1.XML配置过于复杂，使我们宁愿使用Java代码来完成这个实例化过程的时候

2.某些第三方库不能直接注册到Spring容器的时候

```bash
public interface FactoryBean {
      Object getObject() throws Exception;
      Class getObjectType();
      boolean isSingleton();
}

getObject()方法会返回该FactoryBean“生产”的对象实例，我们需要实现该方法以给出自己的对象实例化逻辑； 

getObjectType()方法仅返回getObject()方法所返回的对象的类型，如果预先无法确定，则返回null；

isSingleton()方法返回结果用于表明，工厂方法(getObject())所“生产”的对象是否要以singleton形式存在于容器中。
如果以singleton形式存在，则返回true ，否则返回false；
```

##### 方法注入（Method-Injection）以及方法替换（Method-Replacement）

bean的scope的使用“陷阱”，特别是prototype在容器中的使用。当使用prototype时，每个对象都应该是新的独立个体，但是当容器注入一个实例的时候，类有可能会一直持有这个实例的引用，虽然每次都返回了实例，但是都是第一次所注入的实例，没有重新注入实例。

无状态Bean的作用域一般可以配置为singleton（单例模式），如果我们往singleton的Boss中注入prototype的Car，并希望每次调用bossBean的getCar()方法时都能够返回一个新的carBean，使用传统的注入方式将无法实现这样的要求。因为singleton的Bean注入关联Bean的动作仅有一次，虽然carBean的作用范围是prototype类型，但Boss通过getCar()方法返回的对象还是最开始注入的那个carBean。

（[方法注入](https://www.cnblogs.com/jwen1994/p/11048658.html)）

为此Spring使用了两个方法：方法注入（Method-Injection）以及方法替换（Method-Replacement）

###### 方法注入

让getNewsBean方法声明符合规定的格式，并在配置文件中通知容器，当该方法被调用的时候，每次返回指定类型的对象实例即可。方法声明需要符合的规格定义如下：

```bash
<public|protected> [abstract] <return-type> theMethodName(no-arguments);

//例子
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false">
</bean>
<bean id="mockPersister" class="..impl.MockNewsPersister">
  <lookup-method name="getNewsBean" bean="newsBean"/>
</bean>
```
该方法必须能够被子类实现或者覆写，因为容器会为我们要进行方法注入的对象使用Cglib动态生成一个子类实现，从而替代当前对象。通过 <lookup-method>的name属性指定需要注入的方法名，bean属性指定需要注入的对象，当getNewsBean方法被调用的时候，容器可以每次返回一个新的FXNewsBean类型的实例。
  
除了使用方法注入来达到“每次调用都让容器返回新的对象实例”的目的，还可以使用其他方式达到相同的目的。下面给出其他两种解决类似问题的方法：

（1）使用BeanFactoryAware接口

Spring框架提供了一个BeanFactoryAware接口，容器在实例化实现了该接口的bean定义的过程中，会自动将容器本身注入该bean。这样，该bean就持有了它所处的BeanFactory的引用。BeanFactoryAware的定义如下代码所示：

```bash
public interface BeanFactoryAware {
void setBeanFactory(BeanFactory beanFactory) throws BeansException;
}
```
（2）使用ObjectFactoryCreatingFactoryBean

ObjectFactoryCreatingFactoryBean是Spring提供的一个FactoryBean实现，返回一个ObjectFactory实例。从ObjectFactoryCreatingFactoryBean返回的这个ObjectFactory实例可以为我们返回容器管理的相关对象。实际上，ObjectFactoryCreatingFactoryBean实现了BeanFactoryAware接口，它返回的ObjectFactory实例只是特定于与Spring容器进行交互的一个实现而已。使用它的好处就是，隔离了客户端对象对BeanFactory的直接引用。


###### 方法替换

方法替换更多体现在方法的实现层面上，它可以灵活替换或者说以新的方法实现覆盖掉原来某个方法的实现逻辑。

（1）给出org.springframework.beans.factory.support.MethodReplacer的实现类，在这个类中实现将要替换的方法逻辑。
（2）有了要替换的逻辑之后，我们就可以把这个逻辑通过 <replaced-method> 配置到FXNewsProvider的bean定义中，使其生效。
 
 ```bash
<replaced-method name="getAndPersistNews" replacer="providerReplacer">
</replaced-method>
```

## 注解方式

通过注解标注的方式为 FXNewsProvider 注入所需要的依赖，现在可以使用 @Autowired 以及 @Component 对相关类进行标记。再向Spring的配置文件中增加一个“触发器”，使用 @Autowired 和 @Component 标注的类就能获得依赖对象的注入了。

### @Autowired

它的存在将告知Spring容器需要为当前对象注入哪些依赖对象。

### @Component

配合Spring 2.5中新的classpath-scanning功能使用的。



