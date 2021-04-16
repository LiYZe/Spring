# AOP

AOP全称为Aspect-Oriented Programming，中文常翻译为面向方面编程。使用AOP可以对系统需求进行模块化的组织，简化系统需求与实现之间的对比关系，从而使整个系统得实现更具模块化。

# 问题

AOP没有主权，需要“寄生”于OOP的主权领土中，系统的维度也依然保持曾经OOP持有的“维度世界纪录”。

# 实现

## AOL

AOP是一种理念，所以需要语言实现。统称这些实现AOP的语言为AOL(Aspect-Oriented Language)。AOL可以与系统实现语言相同，AspectJ是扩展自Java的一种AOL。

## 历史

### 静态AOP时代

第一代AOP。特点是，相应的横切关注点以Aspect形式实现之后，会通过特定的编译器，将实现后的Aspect编译并织入到系统的静态类中。

优点：Aspect直接以Java字节码的形式编译到Java类中，Java虚拟机可以像通常一样加载Java，不会对整个系统运行造成任何性能损失。

缺点：灵活性不够。

### 动态AOP时代

第二代AOP。该时代的AOP框架或者产品，大都通过Java语言提供的各种动态特性来实现Apsect织入到当前系统中。在AspectJ融合了AspectWerkz框架后，也引入了动态织入的行为，所以称为唯一一个同时支持静态和动态AOP的实现产品。

优点：AOP的织入过程在系统运行开始之后进行，而不是预先编译到系统类中，而且织入信息大都采用外部XML文件格式保存，可以在调整织入点以及织入逻辑单元的同时，不必变更系统的其他模块，在系统运行时也可以动态更改织入逻辑。

缺点：或造成一定的运行时性能损失，但是是可容忍的。

## 实现方式

### 动态代理

可以在运行期间，为相应的接口动态生成对应的代理对象。可以将横切关注点逻辑封装到动态代理的InvocationHandler中，然后在系统运行期间，根据横切关注点需要织入的模块位置，将横切逻辑织入到相应的代理类中。

唯一缺点或者说是优点是，需要织入横切关注点逻辑的模块类都得实现相应的接口，因为动态代理机制只针对。

### 动态字节码增强

为需要织入横切逻辑的模块类在运行期间，通过动态字节码增强技术，为这些系统模块类生成相应的子类，而将横切逻辑加到这些子类中，让应用程序在执行期间使用的是这些动态生成的子类，从而达到将横切逻辑织入到系统的目的。

### Java代码生成

EJB容器根据部署描述文件提供的织入信息，会为相应的功能模块类根据描述符所提供的信息生成对应的Java代码，然后通过部署工具或者部署接口编译Java代码生成相应的Java类，之后部署到EJB容器的功能模块类可以正常工作。

### 自定义类加载器

自定义类加载器通过读取外部文件规定的织入规则和必要信息，在加载class文件期间就可以将横切逻辑添加到系统模块类的现有逻辑中，然后将改动后的class交给java虚拟机。

### AOL扩展

可以使用扩展过的AOL实现任何AOP概念实体甚至是OOP概念实体。采用扩展的AO具有强类型检查，基本可以对横切关注点要切入系统运行时点有更全面的控制。

## AOP相关概念

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115004414-5c62c200-9ed9-11eb-8ad4-3c078f9e34f1.png"/></div> 

### Joinpoint

AOP功能模块都需要织入到OOP的功能模块中，所以要进行织入过程，需要知道在系统的哪些执行点上进行织入操作，这些将要在其之上进行织入操作的系统执行点就称之为Joinpoint。基本上，程序执行过程中的任何时点都可以作为横切逻辑的织入点，而所有这些执行时点都是Joinpoint

#### 方法调用

当某个方法被调用的时候所处的程序执行点。方法调用要先于方法执行。

#### 方法调用执行

某个方法内部执行开始时点。
<div align="center"><img src="https://user-images.githubusercontent.com/37955886/114996808-c7a89600-9ed1-11eb-97df-9f651c9230dc.png"/></div> 

#### 构造方法调用

程序执行过程中对某个对象调用其构造方法进行初始化的时点。

#### 构造方法执行

某个对象构造方法内部执行的开始时点。

### 字段设置

对象的某个属性通过setter方法被设置或者直接被设置的时点。本质是对象的属性被设置。

#### 字段获取

某个对象相应属性被访问的时点。

#### 异常处理执行

在某些类型异常抛出后，对应的异常处理逻辑执行的时点。

#### 类初始化

指的是类中某些静态类型或静态块的初始化时点。

### Pointcut

Pointcut概念代表的是Joinpoint的表述方式。将横切逻辑织入当前系统的过程中，需要参照Pointcut规定的Joinpoint信息，才知道应该往系统的哪些Joinpoint上织入横切逻辑。

#### 表述方式

##### 直接指定Joinpoint所在方法名称

表述方式简单，功能单一，通常只限于支持方法级别Joinpoint的AOP框架，或者只是方法调用类型的JointPoint，或者只是方法执行类型的Joinpoint。

##### 正则表达式

比较普遍的Pointcut表达方式，可以充分利用正则表达式的强大功能，来归纳表述需要符合某种条件的多组Joinpoint。

##### 使用特定的Pointcut表述语言

AspectJ使用这种方式来指定Pointcut，它提供了一种类似于正则表达式的针对Pointcut的表述语言，在表达ointcut方面支持比较完善。

#### Pointcut运算

Pointcut与Pointcut之间可以进行逻辑运算。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/115000537-6edafc80-9ed5-11eb-8352-eb59863c17a7.png"/></div> 

### Advice

Advice是单一横切关注点逻辑的载体，它代表将会织入到Joinpoint的横切逻辑。

#### Before Advice

在Joinpint指定位置之前执行的Advice类型，通常不会中断程序执行流程，可抛出异常中断当前流程。如果当前Before Advice将被织入到方法执行类型的Joinpoint，那么就会先于方法执行而执行。通常做一些系统的初始化工作。

#### After Advice

在相应连接点之后执行的Advice类型。

可以细分为以下三种：

##### After returning Advice

只有当前Joinpoint处执行流程正常完成后，After returning Advice才会执行。

##### After throwing Advice

又称Throws Advice，只有在当前Joinpoint执行过程中抛出异常的情况下，才会执行。

##### After Advice

该类型不管Joinpoint处执行流程是正常终了还是抛出异常都会执行。

#### Around Advice

对附加其上的Joinpoint进行“包裹”，可以在Joinpoint之前和之后都指定相应的逻辑，甚至于中断或者忽略Joinpoint处原来程序流程的执行。Around Advice既然可以在Joinpoint之前和之后都能执行相应的逻辑，那么就可以完成Before Advice和After Advice功能。

#### Introduction

Introduction不是根据横切逻辑在Joinpoint出的执行时机来区分的，而是根据它可以完成的功能而区别于其他Advice类型：为原有的对象添加新的特性或者行为。

### Aspect

是对系统中的横切关注点逻辑进行模块化封装的AOP概念实体。通常情况下考研包含多个Pointcut以及相关的Advice定义。

### 织入和织入器

织入过程就是“飞架”AOP和OOP的那座桥，只有经过织入过程之后，以Aspect模块化的横切关注点才会集成到OO的现存系统中，而完成织入过程的那个“人”就称之为织入器。

Spring AOP使用一组类来完成最终的织入操作，ProxyFactory类则是Spring AOP中最通用的织入器。

### 目标对象

符合Pointcut所指定的条件，将在织入过程中被织入横切逻辑的对象，称为目标对象。










