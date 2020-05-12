#### Spring MVC基础应用

##### 经典三层代码架构

​	开发架构通常基于两种形式，一种是C/S（客户端/服务器）架构，一种是B/S架构（浏览器/服务器）。在Java EE开发中，几乎基于B/S架构，在B/S架构中标准的三层架构为：表现层、业务层、持久层

​	每一层分工明确，好处是解耦及便于维护与分工

​	表现层（web层）：负责接收客户端请求，向客户端响应结果。表现层包括展示层和控制层：展示层负责结果的展示，控制层负责接收请求。表现层的设计一般使用MVC模型

​	业务层（Service层）：负责业务逻辑处理

​	持久层（dao层）：负责数据持久化，包括数据库和数据访问层，数据库是持久化的载体，数据访问层是业务层和持久层交互的接口

##### MVC模式

​	Model View Controller（模型-视图-控制器），是用于设计创建web应用程序表现层的模式。

​	Model模型：封装数据

​	View视图：展示数据

​	Controller控制器：处理与用户交互的部分

##### Spring MVC处理流程

​	![image-20200506143132154](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200506143132154.png)

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求后调用处理器映射器HandlerMapping，HandlerMapping根据请求url找到对应的Handler(处理器)，生成处理器对象及处理器拦截器返回给DispatcherServlet
3. DispatcherServlet调用处理器适配器HandlerAdapter
4. 处理器适配器去调用Handler
5. Handler执行完成后返回ModelAndView
6. 处理器适配器将ModelAndView返回给DispatcherServlet
7. DispatcherServlet请求视图解析器去进行视图解析
8. 视图解析器向DispatcherServlet返回view
9. DispatcherServlet进行视图渲染（将模型数据填充到request域中）
10. DispatcherServlet向用户返回结果

#####  九大组件

###### HandlerMapping处理器映射器

​	根据请求URL查找处理器Handler

###### HandlerAdapter处理器适配器

​	找到合适的适配器调用处理器

###### HandlerExceptionResolver异常处理器

​	处理Handler产生异常的情况，它的作用是根据异常设置ModelAndView，之后交给渲染方法进行渲染，渲染方法会将ModelAndView渲染成页面

###### ViewResolver视图解析器

​	将String类型的视图名和Locale解析为View类型的视图，View是用来渲染页面的，它将程序返回的数据填入模板中，生成html文件

###### RequestToViewNameTranslator默认视图名转换组件

​	从请求中获取ViewName（ViewResolver根据ViewName查找View）

###### LocaleResolver国际化组件

​	从请求种解析出Locale

###### ThemeResovler主题组件

​	解析主题

###### MultipartResolver多元素解析器

​	通过将普通请求包装成MultipartHttpServletRequest来使实现具有上传功能

###### FlashMapManager flash属性管理组件

​	可用于重定向时的参数传递

##### url-pattern配置

​	*.xxx ：拦截指定后缀xxx的url

​	/：拦截servlet和静态资源；不会拦截jsp

​	/*：拦截所有（servlet、jsp、静态资源）

##### 数据封装机制

- ModelAndView
- ModelMap + viewName
- Model + viewName
- Map + viewName

##### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd
">

    
    <!--开启controller扫描 springmvc负责controller层-->
    <context:component-scan base-package="com.xcl.controller"/>


    <!--配置springmvc的视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>


    <!--
        自动注册最合适的处理器映射器，处理器适配器(调用handler方法)
    -->
    <mvc:annotation-driven/>

    <!--静态资源配置，方案一-->
    <!--
        原理：添加该标签配置之后，会在SpringMVC上下文中定义一个DefaultServletHttpRequestHandler对象
             这个对象如同一个检查人员，对进入DispatcherServlet的url请求进行过滤筛查，如果发现是一个静态资源请求
             那么会把请求转由web应用服务器（tomcat）默认的DefaultServlet来处理，如果不是静态资源请求，那么继续由
             SpringMVC框架处理
    -->
    <!--<mvc:default-servlet-handler/>-->



    <!--静态资源配置，方案二，SpringMVC框架自己处理静态资源
        mapping:约定的静态资源的url规则
        location：指定的静态资源的存放位置

    -->
    <mvc:resources location="classpath:/"  mapping="/resources/**"/>
    <mvc:resources location="/WEB-INF/js/" mapping="/js/**"/>


    <mvc:interceptors>
        <!--拦截所有handler-->
        <!--<bean class="com.xcl.interceptor.MyIntercepter"/>-->   
            
        <mvc:interceptor>
            <!--配置当前拦截器的url拦截规则，**代表当前目录下及其子目录下的所有url-->
            <mvc:mapping path="/**"/>
            <!--exclude-mapping可以在mapping的基础上排除一些url拦截-->
            <mvc:exclude-mapping path="/demo/**"/>
            <bean class="com.xcl.interceptor.MyIntercepter01"/>
        </mvc:interceptor>
    </mvc:interceptors>


    <!--多元素解析器
        id固定为multipartResolver
    -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--设置上传文件大小上限，单位是字节，-1代表没有限制也是默认的-->
        <property name="maxUploadSize" value="5000000"/>
    </bean>

</beans>
```

##### web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>demo</display-name>

  <!--springmvc提供的针对post请求的编码过滤器-->
  <filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <!--配置springmvc请求方式转换过滤器，会检查请求参数中是否有_method参数，如果有就按照指定的请求方式进行转换-->
  <filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
    
    <!--启动spring框架-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

    <!--前端控制器（启动springmvc）-->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>

    <!--
      方式一：带后缀，比如*.action  *.do *.aaa
             该种方式比较精确、方便，在以前和现在企业中都有很大的使用比例
      方式二：/ 不会拦截 .jsp，但是会拦截.html等静态资源（静态资源：除了servlet和jsp之外的js、css、png等）

            为什么配置为/ 会拦截静态资源？？？
                因为tomcat容器中有一个web.xml（父），你的项目中也有一个web.xml（子），是一个继承关系
                      父web.xml中有一个DefaultServlet,  url-pattern 是一个 /
                      此时我们自己的web.xml中也配置了一个 / ,覆写了父web.xml的配置
            为什么不拦截.jsp呢？
                因为父web.xml中有一个JspServlet，这个servlet拦截.jsp文件，而我们并没有覆写这个配置，
                所以springmvc此时不拦截jsp，jsp的处理交给了tomcat


            如何解决/拦截静态资源这件事？


      方式三：/* 拦截所有，包括.jsp
    -->
    <!--拦截匹配规则的url请求，进入springmvc框架处理-->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>

```

##### RESTful API

​	RESTful是一种软件架构风格，它提倡的是一个资源定位及资源操作的风格

​	REST(Representational State Transfer)资源表现层状态转移

​	资源：网络上的一个实体。可以用URI定位它

​	表现层：把资源具体呈现出来的形式，叫做它的表现层，比如文本可用txt表现，也可以用HTML格式、XML格式、JSON格式或者二进制格式

​	状态转移：每发出一个请求，就代表一次客户端与服务器的交互

​	HTTP是一种无状态的协议，所有的状态都保存在服务端

---

#### Spring MVC高级技术

##### 监听器 VS 过滤器 VS 拦截器

监听器：实现了ServletContextLister接口的组件，在web应用启动时初始化，然后一直运行监视

过滤器：对request请求进行过滤，作用在servlet之前。可以配置为/*对所有资源（servlet、js/css）进行过滤

拦截器：不会拦截静态资源的访问，之拦截Handler的请求

​	三个拦截点

​		Handler执行前拦截一次

​		Handler执行完毕跳转页面前拦截一次

​		跳转页面之后拦截一次

​	拦截器执行流程

​	执行顺序与配置文件中所定义的拦截器的顺序相关

注：监听器、过滤器属于Java EE的技术，拦截器是框架（springMVC）的技术。所以可以看到servlet、filter、listener配置在web.xml中，而interceptor配置在框架的配置文件中

##### multipart文件上传

​	引jar包

```xml
<dependency>  
    <groupId>commons-fileupload</groupId>  				<artifactId>commons-fileupload</artifactId>  		<version>1.3.1</version> 
</dependency>

```

​	配置文件上传解析器

```xml
<!--id固定-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">        
    <property name="maxUploadSize"value="1000000000"/> </bean>
```

​	前端处理：method="post" enctype=""multipart/form-data" 上传组件

​	后端处理：MultipartFile uploadFile

##### 异常处理机制

​	全局配置：编写异常处理类使用@ControllerAdvice注解类、@ExceptionHandler注解方法

```java
@ControllerAdvice public class GlobalExceptionResolver {
    @ExceptionHandler(ArithmeticException.class)    public ModelAndView handleException(ArithmeticException exception, HttpServletResponse response) {        ModelAndView modelAndView = new ModelAndView();        modelAndView.addObject("msg",exception.getMessage());        modelAndView.setViewName("error");        return modelAndView;    } }

```



##### 重定向 VS 转发

​	转发：是web应用内部的行为，客户端无法感知，属于同一个请求

​	重定向：web应用告知客户端应该访问的地址，由客户端发起访问，为两次不同的请求。所以在重定向时要考虑参数丢失的问题

​	参数丢失的两种解决方案

​		1、手动拼接参数

​		2、将参数放入RedirectAttributes的属性中	

```java
 redirectAttributes.addFlashAttribute("name",name); 
```



---

#### 自定义MVC框架

​	code

---

#### Spring MVC源码刨析

###### DispatcherServlet继承结构

![image-20200507154041486](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200507154041486.png)

###### Spring MVC请求处理的大致流程

![image-20200507154427254](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200507154427254.png)

关键时机点

- getHandle()：获取能够处理当前请求的执行链

  handlerMappins = {BeanNameUrlHandlersMapping,RequestMappingHandlerMapping}	

- getHandleAdapter()：遍历HandlerAdapter，找到支持当前Handler的Adapter

  handlerAdapters = {HttpRequestHandlerAdapter,SimpleControllerHandlerAdapter,RequestMappingHandlerAdapter}	

- ha.handle()：处理器处理请求，返回ModelAndView对象

- processDispatchResult()：异常处理 & render完成渲染

---

#### SSM整合

###### 整合策略

​	先整个spring+mybatis，最后加入spring mvc

###### 整合内容

​	目标

​		数据库连接池以及事务管理都交给spring来管理

​		SqlSessionFactory作为单例对象放入spring容器中

​		Mapper动态代理对象交给spring管理

​	所需Jar包

​		Junit测试jar

​		Mybatis相关jar包

​		Spring相关jar包

​		Mybatis与Spring的整合包

​		Mysql数据库驱动

​		Druid数据库连接池的jar包

​	pom

```xml
<!--junit--> 
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.12</version>  
    <scope>test</scope> 
</dependency> 
<!--mybatis-->
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.4.5</version> 
</dependency> 
<!--spring -->
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.1.12.RELEASE</version> 
</dependency> 
<dependency>
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>5.1.12.RELEASE</version> 
</dependency> <
dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.1.12.RELEASE</version> 
</dependency> 
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-tx</artifactId>  
    <version>5.1.12.RELEASE</version> 
</dependency> 
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-aop</artifactId>  
    <version>5.1.12.RELEASE</version> 
</dependency> 
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.8.9</version> </dependency>
<!--mybatis整合spring -->
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>2.0.3</version> 
</dependency> 
<!--MySQL数据库驱动--> 
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.46</version> 
</dependency> 
<!--druid数据库连接池--> 
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.21</version> 
</dependency>
```



---

#### Spring Data JPA

###### 概述

​	是spring基于JPA规范的基础上封装的一套JPA应用框架

###### 应用

1. 使用现成的API

2. JQPL语言

   ```java
   @Query("from Resume  where id=?1 and name=?2")
   ```

3. 原生SQL语句

   ```java
   @Query(value = "select * from tb_resume  where name like ?1 and address like ?2",nativeQuery = true)
   ```

4. 方法命名规则

   ```
   *  方法名以findBy开头
   *          -属性名（首字母大写）
   *                  -查询方式（模糊查询、等价查询），如果不写查询方式，默认等价查询
   
   举例：根据name尽心模糊查询
   findByNameLike(String name)
   ```

5. 动态查询，将查询条件封装为一个Specification对象

```java
  // 动态查询，查询单个对象
    @Test
    public void testSpecfication(){

        /**
         * 动态条件封装
         * 匿名内部类
         *
         * toPredicate：动态组装查询条件
         *
         *      借助于两个参数完成条件拼装，，， select * from tb_resume where name='张三'
         *      Root: 获取需要查询的对象属性
         *      CriteriaBuilder：构建查询条件，内部封装了很多查询条件（模糊查询，精准查询）
         *
         *      需求：根据name（指定为"张三"）查询简历
          */

        Specification<Resume> specification = new Specification<Resume>() {
            @Override
            public Predicate toPredicate(Root<Resume> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                // 获取到name属性
                Path<Object> name = root.get("name");

                // 使用CriteriaBuilder针对name属性构建条件（精准查询）
                Predicate predicate = criteriaBuilder.equal(name, "张三");
                return predicate;
            }
        };


        Optional<Resume> optional = resumeDao.findOne(specification);
        Resume resume = optional.get();
        System.out.println(resume);

    }
```





