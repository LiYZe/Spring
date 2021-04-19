# Spring MVC

<div align="center"> <img src= "https://user-images.githubusercontent.com/37955886/115175859-67e30280-a0fe-11eb-9588-b2b6b71d1b7f.png"/></div>

# DispatcherServlet

## 体系结构

### HandleMapping 

其为mvc中url路径与控制器的映射，DispatcherServlert就是基于此组件来寻找对应的Control，找不到则报Not Found mapping的异常

#### 三种实现方式

BeanNameUrlHandlerMapping

基于IoC name中以“/”开头的bean时注册到映射

SimpleUrlHandlerMapping

基于手动配置url与control映射

RequestMappingHandleMapping

基于@RequestMapping注解配置对应映射

### Handlerdapter

Spring MVC采用适配器模式来适配调用指定Handler，根据Handler的不同种类采用不同的Adapter

#### Handler与HandlerAdapter对应关系

<div align="center"> <img src= "https://user-images.githubusercontent.com/37955886/115177550-0c1a7880-a102-11eb-93b4-3d696629837d.png"/></div>

### ViewResolver
视图仓库，基于viewName找到对应的view

### view
具体分析视图，解析生成HTML，返回

### HandlerExceptionResolver

用于指出当异常出现时，mvc如何处理。dispatcherServlet会调用org.springframework.web.servlet.DispatcherServlet#processHandlerException()方法，遍历handlerExceptionResolvers处理异常，完成之后返回errorView跳转异常视图。

### HandlerInterceptor

拦截器

## 关系

<div align="center"> <img src= "https://user-images.githubusercontent.com/37955886/115176606-04f26b00-a100-11eb-8224-00300fb5393d.png"/></div>

## mvc各组件执行流程

<div align="center"> <img src= "https://user-images.githubusercontent.com/37955886/115177134-26078b80-a101-11eb-96e9-15716ee6a32f.png"/></div>

（注：在执行healder之前会先执行handlerinterceptor）

