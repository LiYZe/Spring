# Spring AOP

Srping AOP 是Spring核心框架的重要组成部分，通常认为它与Spring的IoC容器以及Spring框架对其他JavaEE服务的集成共同组成了Spring框架的：质量三角。

采用Java作为AOP的实现语言（AOL），Spring AOP 可以快捷地融入开发过程，学习曲线相对平滑。

# 实现机制

Spring AOP属于第二代AOP，采用动态代理机制和字节码生成技术实现，这两个技术都是在运行期间为目标对象生成一个代理对象，而将横切逻辑织入到这个代理对象中，系统最终使用的是织入了横切逻辑的代理对象，而不是真正的目标对象。

## 代理模式

代理模式能够减少被访问者的负担，即使代理最终要将访问请求转发给真正的访问者，它也可以在转发访问请求之前或者之后加入特定的逻辑。

在代理模式中通常涉及四种角色。

<div  align="center"> <img src="https://user-images.githubusercontent.com/37955886/115006355-5a99fe00-9edb-11eb-9354-182c7f868c42.png"/></div> 

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






















