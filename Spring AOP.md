# Spring AOP

Srping AOP 是Spring核心框架的重要组成部分，通常认为它与Spring的IoC容器以及Spring框架对其他JavaEE服务的集成共同组成了Spring框架的：质量三角。

采用Java作为AOP的实现语言（AOL），Spring AOP 可以快捷地融入开发过程，学习曲线相对平滑。

# 实现机制

Spring AOP属于第二代AOP，采用动态代理机制和字节码生成技术实现，这两个技术都是在运行期间为目标对象生成一个代理对象，而将横切逻辑织入到这个代理对象中，系统最终使用的是织入了横切逻辑的代理对象，而不是真正的目标对象。

## 代理模式

代理模式能够减少被访问者的负担，即使代理最终要将访问请求转发给真正的访问者，它也可以在转发访问请求之前或者之后加入特定的逻辑。

在代理模式中通常涉及四种角色。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115006355-5a99fe00-9edb-11eb-9354-182c7f868c42.png"/></div> 

### ISubject

该接口是对被访问者或者被访问资源的抽象。

### SubjectImpl

被访问者或者被访问资源的具体实现类。

实现了接口ISubject。

### SubjectProxy

被访问者或者被访问资源的代理实现类，该类持有一个ISubject接口的具体实例。我们要对SubjectImpl进行代理，那么SubjectProxy现持有的就是SubjectImpl。

实现了接口ISubject。并且内部持有SubjectImpl的引用

### Client

代表访问者的抽象角色，将会访问ISubject类型的对象或者资源。Client将会请求具体的SubjectImpl实例，但是Client无法直接请求其真正要访问的资源SubjectImpl，二通过ISubject资源的访问代理类SubjectProxy进行。

## 动态代理

使用这个机制，我们可以为指定的接口在系统运行期间动态的生成代理对象。

动态代理机制的实现主要由一个类和一个接口组成，及java.lang.reflect.Proxy类和java.lang.reflect.InvocationHandler接口。

InvocationHandler就是实现横切逻辑的地方，是很且逻辑的载体，作用与Advice是一样的。

### 缺点

动态代理机制只能对实现了相应Interface的类的使用，如果某个类没有实现任何的Interface，就无法使用动态代理机制为其生成相应的动态代理对象。

默认成情况下，如果Spring AOP 发现目标对象实现了相应的Interface，则采用动态代理机制为其生成代理对象实例。如果目标对象没有实现任何Interface，Spring AOP 会尝试使用一个称为CGLIB的开源动态字节码生成类库，为目标对象生成动态的代理对象实例。

## 动态字节码生成

使用动态字节码生成技术扩展对象行为的原理是，对目标对象进行继承扩展，为其生成相应的子类，而子类可以通过覆写来扩展父类的行为，只要将横切逻辑的实现放在子类中，然后让系统使用扩展后端目标对象的子类，就可以达到与代理模式相同的效果。

但是使用继承的方式扩展对象定义，不能像静态代理模式那样，为每个不同类型的目标对象都是单独创建相应的扩展子类，所以需要借助CGLIB的动态字节码生成库，在系统运行期间动态的为目标对象生成相应的扩展子类。

CGLIB可以对实现了某种接口的类，或者没有实现任何接口的类进行扩展。使用CGLIB对类进行扩展的唯一限制就是无法对final方法进行覆写。

# Joinpoint

仅支持方法级别的Joinpoin，确切地说，只支持方法执行类型的Joinpoint。

对于类中属性级别的Joinpoint，如果提供这个级别的拦截支持，那么久破坏了面向对象的封装，而且可以通过对setter和getter方法的拦截达到同样的目的。

如果超出了需求支持，可以使用AspectJ。

# Pointcut

Spring中以接口定义org.springframework.aop.Pointcut作为其AOP框架中所有Pointcut的最顶层抽象，该接口定义了两个方法帮助捕捉系统中的相应Joinpoint，并提供了一个TruePointcut类型实例。如果Pointcut类型为TruePointcut，默认会对系统中的所有对象，以及对象上所有被支持的Joinpoint进行匹配。

org.springframework.aop.Pointcut接口定义如下：

```bash
public interface Pointcut{
  ClassFilter getClassFilter();
  MethodMatcher getMethodMatcher();
  Pointcut TRUE = TruePointcut.INSTANCE;
}
```

ClassFilter和MethodMatcher分别用于匹配将被执行织入操作的的对象以及相应的方法。将类型匹配和方法匹配分开定义的原因是可以重用不同级别的匹配定义，并且可以在不同的级别或者相同的级别上进行组合操作，或者强制让某个子类只覆写相应的方法定义等。

## ClassFilter

ClassFilter接口的作用是对Joinpoint所处的对象进行Class级别的类型匹配，定义如下：

```bash
public interface ClassFilter{
  boolean matches(Class clazz);
  ClassFilter TRUE = TrueClassFilter.INSTANCE; 
}
```

当织入的目标对象的Class类型与Pointcut所规定的类型相符时，matches方法将会返回true，否则，返回false。如果类型对我们所捕捉的Joinpoint无所谓，那么Pointcut中使用的ClassFilter可以直接使用“ClassFilter TRUE = TrueClassFilter.INSTANCE; ”。当Pointcut中返回的ClassFilter类型为该类型的实例时，Pointcut的匹配将会针对系统中所有的目标类以及它们的实例进行。

## MethodMatcher

定义如下：
```bash
public interface MethodMatcher{
  boolean matches(Method method, Class targetClass);
  boolean isRuntime();
  boolean matches(Method method, Class targetClass, Object[] args);
  MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
}
```

MethodMatcher通过重载，定义了两个matches方法，而这两个方法的分界线就是isRuntime()方法。在对对象具体方法进行拦截的时候，可以忽略每次方法执行的时候调用者传入的参数，也可以每次都检查这些方法调用参数，以强化拦截条件。

在MethodMatcher基础上，Pointcut可以分为两类，StaticMethodMatcher和DynamicMethodMatcher：

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115036842-71068080-9f00-11eb-8380-3e137795ba43.png"/></div> 


### StaticMethodMatcher

isRuntime返回false，表示不会考虑具体Jointpoint的方法参数，这种MethodMatcher称之为StaticMethodMatcher。

因为不用每次每次检查参数，那么对于同样类型的方法匹配结果，就可以在框架内部缓存以提高性能。

当为StaticMethodMatcher的时候，只有boolean matches(Method method, Class targetClass);方法执行，其结果将会成为其所属的Pointcut主要依赖。

### DynamicMethodMatcher

isRuntime返回true时，表明该MethodMatcher将会每次都对方法调用的参数进行匹配检查，这种类型的MethodMatcher称之为DynamicMethodMatcher。

因为要每次检查参数，匹配效率较差。

如果一个MethodMatcher为DynamicMethodMatcher（isRuntime()返回true），并且当方法boolean matches(Method method, Class targetClass, Object[] args);也返回true的时候，三个参数的matches方法将被执行，以进行进一步检查匹配。如果  boolean matches(Method method, Class targetClass, Object[] args);返回false，则为最终结果。

## 常见Pointcut

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115037029-a01cf200-9f00-11eb-81ca-cc1929521dd7.png"/></div> 

### NameMatchMethodPointcut

最简单的Pointcut实现，属于StaticMethodMatcherPointcut的子类，可以根据自身指定的一组方法名称与Joinpoint处的方法的方法名称进行匹配。但是无法对重载的方法名进行匹配，因为它仅对方法名进行匹配，不考虑参数相关信息，也没有提供可以指定参数匹配信息的途径。

除了可以指定方法名进行匹配，还可以使用“ * ”通配符，实现模糊查询。

### JdkRegexpMethodPointcut和Per15RegexpMethodPointcut

StaticMethodMatcher子类中有一个专门提供基于正则表达式的实现分支，以抽象类AbstractRegexpMethodPointcut为统帅。这个抽象类中声明了pattern和patterns属性，可以指定一个或者多个正则表达式的匹配模式。其下设JdkRegexpMethodPointcut和Per15RegexpMethodPointcut两种具体实现。

使用正则表达式来匹配相应的Joinpint所处的方法时，正则表达式的匹配模式必须以匹配整个方法签名的形式指定，而不能仅指出匹配的方法名称。

### AnnotationMatchingPointcut

根据目标对象中是否存在指定类型的注解来匹配Joinpoint，要使用该类型的Pointcut，首相需要声明相应的注解。

### ComposablePointcut

ComposablePointcut就是Spring AOP提供的可以进行Pointcut逻辑运算的Pointcut实现，可以进行Pointcut之间的“并”，“交”运算。
 
### ControlFlowPointcut

ControlFlowPointcut匹配程序的调用流程，不是对某个方法执行所在的Joinpoint处的单一特征进行匹配。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115040233-cb551080-9f03-11eb-914a-092368ec5d0b.png"/></div> 

如果使用之前的任何Pointcut实现，一旦通过Pointcut指定method1处为Joinpoint，那么对该方法的执行进行拦截是必定的，不管method1被谁调用。

而通过ControlFlowPointcut，可以指定只有当TargetObject的method1方法在TargetCaller类所声明的方法中被调用的时候，才对method1方法进行拦截，其他方法调用method1的话，不对method1进行拦截。

## 扩展Pointcut

Spring AOP已经提供了相应的扩展抽象类支持，只需继承相应的抽象父类，然后实现或者覆写相应方法即可，

### 自定义StaticMethodMatcherPointcut

1.所有的StaticMethodMatcherPointcut的子类的ClassFilter均为ClassFilter.TRUE,即忽略类的类型匹配。如果具体子类需要对目标对象的类型进一步限制，可以通过public void setClassFilter(ClassFilter classFilter)方法设置相应的ClassFilter实现。

2.因为是StaticMethodMatcherPointcut，所以需要实现两个参数的matches方法。

### 自定义DynamicMethodMatcherPointcut

1.getClassFilter()方法返回ClassFilter.TRUE，如果需要对特定的目标对象类型进行限定，子类只需覆写这个方法。

2.是DynamicMethodMatcherPointcut，所以需要实现三个参数的matches方法。

## IoC容器中的Pointcut

Spring中的Pointcut实现都是普通的Java对象，所以可以通过Spring的IoC容器来注册并使用。

# Advice

Spring中的Advice实现全部都遵循AOP Alliance规定的接口。Advice实现了将被织入到Pointcut规定的Joinpoint处的横切逻辑，可按照其自身实例能否在目标对象类的所有实例中共享这一标准。可以划分两类，per-class类型和per-instance类型。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115044873-854e7b80-9f08-11eb-85c1-669f477073e3.png"/></div> 

## per-class

该类型的Advice的实例可以在目标对象类的所有实例之间共享。通常只提供方法拦截的功能，不会为目标对象保存任何状态或者添加新的特性。

### Before Advice

Before Advice实现的横切逻辑将在相应的Joinpoint之前执行，在Before Advice执行完成之后，程序执行流程将从Joinpoint处继续执行，所以Before Advice通常不会打断程序的执行流程。如果必要，可以跑出相应异常的形式中断流程。

可以使用Before Advice进行整个系统的某些资源初始化或者其他一些准备性的工作。

要实现Before Advice，通常只要实现org.springframework.aop.MethodBeforeAdvice接口，定义如下：

```bash
public interface MethodBeforeAdvice extends BeforeAdvice{
  void before(Method method, Object[] args, Object target) throws Throwable;
}
```
MethodBeforeAdvice继承了BeforeAdvice，而BeforeAdvice只是标志接口，没有定义任何方法。

### ThrowsAdvice

ThrowsAdvice通常用于对系统中特定的异常情况进行监控，以统一的方式对所发生的异常进行处理，可以在Fault Barrier的模式中使用它。

Spring中以接口定义org.springframework.aop.ThrowsAdvice对应通常AOP概念中的AfterthrowingAdvice。

实现ThrowsAdvice需遵循如下规则：

void afterThrowing ([Method, args, target], ThrowableSubclass);

### AfterReturningAdvice

通过Spring中的AfterReturningAdvice，可以访问当前Joinpoint的方法返回值、方法、方法参数以及所在的目标对象。该横切逻辑会在方法成功执行后进行，并且不能对方法返回值进行更改。

org.springframework.aop.AfterReturningAdvice接口定义了Spring的AfterR额turningAdvice，其定义如下：

```bash
public interface AfterRerturningAdvice extends AfterAdvice{
  void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable;
}
```

### Around Advice

Spring没有直接定义对应Around Advice的实现接口，而是直接采用AOP Alliance的标准接口，即org.aopalliance.intercept.MethodInterceptor，定义如下：

```bash
public interface MethodInterceptor extends Interceptro{
  Object invoke(MethodInvocation invocation) throws Throwable;
}
```

通过MethodInterceptor的invoke方法的MethodInvocation参数可以控制对相应Joinpoint的拦截行为。通过调用MethodInvocation的proceed()方法，可以让程序执行继续沿着调用链传播。如果MethodInterceptor没有电泳proceed()，程序会在MethodInterceptor处“短路”，Jooinpoint上的调用链接被中团，同一Joinpoint上的其他MethodInterceptor的逻辑以及Joinpoint处的方法逻辑不会执行。

可以在proce()方法，也就是Joinpoint处的逻辑执行之前或者之后插入相应的逻辑，甚至捕获proceed()方法可能抛出的异常。

## per-instance

per-instance类型的Advice不会再目标类所有对象实例之间共享，而是会为不同的实例对象保存各自的状态以及相关逻辑，

在Spring AOP中，Introduction是唯一一种per-instance型Advice

### Introduction

Introduction可以在不改动目标类定义的情况下，为目标类添加新的属性以及行为。

在Spring中，为目标对象添加新的属性以及行为必须声明相应的接口以及相应的实现。在通过特定的拦截器将新的接口定义以及实现类中的逻辑附加到目标对象之上。之后目标对象拥有了新的状态和行为。

这个拦截器就是org.springframework.aop.IntroductionInterceptor，定义如下：
```bash
public interface IntroductionInterceptor extends MethodInterceptor, DynamicIntroductionAdvice{
}

public interface DynamicIntroductionAdvice extends Advice(){
  boolean implementsInterface(Class intf);
}
```
IntroductionInterceptor继承了MethodInterceptor和DynamicIntroductionAdvice。通过DynamicIntroductionAdvice可以界定当前的IntroductionInterceptor为哪些接口提供相应的拦截功能。通过MethodInterceptor，可以处理新添加的接口上的方法调用。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115050614-808cc600-9f0e-11eb-8a57-ac57b42e2e00.png"/></div> 

#### 实现

实现Introduction型Advice的两条分支，以DynamicintroductionAdvice为首的动态分支和IntroductionInfo为首的静态可配置分支

##### DynamicintroductionAdvice

可以到运行时在判定当前Introduct可应用到的目标接口类型，不用预先设定。

##### IntroductionInfo

实现类必须返回预定的目标接口类型。

定义如下：
```bash
public interface IntroductionInfo{
  Class[] getInterfaces();
}
```
#### Delegatinglntroductionlnterceptor

Delegatinglntroductionlnterceptor不会自己实现将要添加到目标对象上的新的逻辑行为，而是委派给其他实现类。

使用Delegatinglntroductionlnterceptor添加新的状态或者行为，步骤如下：

1.为新的状态和行为定义接口；

2.给出新接口的实现类；

3.通过Delegatinglntroductionlnterceptor进行Introduction的拦截，有了新增职能的接口定义以及相应实现类，；

实际上，Delegatinglntroductionlnterceptor会使用他所持有的同一个接口实例，供同一目标类的所有实例共享使用。所以，没有实现per-instance型的Adcvice。

#### DelegatePerTargetObjectIntroductionInterceptor

DelegatePerTargetObjectIntroductionInterceptor会在内部持有一个目标对象与相应Introduction逻辑实现类之间的映射关系。当每个目标对象上的新定义的接口方法被调用的时候，DelegatePerTargetObjectIntroductionInterceptor会拦截这些调用，然后以目标对象实例作为键，到它持有的那个映射关系中取得对应当前目标对象实例的Introduction实现类实例。如果根据当前目标对象实例没有找到对应的Introduction实现类实例，DelegatePerTargetObjectIntroductionInterceptor会自己为其创建一个新的，然后添加到映射关系中。

不需自己构造delegate接口实例，只需告知DelegatePerTargetObjectIntroductionInterceptor相应的delegate接口类型和对应实现类的类型

















