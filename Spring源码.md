# Spring 

---

## 1.BeanFactory与ApplicationContext接口

![image-20220427145813921](https://gitee.com/ingachin/mdimage/raw/master/image-20220427145813921.png)



### BeanFactory和ApplicationContext的关系

1. AppxlicationContext间接继承BeanFactory
2. BeanFactory才是Spring核心容器，主要的ApplicationContext都组合的他的方法
3. BeanFactory是ApplicationContext的一个成员变量  

![image-20220427151214019](https://gitee.com/ingachin/mdimage/raw/master/image-20220427151214019.png)

### BeanFactory的作用

1. 提供getBean

2. 控制反转，依赖注入以及Bean的生命周期都由BeanFactory的实现类`(DefaultListableBeanFactory)`来提供

3. `(DefaultListableBeanFactory)`将所有的单例对象都存在`DefaultSingletonRegistry`的ObjectsMap中可以通过反射的方式获取到

   ```java
   public static void main(String[] args) throws Exception {
       ConfigurableApplicationContext context = SpringApplication.run(SpringSourceApplication.class, args);
       Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
       ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
       singletonObjects.setAccessible(true);
       Map<String, Object> stringObjectMap = (Map<String, Object>) singletonObjects.get(beanFactory);
       stringObjectMap.entrySet().stream().filter(e -> e.getKey().startsWith("component")).forEach(e -> {
           System.out.println(e.getKey() + " " + e.getValue());
       });
   }
   ```

   

![image-20220427151107608](https://gitee.com/ingachin/mdimage/raw/master/image-20220427151107608.png)



### Application相较BeanFactory扩展了什么

![image-20220427165412480](https://gitee.com/ingachin/mdimage/raw/master/image-20220427165412480.png)

1. MessageSource 对于不同语言后台消息的转换(处理国际化资源的能力)   



2. ResourcePatternResolver 根据通配符`(classpath*:)`获取资源
3. ApplicationEventPublisher发布事件对象  

```java
System.out.println(context.getEnvironment().getProperty("java_home"));
```

4. Eviromentcapble 处理环境变量

![image-20220427170704487](https://gitee.com/ingachin/mdimage/raw/master/image-20220427170704487.png)

## 2.BeanFactory实现

```java
@Slf4j
public class TestBeanFactory {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //创建关于Bean定义(class,scope,initMethods,destroyMethods)
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
        //将config注册到beanFactory中
        beanFactory.registerBeanDefinition("config",beanDefinition);
        //向beanFactory中添加后处理器，beanFactory并不存在解析注解的能力，需要依赖后处理器进行解析注解，进行bean的创建
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
        //从beanFactory中获取beanFactory后处理器，并且一次执行解析注解
        beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values().stream().forEach(postProcess -> {
            postProcess.postProcessBeanFactory(beanFactory);
        });
        //输出beanFactory中的对象
        Arrays.stream(beanFactory.getBeanDefinitionNames()).forEach(System.out::println);
        //Bean后处理器，针对bean的声明周期的各个阶段进行拓展，例如@AutoWired,@Resource
        //处理bean
      beanFactory.getBeansOfType(BeanPostProcessor.class).values().stream().forEach(postProcess -> {
            beanFactory.addBeanPostProcessor(postProcess);
        });
        System.out.println(beanFactory.getBean(Bean1.class).getBean2());
    }


    @Configuration
    @Slf4j
    static class Config {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }
    }

    @Slf4j
    @Data
    static class Bean1 {
        @Autowired
        Bean2 bean2;

        Bean1() {
            log.info("Bean1 constructer");
        }
    }

    @Slf4j
    static class Bean2 {
        Bean2() {
            log.info("Bean2 constructer");
        }
    }


}

```

1. 创建关于Bean定义(class,scope,initMethods,destroyMethods)

2. beanFactory并不存在解析注解的能力，需要依赖beanFactory 后处理器进行解析注解，进行bean的创建

3. Bean后处理器，针对bean的声明周期的各个阶段进行拓展，例如@AutoWired,@Resource
4.  首先添加后处理器到BeanFactory的一个Bean，后面建立后处理器与BeanFactory的联系
5. 刚开始仅仅保存BeanDefination 不创建Bean，当Bean需要使用时才回去创建Bean对象，可以通过`beanFactory.preInstantiateSingletons();`提前创建全部单例对象

![image-20220430120948490](https://gitee.com/ingachin/mdimage/raw/master/image-20220430120948490.png)



### Bean后处理器  

1. 后处理器的顺序决定了哪个注解先被解析

假如同时使用

```java
@Autowired  
@Resource("bean4")
private Inter bean3;
```

变量中注入的bean为bean3

因为Bean处理器Autowired在前

可以通过比较器实现顺序的修改

![image-20220430122414964](https://gitee.com/ingachin/mdimage/raw/master/image-20220430122414964.png)

`beanFactory.getDependencYComparator()`的来源为AnnotationConfigUtils所添加![image-20220430122504595](https://gitee.com/ingachin/mdimage/raw/master/image-20220430122504595.png)



## 3.ApplicationContext的实现  

继承与实现关系图

![image-20220430143236600](https://gitee.com/ingachin/mdimage/raw/master/image-20220430143236600.png)

![image-20220430143247257](https://gitee.com/ingachin/mdimage/raw/master/image-20220430143247257.png)

### 1.ClassPathXmlApplicationContext

```java
public class ApplicationContextTest {
    public static void main(String[] args) {
        ApplicationContextTest contextTest = new ApplicationContextTest();
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("b01.xml");
        Arrays.stream(context.getBeanDefinitionNames()).forEach(System.out::println);
        Bean01 bean = context.getBean(Bean01.class);
        System.out.println(bean.getBean02());
    }

    static class Bean01{

        private Bean02 bean02;

        public void setBean02(Bean02 bean02){
            this.bean02 = bean02;
        }

        public Bean02 getBean02(){
            return bean02;
        }

    }

    static class Bean02{

    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bean02" class="cn.maisann.springdemo.ApplicationContextTest.Bean02"/>

    <bean id="bean01" class="cn.maisann.springdemo.ApplicationContextTest.Bean01">
        <property name="bean02" ref="bean02"/>
    </bean>

</beans>
```

#### ClassPathXmlApplicationContext底层原理

```java
public static void main(String[] args) {
        /*ApplicationContextTest contextTest = new ApplicationContextTest();
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("b01.xml");
        Arrays.stream(context.getBeanDefinitionNames()).forEach(System.out::println);
        Bean01 bean = context.getBean(Bean01.class);
        System.out.println(bean.getBean02());*/
        //在ApplicationContextTest底层实现Bean加载原理
        //首先创建DefaultListableBeanFactory对象
        DefaultListableBeanFactory defaultListableBeanFactory = new DefaultListableBeanFactory();
        //通过XmlBeanDefinitionReader读取配置文件
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(defaultListableBeanFactory);
        reader.loadBeanDefinitions("b01.xml");
        //Bean定义加载成功
        Arrays.stream(defaultListableBeanFactory.getBeanDefinitionNames()).forEach(System.out::println);
    }
```

1. 在ApplicationContextTest底层实现Bean加载原理
2. 首先创建DefaultListableBeanFactory对象

3. 通过XmlBeanDefinitionReader读取配置文件

### 2.AnnotationConfigApplicationContext

```java
public class ApplicationContextTest {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        Arrays.stream(context.getBeanDefinitionNames()).forEach(System.out::println);
    }

    static class Bean01{

        private Bean02 bean02;

        public void setBean02(Bean02 bean02){
            this.bean02 = bean02;
        }

        public Bean02 getBean02(){
            return bean02;
        }

    }

    static class Bean02{

    }

    static class Config{
        @Bean
        public Bean01 bean01(Bean02 bean02){
            Bean01 bean01 =  new Bean01();
            bean01.setBean02(bean02);
            return bean01;
        }
        
        @Bean
        public Bean02 bean02(){
            return new Bean02();
        }
    }

}
```

配置类在容器中也是一个类

> 也会注册处理后处理器(ClassPathApplicationContext不会自动添加，需要在xml文件中配置`<context:annotation-condig/>`)

![image-20220430150915755](https://gitee.com/ingachin/mdimage/raw/master/image-20220430150915755.png)

### 3.AnnotationConfigServletWebServerApplicationContext(用于web环境)  

```java
public static void main(String[] args) {
        AnnotationConfigServletWebServerApplicationContext context = new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);

    }

    @Configuration
    static class WebConfig{
        //基于Servlet的web服务器
        @Bean
        public ServletWebServerFactory servletWebServerFactory(){
            return new TomcatServletWebServerFactory();
        }
        //配置dispathcherServlet
        @Bean
        public DispatcherServlet dispatcherServlet(){
            return new DispatcherServlet();
        }
        //注册DispatcherServlet到Tomcat服务器
        @Bean
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet){
            return new DispatcherServletRegistrationBean(dispatcherServlet,"/");
        }
        @Bean("/hello")//当浏览器访问/hello路径时即可访问到该控制器
        public Controller controller1(){
            return ((request, response) -> {
                response.getWriter().write("hello");
                return null;
            });
        }
    }
```

![image-20220430153215483](https://gitee.com/ingachin/mdimage/raw/master/image-20220430153215483.png)

## 4.Bean的生命周期

```java

@Component
@Slf4j
public class LifeCycleBean {

   public LifeCycleBean(){};


   @Autowired
    public void autowired(@Value("${JAVA_HOME}") String home){
        log.info("home=====>"+home);
    }

    @PostConstruct
    public void constructor(){
        log.info("cons");
    }

    @PreDestroy
    public void destroy(){
        log.info("destory");
    }

}
```

> 执行顺序
>
> 1. 构造方法
> 2. 依赖注入方法
> 3. Bean的初始化方法
> 4. Bean的销毁方法

### 自定义Bean后处理器

BeanFactory中并不存在如@AutoWried @Resourece . . . 需要通过后处理器实现功能的增强

自定义后处理器代码

```java

@Component
public class MyBeanPostProcessor implements DestructionAwareBeanPostProcessor, InstantiationAwareBeanPostProcessor {
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.debug("<<<<<<<<实例化之前执行，这里的返回对象会替换掉原本的bean");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.debug("<<<<<<<<实例化之后执行，返回false则会跳过依赖注入环节");
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.debug("<<<<<<<<依赖注入阶段，如 @AutoWired ,@Value, @Resource");
        }
        return pvs;
    }

    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.debug("<<<<<<<<销毁之前执行，如@PreDestory");
        }
    }

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.debug("<<<<<<<<初始化之前执行，返回对象会替换原本Bean，如@PostConstruct,@ConfigurationProperties");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("LifeCycleBean")){
            log.debug("<<<<<<<<初始化之后执行，返回对象会替换原本Bean,代理增强");
        }
        return bean;
    }
}

```



### 模板方法设计模式    

> 通过该种方式实现功能的增强

```java

public class TemplateMethodTest {
    public static void main(String[] args) {
        MyFactoryBean factoryBean = new MyFactoryBean();
        factoryBean.addBeanPostProcessor(bean -> {
            System.out.println("===================> @Autowired");
        });
        factoryBean.addBeanPostProcessor(bean -> {
            System.out.println("===================> @Resource");
        });
        factoryBean.getBean();
    }

    static class MyFactoryBean{

        List<BeanPostProcessor> beanPostProcessors = new ArrayList<>();

        public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor){
            beanPostProcessors.add(beanPostProcessor);
        }

        public Object getBean(){
            Object obj = new Object();
            System.out.println("构造");
            System.out.println("依赖注入");
            beanPostProcessors.stream().forEach(beanPostProcessor -> {
                beanPostProcessor.injectProperties(obj);
            });
            System.out.println("销毁");
            return obj;
        }
    }

    static interface BeanPostProcessor{
        public void injectProperties(Object bean);
    }

}
```



### Bean的后处理器  

添加后处理器代码

```java

@Slf4j
public class A04Test {
    public static void main(String[] args) {
        GenericApplicationContext context = new GenericApplicationContext();

        context.registerBean("bean1",Bean1.class);
        context.registerBean("bean2",Bean2.class);
        context.registerBean("bean3",Bean3.class);

        context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());//增加解析@Value("")括号中字段的能力
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);//增加解析@AutoWired 和 @Value的能力
        context.registerBean(CommonAnnotationBeanPostProcessor.class);//处理@Resource 和 @PreConStruct @PreDestroy
        //先解析@Resource后解析@AutoWired
        //BeanFactory将会对后处理器进行排序，CommonAnnotationBeanPostProcessor在前，AutowiredAnnotationBeanPostProcessor在后
        context.refresh();
        context.close();

    }


}

```

Bean代码

```java

    @Slf4j
    static class Bean1{

        String Home;

        private Bean2 bean2;

        @Autowired
        public void setBean2(Bean2 Bean2){
            log.info("bean2 inject!");
            this.bean2 = bean2;
        }

        private Bean3 bean3;

        @Resource
        public void setBean3(Bean3 bean3){
            log.info("bean3 inject!");
            this.bean3 = bean3;
        }

        @PostConstruct
        public void init(){
            log.info("init.......");
        }

        @PreDestroy
        public void destroy(){
            log.info("destory..........");
        }

        @Autowired
        public void setHome(@Value("${JAVA_HOME}") String Home){
            log.info(Home);
            this.Home = Home;
        }

        @Override
        public String toString() {
            return "Bean1{" +
                    "Home='" + Home + '\'' +
                    ", bean2=" + bean2 +
                    ", bean3=" + bean3 +
                    '}';
        }
    }

    static class Bean2{

    }

    static class Bean3{

    }



        @ConfigurationProperties(prefix = "java")
        @Data
        public class Bean04 {
            String home;
            String version;
        }

```

1.  `AutowiredAnnotationBeanPostProcessor`为解析@Autowired和@Value的能力的后处理器
2. `CommonAnnotationBeanPostProcessor`为解析@Resource,@PreConstructor,@PreDestory的能力  
3. `ConfigurationPropertiesBindingPostProcessor`解析`@ConfigurationProperties`的能力 

#### AutowiredAnnotationBeanPostProcessor运行分析

```java

public class AutTest {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        beanFactory.registerSingleton("bean2",new Bean2());
        beanFactory.registerSingleton("bean3",new Bean3());

        beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());//增加解析@Value能力
        
        AutowiredAnnotationBeanPostProcessor postProcessor = new AutowiredAnnotationBeanPostProcessor();
        postProcessor.setBeanFactory(beanFactory);//设置BeanFactory，使其成为依赖注入来源

        Bean1 bean1  = new Bean1();

        System.out.println(bean1);
        postProcessor.postProcessProperties(null,bean1,"bean1");//解析@Autowired 和 @Value
        System.out.println(bean1);

    }
}

```



 postProcessProperties方法会去调用`findAutowiringMetadata`方法获取所有加了AutoWired的值和字段

```java

public class AutTest {
    public static void main(String[] args) throws Exception{
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        beanFactory.registerSingleton("bean2",new Bean2());
        beanFactory.registerSingleton("bean3",new Bean3());

        beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());//增加解析@Value能力

        AutowiredAnnotationBeanPostProcessor postProcessor = new AutowiredAnnotationBeanPostProcessor();
        postProcessor.setBeanFactory(beanFactory);//设置BeanFactory，使其成为依赖注入来源

        Bean1 bean1  = new Bean1();
        
        Method findAutowiringMetadata = AutowiredAnnotationBeanPostProcessor.class.getDeclaredMethod("findAutowiringMetadata", String.class, Class.class, PropertyValues.class);
        findAutowiringMetadata.setAccessible(true);
        InjectionMetadata metadata = (InjectionMetadata) findAutowiringMetadata.invoke(postProcessor, "bean1", Bean1.class, null);

        System.out.println(metadata);

    }
}

```

![image-20220502154015574](https://gitee.com/ingachin/mdimage/raw/master/image-20220502154015574.png)

最后InjectionMetadata调用`inject`方法注入值

```java
metadata.inject(bean1,"bean1",null);
```

![image-20220502154501239](https://gitee.com/ingachin/mdimage/raw/master/image-20220502154501239.png)

```java
beanFactory.addEmbeddedValueResolver(new StandardEnvironment()::resolveRequiredPlaceholders); // ${}的解析器
```

##### InjectionMetadata中inject的实现    

###### 1.成员变量inject实现

```java
        //inject实现
        Field bean3 = Bean1.class.getDeclaredField("bean3");//通过反射获取成员变量
        bean3.setAccessible(true);
        DependencyDescriptor descriptor = new DependencyDescriptor(bean3,false);
        Object o = beanFactory.doResolveDependency(descriptor, null, null, null);//通过BeanFactory获根据需要注入的类型从容器中获取对象
//        System.out.println(o);
        bean3.set(bean1,o);//通过反射设置成员变量的值
        System.out.println(bean1);
```

###### 2.方法参数inject实现

```java
        Method setBean2 = Bean1.class.getDeclaredMethod("setBean2", Bean2.class);
        DependencyDescriptor descriptor1 = new DependencyDescriptor(new MethodParameter(setBean2,0),false);
        Object o1 = beanFactory.resolveDependency(descriptor1,null,null,null);
        setBean2.invoke(bean1,o1);
        System.out.println(bean1);
```

## 5.BeanFactory后处理器

### 1.ConfigurationClassPostProcessor后处理器  

Bean

```java

@Configuration
@ComponentScan("cn.maisann.springsource.component")
public class MyConfig {

    @Bean
    public Bean1 bean1(){
        return new Bean1();
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }

    @Bean(initMethod = "init")
    public DruidDataSource dataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql//localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }
}
```

test

```java
 @Test
    public void test(){
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean(ConfigurationClassPostProcessor.class);
        context.registerBean("config", MyConfig.class);
        context.refresh();
        Arrays.stream(context.getBeanDefinitionNames()).forEach(System.out::println);
    }
```

ConfigurationClassPostProcessor可以识别`@ComponentScan`注解`@Bean`注解`@Import`

MapperScannerConfigurer可以识别@Mapper注解，Mybatis整合时常见

#### 实现原理

```java
@SpringBootTest
public class BeanFactoryPostprocessorTest {

    @Test
    public void test() throws Exception{
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean("config", MyConfig.class);
        ComponentScan annotation = AnnotationUtils.findAnnotation(MyConfig.class, ComponentScan.class);//寻找已经添加@ComponentScan注解的类
        CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory(); // 根据解析后的资源文件读取类的内容
        AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator(); //生成Bean的名字
        if(annotation != null){
            for (String  basePackage: annotation.basePackages()) {
                basePackage = "classpath*:" + basePackage.replace(".","/") + "/**/*.class";
                //   将 cn.maisann.component.** -> classpath*:cn/maisann/component/**/**.class 路径，方便后续扫描
                PathMatchingResourcePatternResolver matchingResourcePatternResolver = new PathMatchingResourcePatternResolver();
                Resource[] resources = matchingResourcePatternResolver.getResources(basePackage); //获取资源文件
                for (Resource resource : resources) {
                    MetadataReader metadataReader = factory.getMetadataReader(resource);
                    System.out.println("类名:"+metadataReader.getClassMetadata().getClassName());
                    AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
                    System.out.println("是否添加@Component注解 "+ annotationMetadata.hasAnnotation(Component.class.getName()));
                    System.out.println("是否添加@Component 派生注解 "+ annotationMetadata.hasMetaAnnotation(Component.class.getName()));
                    if(annotationMetadata.hasAnnotation(Component.class.getName()) ||
                            annotationMetadata.hasMetaAnnotation(Component.class.getName())){
                        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.
                                genericBeanDefinition(metadataReader.getClassMetadata().getClassName()).
                                getBeanDefinition();
                        DefaultListableBeanFactory defaultListableBeanFactory = context.getDefaultListableBeanFactory();
                        String name = generator.generateBeanName(beanDefinition, context);
                        defaultListableBeanFactory.registerBeanDefinition(name,beanDefinition);
                    }
                }
            }
        }
        context.refresh();
        Arrays.stream(context.getBeanDefinitionNames()).forEach(System.out::println);

    }

}
```

