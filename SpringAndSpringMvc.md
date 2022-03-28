# 1.Spring框架

## 1.IOC配置

依赖注入  ``application.xml``

#### 使用spring ioc容器管理bean

##### 1.xml配置方法


```xml
<bean class="全包名" id="唯一标识"></bean>
```

  

**scope标签配置**

![](https://gitee.com/ingachin/mdimage/raw/master/image-20220323202337228.png)

_当scope配置singleton时对象实例将会在Spring容器启动时生成，当配置prototype时将会在创建对象时生成_


##### 2.注解配置方法

```java
@Compentent
```

使Spring ioc容器管理bean实现解耦

####  将spring管理的bean注入

##### 1.使用setter方法注入

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>

<bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
<property name="userDao" ref="userDao"/>
</bean>
```

##### 2.使用构造方法进行依赖注入

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
<constructor-arg name="userDao" ref="userDao"></constructor-arg>
</bean>
```

##### 使用注解进行注入

```java
@Autowrited//默认注入对应类型,如果存在多个相同的类则会报错，推荐使用下面方法进行注入

@AutoWrited
@Qualifier(value=“类名”)

@Resource(value="id")

```

*``@AutoWrited``和``@Resource``的区别

_@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入罢了。@Resource有两个属性是比较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。_

##### 4.注入properties类型数据(``配置数据源``)

###### 1.引入``properties``文件

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

###### 2.配置数据源

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
<property name="driverClass" value="${jdbc.driver}"/>
<property name="jdbcUrl" value="${jdbc.url}"/>
<property name="user" value="${jdbc.username}"/>
<property name="password" value="${jdbc.password}"/>
</bean>
```



#### Spring IOC注解开发


###### 0.Spring常用注解

![image-20220323204153132](https://gitee.com/ingachin/mdimage/raw/master/image-20220323204153132.png)

![image-20220323210640281](https://gitee.com/ingachin/mdimage/raw/master/image-20220323210640281.png)

###### 1.配置组件扫描

*配置组件扫描是为了配置的注解能够被Spring容器扫描到

将以下配置配置于`ApplicationContext.xml`下

```xml
<context:component-scan base-package="需要扫描的包名"></context:componentscan>
```

###### 2.Spring整合``junit``进行测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
//加载spring核心配置文件
//@ContextConfiguration(value = {"classpath:applicationContext.xml"})
//加载spring核心配置类
@ContextConfiguration(classes = {SpringConfiguration.class})

```



## 2.Spring集成Web环境  



_解决每次都要从ApplicationContext中获取Bean将ApplicationContext对象存储到servletContext域中_

#### 1.将ApplicationContext对象存储到``servletContext``域中

**在`Web.xml`配置中通过Spring集成的`ContextLoaderListener`存储，服务器将会再每次启动时自动创建ApplicationContext对象将其存储到`servletContext`域中**

```xml
<!--全局参数-->
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--Spring的监听器-->
<listener>
<listener-class>
org.springframework.web.context.ContextLoaderListener
</listener-class>
</listener>
```

#### 2.通过  ``WebApplicationContextUtils``工具类获取

```java
ApplicationContext applicationContex=WebApplicationContextUtils.getWebApplicationContext(servletContext);
```









# Spring MVC

### 1.Spring MVC配置  

#### 1.在`web.xml`配置Spring MVC的前端控制器`DispatcherServlet`

```xml
<servlet>
<servlet-name>DispatcherServlet</servlet-name>
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<init-param>
<param-name>contextConfigLocation</param-name>
<param-value>classpath:spring-mvc.xml</param-value>
</init-param>
<load-on-startup>1</load-on-startup>//当服务器启动时创建DispatcherServlet对象
</servlet>
<servlet-mapping>
<servlet-name>DispatcherServlet</servlet-name>
<url-pattern>/</url-pattern>
</servlet-mapping>
```

  

#### Spring-mvc config.xml配置

```xml
<context:component-scan base-package="com.itheima"/>
```

  

#### 使用注解配置`controller`

```java
@RequestMapping("/url")//配置访问地址,即可配置在方法上，也可以配置在类的上面

@***Mapping //***为请求方式 指定特定的请求方式访问该方法，通常用于风格的开发中


```





### 2.Spring MVC执行流程

![image-20220323212825342](https://gitee.com/ingachin/mdimage/raw/master/image-20220323212825342.png)

## 3.Spring MVC的请求与响应

### 1.Spring MVC响应

1.直接返回字符串数据

2.返回json数据

**只需要在方法上配置`@ResopnedsBody`注解，SpringMVC返回时将会自动将对象转为JSON数据**

**或者在类上配置`@RestController`注解，该类下的所有方法都会将JSON数据转化为对象进行返**

并且同时在Spring MVCconfig.xml中配置

```xml
<mvc:annotation-driven/>
```

![image-20220323214336850](https://gitee.com/ingachin/mdimage/raw/master/image-20220323214336850.png)

3.返回ModelandView

```java
ModelandView modelandview = new ModelandView();
...
return modelandview;
```

4.进行页面重定向  



### 2.Spring MVC获得请求数据



#### `Restful`风格获得数据

1.RequestMapping配置

```java
@RequestMapping("/{占位符}")
```

2.方法参数配置

```java
public R save(@PathVarible Interge currentPage){...};
```

#### 获得JSON数据

1.方法参数配置

```java
 public R save(@RquestBody Interge currentPage){...};
```

#### 数据乱码处理

在web.xml中配置如下属性

````xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
````

#### 当传入的数据与方法参数表不一致

使用`@RequestParam`注解进行参数的绑定

![image-20220323215527661](https://gitee.com/ingachin/mdimage/raw/master/image-20220323215527661.png)

#### `@RequstHeader`获得请求头

```java
@RequestHeader(value = "User-Agent",required = false) String
headerValue){
```

#### `@CookieValue`获取请求时所携带的cookie

```java
@CookieValue(value = "JSESSIONID",required = false) String jsessionid)
```

#### 配置文件上传

1.将表单配置为`enctype=“application/x-www-form-urlencoded`

2.配置文件资源上传解析器

```xml
<class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--上传文件总大小-->
<property name="maxUploadSize" value="5242800"/>
<!--上传单个文件的大小-->
<property name="maxUploadSizePerFile" value="5242800"/>
<!--上传文件的编码类型-->
<property name="defaultEncoding" value="UTF-8"/>
</bean>
```

3.参数类型为`MultipartFile[] uploadFiles`

### 3.Spring MVC对静态资源进行放行

**在web.xml中配置如下属性**

```xml
<mvc:default-servlet-handler/>
```



## 4.Spring MVC中的Interceptor配置

### 1.拦截器创建

**只需要实现`HandlerInterceptor`接口即可**

```java
public class MyHandlerInterceptor1 implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
    response, Object handler) {
        ...
        return true;
    }
    public void postHandle(HttpServletRequest request, HttpServletResponse
    response, Object handler, ModelAndView modelAndView) {
    	...
    }
    public void afterCompletion(HttpServletRequest request, HttpServletResponse
    response, Object handler, Exception ex) {
    	...
    }
}
```

​    

### 2.拦截器配置

#### 1.在SpringMVCConfig.xml中进行配置

在SpringMVCConfig.xml进行如下配置

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="拦截器"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 2.在WebConfig类中配置

```java
@Configuration
public class WebConfig implements WebMvcConfigurer{

    @Autowired
    private Tokeninterceptor tokeninterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        WebMvcConfigurer.super.addInterceptors(registry);
        registry.addInterceptor(tokeninterceptor).addPathPatterns("/**");
    }
}
```

##  5.Spring 异常统一处理

#### 1.使用Spring定义的`SimpleMappingExceptionResolver`

```xml
<bean
class=“org.springframework.web.servlet.handler.SimpleMappingExceptionResolver”>
    <property name=“defaultErrorView” value=“error”/>
    <property name=“exceptionMappings”>
<map> 
        <entry key="com.itheima.exception.MyException" value="error"/>
        <entry key="java.lang.ClassCastException" value="error"/>
</map>
</property>
</bean>
```



#### 2.自定义异常处理器

**Spring Boot方法**

```java
@RestControllerAspect
public class MyExceptionResolver  {
	@ExceptionHandler
    public R doEX(Exception e){
        return new R(false);
    }
}
```

**Spring方法**

##### 1.创建异常处理对象

```java
@Conpetent
public class MyExceptionResolver implements HandlerExceptionResolver {
@Override
    public R resolveException(HttpServletRequest request,
        HttpServletResponse response, Object handler, Exception ex) {
        return new R(false);
    }
}

```

##### 2.在SpringMVC配置中添加异常处理对象

```xml
<bean id="exceptionResolver" class="MyExceptionResolver"/>
```



## 3. Spring AOP

### 1.Aop原理

**实际上，AOP 的底层是通过 Spring 提供的的动态代理技术实现的。在运行期间，Spring通过动态代理技术动态 的生成代理对象，代理对象方法执行时进行增强功能的介入，在去调用目标对象的方法，从而完成功能的增强.**

---

### 2.Spring底层实现AOP的两种方式

![image-20220324214600541](https://gitee.com/ingachin/mdimage/raw/master/image-20220324214600541.png)

### 3.AOP常见概念

![image-20220324214652005](https://gitee.com/ingachin/mdimage/raw/master/image-20220324214652005.png)

### 4. 在`Application.xml`配置织入关系

```xml
<aop:config>
<!--引用myAspect的Bean为切面对象-->
<aop:aspect ref="myAspect">
<!--配置Target的method方法执行时要进行myAspect的before方法前置增强-->
<aop:before method="before" pointcut="execution(public void
com.itheima.aop.Target.method())"></aop:before>
</aop:aspect>
</aop:config>

```

  ```xml
  <aop:通知类型 method=“切面类中方法名” pointcut=“切点表达式"></aop:通知类型>
  ```

##### 常用配置

![image-20220324215020383](https://gitee.com/ingachin/mdimage/raw/master/image-20220324215020383.png)

##### 使用注解进行配置

1.在通知(advice)上配置`@Advice`注解  

2.将切点对象以及通知的管理权交给Spring

3.在通知的方法上配置`@Before("execution(切面表达式)")`

4.在Spring中配置扫描

```xml
<!--组件扫描-->
<context:component-scan base-package="aop"/>
<!--aop的自动代理-->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

常用通知配置

![image-20220324215507677](https://gitee.com/ingachin/mdimage/raw/master/image-20220324215507677.png)

### 5.切点表达式语法

![image-20220324214920311](https://gitee.com/ingachin/mdimage/raw/master/image-20220324214920311.png)



## 4.Spring声明式事务控制(基于AOP)

### 1.事务隔离级别

![image-20220324220701279](https://gitee.com/ingachin/mdimage/raw/master/image-20220324220701279.png)



---

### 2.事务传播行为

常用

 REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选择（默认值） 

 SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）

---

### 3.配置事务控制

##### 1.xml配置事务控制  



```xml
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
<!--事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
    	<tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="切点表达式"></aop:advisor>
</aop:config>
```

##### 2.注解配置事务控制  

###### 1.在需要事务控制的方法上添加`@Transactional`注解

###### 2.配置ApplicationContext.xml

```xml
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<context:component-scan base-package="/Mapper"/>
<!--事务的注解驱动-->

<tx:annotation-driven/>

```

  





