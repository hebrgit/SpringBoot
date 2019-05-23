



## 二、配置文件

### 1、配置文件注入值

​	1）、配置文件

```yml

person:
  age: ${random.int}
  dog:
    name: 小狗
    age: 2
  user-name: hebo
  email: 18829780487@613.com
  map: {k1: v1,k2: v2}
  list:
    - lisi
    - wangwu
```

​	2）、javaBean

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {


    private String userName;
    private int age;
    private String email;
    private Map map;
    private Dog dog;
    private List list;


```

​	3）、导入下面的jar包，配置文件就会自动提醒

```xm
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

#### 1、@Value获取值和@ConfigurationProperties获取值比较

|                | @ConfigurationProperties | @Value       |
| -------------- | ------------------------ | ------------ |
| 功能           | 批量注入配置文件的值     | 一个一个注入 |
| 松散绑定语法   | 支持                     | 不支持       |
| SpEL           | 不支持                   | 支持         |
| JSR303数据校验 | 支持                     | 不支持       |
| 复杂类型封装   | 支持                     | 不支持       |

#### 2、@PropertySource&@ImportSource

​	1）、@PropertySource 指定配置文件的路径，让配置文件生效

​	2）、@ImportSource：导入spring的配置文件，让配置文件里面的内容生效；Springboot里面没有Spring的配置文件，我们自己编写的配置文件也不会生效；想让Spring文件生效，加载进来，就在类上添加这个注解

```java
@ImprotSource(location = {"classpath:bean.xml"})
导入sprig的配置文件让其生效
```

spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.hebo.springbootmvcauto.sevice.HelloSerivce" id="helloSerivce">
        
    </bean>
</beans>
```





现在不来编写配置文件，SpringBoot推荐给容器中添加组价的方式，全注解方式

​	1）、配置类 === Spring配置文件

​	2）、使用@Bean给容器中添加组件

```java
@Configuration
public class MyAppConfig {
    
    @Bean
    public HelloSerivce helloSerivce(){
        return new HelloSerivce();
    }
}

```

#### 3、配置文件占位符

##### 	1）、随机数

```properties
person.dog.name=小狗${random.value}
person.age=${random.int}
person.user-name=${random.uuid}
person.map.k1=${random.long}
person.map.k2=${random.long(100)}
person.map.k3=${random.int(10)}
```



#####  	2）、占位符获取之前配置的值，如果没有就使用默认值

```properties
person.user-name=${random.uuid}hebo
person.email=${person.user-name:188}
```

#### 4、Profile

##### 	1）、多Profile文件

​	我们在主配置文件编写的时候，文件名可以是 application-{Profile}.properties/yml;

![](G:\springboot\images\微信截图_20190523140429.png)

##### 2）、yml支持多文档快模式

```yml
server:
  port: 8080
spring:
  profiles:
    active: dev
---
server:
  port: 8081
spring:
  profiles: dev
---
server:
  port: 8082
spring:
  profiles: prod
```

##### 3）、激活指定Profile

​	1、在配置文件中指定 spring.profile.active=dev

​	2、命令行方式 --spring.profile.active=dev

​	3、虚拟机参数 -Dspring.profile.active=dev

---

#### 5、配置文件的加载位置

​	springboot启动会扫描一下位置的application.properties&application.yml文件作为Spring boot默认配置文件

```properties
-file:./config/	
-file:./
-classpath:./config/
-classpath:./
优先级又高到底，高优先级覆盖第优先级
```

---

#### 6、外部配置文件加载顺序

---

#### 7、自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#common-application-properties)

自动配置原理：

> 1、springboot启动的时候加载主配置类，开启了自动配置功能==@EnableAutoConfiguration==
>
> 2、@EnableAutoConfiguration作用：利用AutoConfigurationImportSelector给容器中导入一些组件

> 3、以HttpEncodingAutoConfiguration为列解释自动配置原理：
>
> 根据当前的不同条件判断，决定这个配置类是否生效，一旦这个配置类生效，这个配置类就会给容器添加组件，这些组件对应的属性是从properties类中获取的，这些类里面的每一个属性有和配置文件绑定

```java
@Configuration//表示是一个自动配置类
@EnableConfigurationProperties({HttpProperties.class})//启用指定类的ConfigurationProperties功能
@ConditionalOnWebApplication(
    type = Type.SERVLET
)  //Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类的配置都生效； 判断当前应用是否是web应用
@ConditionalOnClass({CharacterEncodingFilter.class})//判断当前项目有没有这个类
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true
)//判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也成立；即使我们的配置文件中不配置spring.http.encoding.enabled=true，也默认生效
public class HttpEncodingAutoConfiguration {
    //给容器中添加一个组件，这些组件中的某些值从properties中获取  
    @Bean
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }

```

> 4、所有配置文件能配置的属性都封装在了xxxxProperties中，配置文件能配置什么就看这个类

```java
 ----------------------------------------------------------------------------------------    
@ConfigurationProperties(
    prefix = "spring.http"
)//从配置文件中获取指定的值和Bean的值进行绑定
public class HttpProperties {
```

**精髓**

>1、SpringBoot启动会加载大量的自动配置类
>
>2、我们看我们需要的内容有没有SpringBoot默认写好的自动配置类；
>
>3、我们再来看自动配置类中配置了那些组件，只要我们要用的组件有，我们就不用配置了
>
>4、给容器中自动配置类添加组价时，会从properties类中获取某些属性，我们就可以在配置文件中指定这些属性的 值

## 三、日志

### 	1、日志框架





### 2、slf4j使用

(1)、如何在系统中使用slf4j

## 四、web开发

### 1、简介

使用springBoot

1）、创建sprigBoot工程，选中我们需要的模块

2）、SpringBoot已经默认将这些场景配置好，只需要在配置文件中指定少量配置就可以运行起来

3）、自己编写业务代码

**自动配置原理？**

这个SpringBoot帮我们配置了什么？能修改什么？修改哪些配置？能不能扩展？xxxx

```
xxxxAutoConfiguration:帮我们给容器中自动配置组件
xxxxProperties:来封装配置文件的内容；
```

### 2、SpringBoot对静态资源的映射规则

```java
   public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }	
        }
		//配置欢迎页
       @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext 											applicationContext) {
            return new WelcomePageHandlerMapping(new 						       TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), 			this.mvcProperties.getStaticPathPattern());
        }


		//配置图标
 	    @Configuration
        @ConditionalOnProperty(
            value = {"spring.mvc.favicon.enabled"},
            matchIfMissing = true
        )
        public static class FaviconConfiguration implements ResourceLoaderAware {
            private final ResourceProperties resourceProperties;
            private ResourceLoader resourceLoader;

            public FaviconConfiguration(ResourceProperties resourceProperties) {
                this.resourceProperties = resourceProperties;
            }

            public void setResourceLoader(ResourceLoader resourceLoader) {
                this.resourceLoader = resourceLoader;
            }

            @Bean
            public SimpleUrlHandlerMapping faviconHandlerMapping() {
                SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
                mapping.setOrder(-2147483647);
                mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", this.faviconRequestHandler()));
                return mapping;
            }

            @Bean
            public ResourceHttpRequestHandler faviconRequestHandler() {
                ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
                requestHandler.setLocations(this.resolveFaviconLocations());
                return requestHandler;
            }

            private List<Resource> resolveFaviconLocations() {
                String[] staticLocations = WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter.getResourceLocations(this.resourceProperties.getStaticLocations());
                List<Resource> locations = new ArrayList(staticLocations.length + 1);
                Stream var10000 = Arrays.stream(staticLocations);
                ResourceLoader var10001 = this.resourceLoader;
                this.resourceLoader.getClass();
                var10000.map(var10001::getResource).forEach(locations::add);
                locations.add(new ClassPathResource("/"));
                return Collections.unmodifiableList(locations);
            }
        }
    }

```

1)、所有的/webjars/**，都去classpath:/META-INF/resources/webjars/找资源；

​		webjars：以jar包的方式引入静态资源； 

​		webjars路径：https://www.webjars.org

![](G:\springboot\images\微信截图_20190519180316.png)

访问资源路径列：localhost:1001/webjars/jquery/3.3.0/jquery.js

```xml
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.0</version>
        </dependency>
引入响应的jar包
```

2）、"/**" 访问当前路径下的任何资源（静态资源文件夹）

```
"classpath:/META-INF/resources/",
"classpath:/resources/", 
"classpath:/static/", 
"classpath:/public/"
"/*" 当前项目下的根路径
```

localhost:8080/abc === 去静态资源文件夹找abc

3）、欢迎页：静态资源文件下的所有index.html都被"/**"映射；

​	localhost:8080/  ==== 去找index.html

4）、所有的**/favicon.ico 都是在静态资源下找； 

### 3、模板引擎

​	1）、SpringBoot推荐的模板引擎Thymeleaf，语法简单功能强大

​	2）、引入thymeleaf

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
//如果要替换thymeleaf的版本，在pom.xml文件里加入如下代码
  <properties>
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.4.1</thymeleaf-layout-dialect.version>
  </properties>
```

​	3)Thymeleaf使用和&语法

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
```

​	只要把HTML页面放到classpath：/tempates/ 路径下，thymeleaf就会自动渲染

​	使用：

​		1、导入thymeleaf的名称空间 xmlns:th="http://www.thymeleaf.org"

​		2、使用thymeleaf的语法

```html
   <!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<div id="testid" class="testcalss" th:id="${Lion}" th:class="${Lion}" th:text="${Lion}">
    前端数据
</div>
<hr/>
text:<div th:text="${hello}"></div>
utext:<div th:utext="${hello}"></div>
<hr/>
<!--th:each每次遍历都会生成当前这个标签 3个h4-->
theach:
<h4 th:text="${user}" th:each="user:${users}"></h4>
<hr/>
<!--th:each每次遍历都会生成当前这个标签 h4下3个span-->
<h4>
    <span  th:each="user1:${users}"></span>
</h4>
```

​		3、语法规则

![语法规则](G:\springboot\images\微信截图_20190519222723.png)

```properties
Simple expressions:(表达式语法)
Variable Expressions: ${...}  获取变量值；本质就是OGNL
	1)、获取对象的属性、调用方法:
		# ${person.father.name} 
		# ${person.createCompleteName()}
		# ${person.createCompleteNameWithSeparator('-')}
	2)、使用内置对象
		#ctx : the context object.
        #vars: the context variables.
        #locale : the context locale.
        #request : (only in Web Contexts) the HttpServletRequest object.
        #response : (only in Web Contexts) the HttpServletResponse object.
        #session : (only in Web Contexts) the HttpSession object.
        #servletContext : (only in Web Contexts) the ServletContext object.
     3)、内置的工具对象
     	#execInfo : information about the template being processed.
        #messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
        #uris : methods for escaping parts of URLs/URIs Page 20 of 106
        #conversions : methods for executing the configured conversion service (if any).
        #dates : methods for java.util.Date objects: formatting, component extraction, etc.
        #calendars : analogous to #dates , but for java.util.Calendar objects.
        #numbers : methods for formatting numeric objects.
        #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
        #objects : methods for objects in general.
        #bools : methods for boolean evaluation.
        #arrays : methods for arrays.
        #lists : methods for lists.
        #sets : methods for sets.
        #maps : methods for maps.
        #aggregates : methods for creating aggregates on arrays or collections.
        #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
Selection Variable Expressions: *{...}选择表达式，和${}功能一样，补充配合${}使用
           #<div th:object="${session.user}">
               #<p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
           	   #<p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
               # <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
          #</div>
Message Expressions: #{...}获取国际化内容
Link URL Expressions: @{...}定义url
Fragment Expressions: ~{...}片段表达式
Literals ：（字面量）
        Text literals: 'one text' , 'Another one!' ,…
        Number literals: 0 , 34 , 3.0 , 12.3 ,…
        Boolean literals: true , false
        Null literal: null
        Literal tokens: one , sometext , main ,…
Text operations:（文本）
        String concatenation: +
        Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
        Binary operators: + , - , * , / , %
        Minus sign (unary operator): -
Boolean operations:(与或非运算)
        Binary operators: and , or
        Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
        Comparators: > , < , >= , <= ( gt , lt , ge , le )
        Equality operators: == , != ( eq , ne )
Conditional operators:（条件运算）
        If-then: (if) ? (then)
        If-then-else: (if) ? (then) : (else)
        Default: (value) ?: (defaultvalue)
Special tokens:（特殊操作）
        Page 17 of 106
        No-Operation: _
```

----------

### 4、Spring MVC 自动配置原理

29.1.1 Spring MVC Auto-configuration
Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of ==ContentNegotiatingViewResolver== and ==BeanNameViewResolver== beans.
  - 自动配置ViewResoler（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向）
  - ContentNegotiatingViewResolver：组合所有的视图解析器
  - 如何定制：我们可以给容器中添加一个解析器，会自动组合进去
- Support for serving static resources, including support for WebJars (covered later in this document)).
  - 静态资源
- Automatic registration of Converter, GenericConverter, and Formatter beans
  - Converter：转换器 ,类转换使用
  - Formatter：格式化器   2019-4-25==》Date
  - ==我们自己添加格式化转换器，我们只需在容器中添加即可==
- Support for HttpMessageConverters (covered later in this document).
  - HttpMessageConverter：SpringMVC用来转换Http请求和响应；User-->Json
  - HttpMessageConverters ：是从容器中确定获取所有的HttpMessageConverter;
  - ==给自己容器中添加HttpMessageConverter,只需将自己的组件注册到容器中==
- Automatic registration of MessageCodesResolver (covered later in this document).
  - 定义错误代码生成规则的
- Static index.html support.
  - 首页访问
- Custom Favicon support (covered later in this document).
  - 图标
- Automatic use of a ConfigurableWebBindingInitializer bean (covered later in this document).
  - 初始化web数据绑定器的（webDateBinder）；
  - 请求的数据绑定到javaBean中，请求数据---->JavaBean
  - 我们也可以配置一个ConfigurableWebBindingInitializer来替换默认的

If you want to keep Spring Boot MVC features and you want to add additional MVC configuration (interceptors, formatters, view controllers, and other features), you can add your own @Configuration class of type WebMvcConfigurer but without @EnableWebMvc. If you wish to provide custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter, or ExceptionHandlerExceptionResolver, you can declare a WebMvcRegistrationsAdapter instance to provide such components.

If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.

##### 	1）、扩展SpringMVc

​		编写一个配置类（@configuration），是WebMvcConfiguratonSupport类型的；不能标注@EnableWebMvc；既保留了springboot的默认配置，也能用我们扩展的配置

```java
@Configuration
public class MyWebMvc extends WebMvcConfigurationSupport {
    @Override
    protected void addViewControllers(ViewControllerRegistry registry) {
       registry.addViewController("/hello").setViewName("success");
    }

```

​	原理：

​		1、WebMvcAutoConfiguration是springmvc的自动配置类

​		2、在做其他配置时会导入：@Import({WebMvcAutoConfiguration.==EnableWebMvcConfiguration.class==})

```java
 @Configuration
    public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
        
    //从容器中获取所有的WebMvcConfigurer
    @Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
        }

    }
```



### 5、如何修改SpringBoot的默认配置

模式：

​	1）、SringBoot在配置很多组件的时候，先看容器有没有用户自己配置的（@Bean、@Component）如果有就用用户的，如果没有就是用默认的；如果有些组件可以有多个（ViewResovle）将用户配置的和自己默认的组合起来

​	2）、使用@Bean给容器中添加组件

```java
/**
 * @Configuration : 指明当前类就是一个配置类；就替代之前的Spring配置文件的
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloSerivce helloSerivce(){
        return new HelloSerivce();
    }
}
```

