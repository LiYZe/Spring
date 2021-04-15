# ApplicationContext

# 统一资源加载策略

URL全名是Uniform Resource Locator。基本实现只限于网络形式发布的资源的查找和定位工作，基本上只提供了基于HTTP、FTP、File等协议的资源定位功能，也提供了扩展的接口。

## Resource

Spring框架内部使用org.springframework.core.io.Resource接口作为所有资源的抽象和访问接口。

Resource接口可以根据资源的不同类型，或者资源所处的不同场合，给出相应的具体实现。Spring框架在这个理念的基础上，提供了一些实现类

### ByteArrayResource 

将字节（byte）数组提供的数据作为一种资源进行封装，如果通过InputStream形式访问该类型的资源，该实现会根据字节数组的数据，构造相应的 ByteArray-
InputStream 并返回。

### ClassPathResource 

该实现从Java应用程序的ClassPath中加载具体资源并进行封装，可以使用指定的类加载器（ClassLoader）或者给定的类进行资源加载。

### FileSystemResource

对 java.io.File 类型的封装，所以，我们可以以文件或者URL的形式对该类型资源进行访问，只要能跟 File 打的交道，基本上跟 FileSystemResource 也可以。

### UrlResource

通过 java.net.URL 进行的具体资源查找定位的实现类，内部委派URL进行具体的资源操作。

### InputStreamResource 

将给定的 InputStream 视为一种资源的 Resource 实现类，较为少用。可能的情况下，以 ByteArrayResource 以及其他形式资源实现代之。

### 自定义

如果以上这些资源实现还不能满足要求，那么我们还可以根据相应场景给出自己的实现，只需实现 org.springframework.core.io.Resource 接口。

```bash
public interface Resource extends InputStreamSource {
  boolean exists();
  
  boolean isOpen();
  
  URL getURL() throws IOException;
  
  File getFile() throws IOException;
  
  Resource createRelative(String relativePath) throws IOException;
  
  String getFilename();
  
  String getDescription();
}
public interface InputStreamSource {
  InputStream getInputStream() throws IOException;
}
```

## ResourceLoader

org.springframework.core.io.ResourceLoader 接口是资源查找定位策略的统一抽象，具体的资源查找定位策略则由相应的 ResourceLoader 实现类给出。

定义如下：

```bash
public interface ResourceLoader {
  String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;
  Resource getResource(String location);
  ClassLoader getClassLoader();
}
```

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/114845033-f65d3880-9e0d-11eb-8b4f-dd9eef6c9ecc.png"/></div> 

### 实现类

#### DefaultResourceLoader

ResourceLoader有一个默认的实现类，即org.springframework.core.io.DefaultResourceLoader。

该类默认的资源查找处理逻辑如下：

(1) 首先检查资源路径是否以classpath: 前缀打头，如果是，则尝试构造 ClassPathResource 类型资源并返回。

(2) 否则，(a)尝试通过URL，根据资源路径来定位资源，如果没有抛出MalformedURLException，有则会构造UrlResource类型的资源并返回；
(b)如果还是无法根据资源路径定位指定的资源，则委派getResourceByPath(String)方法来定位，DefaultResourceLoader的getResourceByPath(String)方法默认实现逻辑是，构造ClassPathResource类型的资源并返回。

#### FileSystemResourceLoader

org.springframework.core.io.FileSystemResourceLoader，它继承自DefaultResourceLoader，但覆写了getResourceByPath(String)方法，使之从文件系统加载资源并以FileSystemResource类型返回。

#### ResourcePatternResolver

ResourcePatternResolver是ResourceLoader的扩展，ResourceLoader每次只能根据资源路径返回确定的单个Resource实例，而ResourcePatternResolver则可以根据指定的资源路径匹配模式，每次返回多个Resource实例。

定义如下：

```bash
public interface ResourcePatternResolver extends ResourceLoader {
  String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
  Resource[] getResources(String locationPattern) throws IOException;
}
```
ResourcePatternResolver 最常用的一个实现是org.springframework.core.io.support.PathMatchingResourcePatternResolver ，该实现类支持ResourceLoader级别的资源加载，支持基于Ant风格的路径匹配模式，支持ResourcePatternResolver新增加的classpath*:前缀等，基本上集所有技能于一身

## ApplicationContext与ResourceLoader

ApplicationContext继承了ResourcePatternResolver，当然就间接实现了ResourceLoader接口。所以，任何的 ApplicationContext 实现都可以看作是一个ResourceLoader甚至ResourcePatternResolver。而这就是ApplicationContext支持Spring内统一资源加载策略的真相。所有的ApplicationContext实现类会直接或者间接地继承org.springframework.
context.support.AbstractApplicationContext。AbstractApplicationContext继承了DefaultResourceLoader，它的getResource(String)当然就直接用DefaultResourceLoader的了。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/114845767-9d41d480-9e0e-11eb-8c58-bfc0326fef80.png"/></div> 

###  ApplicationContext作用

#### 扮演ResourceLoader的角色

既然ApplicationContext可以作为ResourceLoader或者ResourcePatternResolver来使用，那么，我们可以通过ApplicationContext来加载任何Spring支持的Resource类型。

#### ResourceLoader类型的注入

如果某个bean需要依赖于ResourceLoader来查找定位资源，我们可以为其注入容器中声明的某个具体的ResourceLoader实现，该bean也无需实现任何接口，直接通过构造方法注入或者setter方法注入规则声明依赖即可

容器启动的时候，就会自动将当前ApplicationContext容器本身注入到FooBar中，因为ApplicationContext类型容器可以自动识别Aware接口。

#### Resource类型的注入

对于那些Spring容器提供的默认的PropertyEditors无法识别的对象类型，我们可以提供自定义的PropertyEditor实现并注册到容器中，以供容器做类型转换的时候使用。默认情况下，BeanFactory容器不会为org.springframework.core.io.Resource类型提供相应的PropertyEditor，所以，如果我们想注入Resource类型的bean定义，就需要注册自定义的PropertyEditor到BeanFactory容器。不过，对于ApplicationContext来说，我们无需这么做，因为Application-Context容器可以正确识别Resource类型并转换后注入相关对象。

ApplicationContext启动伊始，会通过一个org.springframework.beans.support.ResourceEditorRegistrar来注册Spring提供的针对Resource类型的PropertyEditor实现到容器中，这个PropertyEditor叫做org.springframework.core.io.ResourceEditor。这样，Application-Context就可以正确地识别Resource类型的依赖了。ResourceEditor的实现是把配置文件中的路径让ApplicationContext作为ResourceLoader定位。

#### 在特定情况下，ApplicationContext的Resource加载行为
URL所接受的资源路径来说，通常开始都会有一个协议前缀，比如file:、http:、ftp:等。既然Spring使用UrlResource 对URL定位查找的资源进行了抽象，那么，同样也支持这样类型的资源路径。

在这个基础上，Spring还扩展了协议前缀的集合。ResourceLoader中增加了一种新的资源路径协议——classpath:，ResourcePatternResolver又增加了一种——classpath*:。

classpath*:与classpath:的唯一区别就在于，如果能够在classpath中找到多个指定的资源，则返回多个。我们可以通过这两个前缀改变某些ApplicationContext实现类的默认资源加载行为。

##### ClassPathXmlApplicationContext

当ClassPathXmlApplicationContext在实例化的时候，即使没有指明classpath:或者classpath*:等前缀，它会默认从classpath中加载bean定义配置文件，以下代码中演示的两种实例化方式效果是相同的：

```bash
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
//两者相同
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:conf/appContext.xml");
```
##### FileSystemXmlApplicationContext

FileSystemXmlApplicationContext，如果我们像如下代码那样指定conf/appContext.xml，它会尝试从文件系统中加载bean定义文件：
```bash
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/appContext.xml");
```
不过，我们可以像如下代码所示，通过在资源路径之前增加classpath:前缀，明确指定FileSystemXmlApplicationContext从classpath中加载bean定义的配置文件：
```bash
ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```
这时，FileSystemXmlApplicationContext就是从Classpath中加载配置，而不是从文件系统中加载。也就是说，它现在对应的是 ClassPathResource 类型的资源，而不是默认的 FileSystemResource类型资源。FileSystemXmlApplicationContext之所以如此，是因为它与org.springframework.core.io.FileSystemResourceLoader 一样，也覆写了DefaultResourceLoader的getRes-ourceByPath(String)方法，逻辑跟FileSystemResourceLoader一模一样。

# 国际化信息支持（I18n MessageSource）

对于Java中的国际化信息处理，主要涉及两个类，即java.util.Locale和java.util.ResourceBundle。

## Java SE提供的国际化支持

### Locale

不同的Locale代表不同的国家和地区，每个国家和地区在Locale这里都有相应的简写代码表示，包括语言代码以及国家代码，这些代码是ISO标准代码。常用的Locale都提供有静态常量，不用我们自己重新构造。一些不常用的Locale的则需要根据
相应的国家和地区以及语言来进行构造。

### ResourceBundle

ResourceBundle用来保存特定于某个Locale的信息（可以是String类型信息，也可以是任何类型的对象）。通常，ResourceBundle管理一组信息序列，所有的信息序列有统一的一个basename，然后特定的Locale的信息，可以根据basename后追加的语言或者地区代码来区分。

用properties文件来分别保存不同国家地区的信息，可以像下面这样来命名相应的properties文件：
```bash
messages.properties
messages_zh.properties
messages_zh_CN.properties
messages_en.properties
messages_en_US.properties
...
```
文件名中的messages部分称作ResourceBundle将加载的资源的basename，其他语言或地区的资源在basename的基础上追加Locale特定代码

有了ResourceBundle对应的资源文件之后，我们就可以通过ResourceBundle的getBundle(String baseName, Locale locale) 方法取得不同Locale对应的ResourceBundle，然后根据资源的键取得相应Locale的资源条目内容。通过结合ResourceBundle和Locale，我们就能够实现应用程序的国际化信息支持。

## MessageSource

Spring在Java SE的国际化支持的基础上，进一步抽象了国际化信息的访问接口，也就是org.springframework.context.MessageSource ，该接口定义如下：
```bash
public interface MessageSource {
  String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
  String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
  String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessage zException;
}
```
通过该接口，我们统一了国际化信息的访问方式。传入相应的Locale、资源的键以及相应参数，就可以取得相应的信息，再也不用先根据Locale取得ResourceBundle，然后再从ResourceBundle查询信息了。

### 方法

对MessageSource所提供的三个方法的简单说明如下：

#### String getMessage(String code, Object[] args, String defaultMessage, Localelocale)

根据传入的资源条目的键（对应方法声明中的code参数）、信息参数以及Locale来查找信息，如果对应信息没有找到，则返回指定的 defaultMessage 。

#### String getMessage(String code, Object[] args, Locale locale) throws NoSuch-

MessageException。与第一个方法相同，只不过，因为没有指定默认信息，当对应的信息找不到的情况下，将抛出NoSuchMessageException异常。

#### String getMessage(MessageSourceResolvable resolvable, Locale locale) throws-

NoSuchMessageException。使用MessageSourceResolvable对象对资源条目的键、信息参数等进行封装，将封住了这些信息的MessageSourceResolvable对象作为查询参数来调用以上方法。如果根据MessageSourceResolvable中的信息查找不到相应条目内容，将抛出NoSuchMessageException异常。

### 实现

Spring提供了三种 MessageSource 的实现，即 StaticMessageSource 、 ResourceBundleMessage-Source 和 ReloadableResourceBundleMessageSource 。这三种实现都可以独立于容器并在独立运行（Standalone形式）的应用程序中使用，而并非只能依托 ApplicationContext 才可使用。

#### org.springframework.context.support.StaticMessageSource 

MessageSource 接口的简单实现，可以通过编程的方式添加信息条目，多用于测试，不应该用于正式的生产环境。

#### org.springframework.context.support.ResourceBundleMessageSource

基于标准的java.util.ResourceBundle 而实现的 MessageSource ，对其父类 AbstractMessageSource的行为进行了扩展，提供对多个 ResourceBundle 的缓存以提高查询速度。同时，对于参数化的信息和非参数化信息的处理进行了优化，并对用于参数化信息格式化的 MessageFormat 实例也进行了缓存。它是最常用的、用于正式生产环境下的 MessageSource 实现。

#### org.springframework.context.support.ReloadableResourceBundleMessageSource

同样基于标准的 java.util.ResourceBundle 而构建的 MessageSource实现类 ，但通过其cacheSeconds 属性可以指定时间段，以定期刷新并检查底层的properties资源文件是否有变更。对于properties资源文件的加载方式也与 ResourceBundleMessageSource 有所不同，可以通过ResourceLoader 来加载信息资源文件。使用 ReloadableResourceBundleMessageSource 时，应该避免将信息资源文件放到classpath中，因为这无助于 ReloadableResourceBundle-MessageSource 定期加载文件变更。更多信息参照该类的Javadoc。

<div align="center"><img src="https://user-images.githubusercontent.com/37955886/114850703-a2ede900-9e13-11eb-97e6-20a2592c3a88.png"/></div> 

### MessageSourceAware和MessageSource的注入

ApplicationContext 启动的时候，会自动识别容器中类型为 MessageSourceAware 的bean定义，并将自身作为 MessageSource 注入相应对象实例中。如果真的某个业务对象需要依赖于 MessageSource 的话，直接通过构造方法注入或者setter方法注入的方式声明依赖就可以了。只要配置bean定义时，将 ApplicationContext 容器内部的那个 messageSource 注入该业务对象即可。



















