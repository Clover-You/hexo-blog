title: Spring Boot核心功能 - 学习笔记
categories: Spring
tags: [JAVA,SpringBoot,Spring,学习笔记,SpringMvc]
date: 2022-01-02 18:06:00
---
## 配置文件

### 文件类型

#### properties

与之前使用`properties`文件的用法一致

#### yaml

##### 简介

YAML是`YAML Ain't markup Language` 的递归缩写。在开发这种语言时，YAML的意思是`Yet Another Markup Language`。

非常适合用来做以数据为中心的配置文件

##### 基本语法

- `key: value`kv之间有空格
- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用**tab**，只允许空格
- 缩进的空格数不重要，只要在相同层级的元素左对齐即可
- `#`表示注释
- 字符串无需加引号，如果需要，`''`与`""`表示字符串内容，会被转义/不转义



##### 数据类型

- 字面量：单个的、不可再分的值。date、boolean、string、number、null

  ```yaml
  k: v
  ```

  

- 对象：简直对的集合，map、hash、set、object

  - 行内写法

    ```yaml
    k: {k1: v1, k2: v2}
    ```

  - 普通写法

    ```yaml
    k: 
    	k1: v1
    	k2: v2
    ```

- 数组： 一组按次序排列的值，array、list、queue

  - 行内写法

    ```yaml
    k: [v1, v2]
    ```

  - 普通写法

    ```yaml
    k: 
    	- v1
    	- v2
    ```

    



##### 示例

使用一个`Person`组件绑定对应配置

```java
@Data
@Component
@ConfigurationProperties(prefix = "person")
public class Person implements Serializable {

    private static final long serialVersionUID = 7514130876563463491L;

    public Person() { }

    private String name;

    private String[] interest;

    private int age;

    private Map map;
}
```

我习惯了JSON的写法，发现它也支持，那就这样用叭～

```yaml
{
  person: {
    name: 2,
    age: 18,
    interest: [ 女 ],
    map: {
      map1: value
    }
  }
}
```

检查`Person`容器是否正常

```java
public static void main(String[] args) {
        ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
        Person bean = app.getBean(Person.class);
        System.out.println(bean);

        System.out.println(" _   _     __   __          \n" +
                "| | | |_ __\\ \\ / /__  _   _ \n" +
                "| | | | '_ \\\\ V / _ \\| | | |\n" +
                "| |_| | |_) || | (_) | |_| |\n" +
                " \\___/| .__(_)_|\\___/ \\__,_|\n" +
                "      |_|                   ");
    }
```

输出：`Person(name=2, interest=[女], age=18, map={map1=value})`



### 自定义类绑定的配置提示

有没有发现我们写的配置都没得提示？而且绑定类顶部还有一个提示框消不掉。

需要添加`spring-boot-configuration-processor`依赖，添加完依赖之后就可以了。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

这个插件其实只是在我们开发的时候有帮助，而在生产环境是无用的，还会让JVM启动时加载无用的依赖导致耗时过长。可以在`pom.xml`文件中将开发环境下才有效的依赖配置为不参与打包。

```xml
<build>
  <plugins>
    <plugin>
      <configuration>
        <excludes>
          <exclude>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
          </exclude>
          ...
        </excludes>
      </configuration>
    </plugin>
  </plugins>
</build>
```



## Web开发

### 简单功能分析



#### 静态资源访问



##### 静态资源目录

SpringBoot的静态资源是在当前项目的类路径下 `/static` 、 `/public` 、 `/resources` 、 `/META-INF/resources`，如果资源在规定的目录下，SpringBoot可以直接访问到这些目录中的资源文件。

例如我在`/resources/static`文件夹下有一张图片，使用浏览器请求看看能不能直接访问这张图片

![项目结构](http://qiniu-note-image.ctong.top/note/images/202112271049118.png)



没毛病



![测试结果](http://qiniu-note-image.ctong.top/note/images/202112271052434.png)



##### 静态资源访问原理

例如我们有一个请求与静态资源文件的路径一致，那么请求的时候是将静态资源文件返回还是处理对应请求呢：

```java
@GetMapping("/back.jpeg")
public String testResource() {
  return "Hello";
}
```

当浏览器访问`/back.jpeg`的时候，响应了'Hello'字符串。

静态资源映射路径是`/**`，而请求进来的时候，是先去找`Controller`看能不能处理这个请求，如果不能则交给静态资源处理器。如果都找不到，则返回**404**



##### 静态资源访问前缀

静态资源访问默认是无前缀的，如果通过拦截器，例如登录拦截器，可能会拦截到静态资源文件，所以我们需要给静态资源指定一个前缀，这样拦截器就可以很方便的将静态资源过滤。

只需要在配置文件中配置`spring.mvc.static-path-pattern`就可以，默认是`/**`

```yaml
spring:
  mvc:
    static-path-pattern: /static/**
```



##### 静态资源路径修改

SpringBoot默认的静态资源路径是在类路径下，如果不需要使用默认的设置，也可以通过配置文件进行修改，它可以是一个数组，因为静态资源路径他是多个的。 

```yaml
spring:
  resources:
    static-locations: classpath:/ctong/
```



##### 欢迎页配置



- 将`index.html`文件放在静态资源路径下
  - 如果配置了静态资源路径前缀，会导致欢迎页的配置无法使用
  - 可以配置静态资源路径
- 在`controller`中处理`/index`请求，他会被当成静态页显示出来。





##### Favicon

如果需要更换Favicon，则需要将对应的文件放到静态资源目录下，名字必须是`favicon.ico`，SpringBoot会默认将它设置为网页的icon。

> 如果设置了静态资源路径前缀，那么favicon将不会生效。





##### 静态资源配置原理

- SpringBoot启动默认加载`xxxAutoConfiguration`「自动配置类」

- SpringMVC功能配置类`WebMvcAutoConfiguration`，检查该配置类是否生效

  ```java
  @Configuration(proxyBeanMethods = false)
  @ConditionalOnWebApplication(type = Type.SERVLET)
  @ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
  @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
  @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
  @AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
  		ValidationAutoConfiguration.class })
  public class WebMvcAutoConfiguration {...}
  ```

- 给容器配了什么

  ```java
  	@Configuration(proxyBeanMethods = false)
  	@Import(EnableWebMvcConfiguration.class)
  	@EnableConfigurationProperties({ WebMvcProperties.class,
  			org.springframework.boot.autoconfigure.web.ResourceProperties.class, WebProperties.class })
  	@Order(0)
  	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {...}
  ```

- 配置文件的相关属性和`WebMvcProperties`、`ResourceProperties`、`WebProperties`，进行了绑定



###### **资源处理默认规则核心代码**



```java
@Override
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
  super.addResourceHandlers(registry);
  if (!this.resourceProperties.isAddMappings()) {
    logger.debug("Default resource handling disabled");
    return;
  }
  ServletContext servletContext = getServletContext();
  addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
  addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
    registration.addResourceLocations(this.resourceProperties.getStaticLocations());
    if (servletContext != null) {
      registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
    }
  });
}
```



如果以下代码为`false`，那么禁用所有默认规则，一旦禁用，所有静态资源无法访问，它对应了`spring.resources.add-mappings`配置 

```java
!this.resourceProperties.isAddMappings()
```

以下代码是注册**wenjars**路径的默认规则，只要前缀为`/webjars/**`那么都去指定目录中找`classpath:/META-INF/resources/webjars/`

```java
addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
```



这才是我们资源路径的默认规则

```java
addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
  registration.addResourceLocations(this.resourceProperties.getStaticLocations());
  if (servletContext != null) {
    registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
  }
});
```

其实就是获取到配置文件中我们自定义的资源路径前缀`/static/**`

```java
this.mvcProperties.getStaticPathPattern()
```

这段代码是获取到配置文件中的资源文件路径`classpath:/ctong/`

```java
this.resourceProperties.getStaticLocations()
```

其实就是这样

```java
addResourceHandler(registry, "/static/**", "classpath:/resources/ctong/");
```





###### **欢迎页的处理规则**



```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                                                           FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
  WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
    new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
    this.mvcProperties.getStaticPathPattern());
  welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
  welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
  return welcomePageHandlerMapping;
}
```

`welcomePage != null && "/**".equals(staticPathPattern)`解释了我们修改资源路径前缀时为何欢迎页失效的问题。

```java
WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
                          ApplicationContext applicationContext, Resource welcomePage, String staticPathPattern) {
  if (welcomePage != null && "/**".equals(staticPathPattern)) {
    logger.info("Adding welcome page: " + welcomePage);
    setRootViewName("forward:index.html");
  }
  else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
    logger.info("Adding welcome page template: index");
    setRootViewName("index");
  }
}
```

> - 如果我们看一个配置文件，那么一定要看它是否生效
>
> - 配置类只有一个有参构造器，有参构造器中所有参数的值都会从容器中确定。



## 请求参数处理

### 请求映射

- `@xxxMapping`，经常使用`@RequestMapping`
- `Rest`风格支持「使用HTTP请求方式动词来表示对资源的操作」
  - 以前`/getUser`获取用户`/deleteUser`删除用户，`/saveUser`保存用户
  - 现在`/user`GET-获取用户 DELETE-删除用户 PUT-修改用户 POST-保存用户

如果需要使用`Rest`风格的请求，那么必须发起`POST`,并且携带一个隐藏参数`_method`，这个参数值为`PUT`、`DELETE`。

当你测试发起`Rest`风格请求时，发现并没有效果。查看检查映射处理源码

```java
@Bean
@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
  return new OrderedHiddenHttpMethodFilter();
}
```



`@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)`很显然我们并没有配置，所以这个是生效的。

`@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)`，`enabled`翻译为启用的意思，`matchIfMissing`为`false`，所以我们请求没生效的原因是`Rest`请求被禁用了。

#### Rest原理

直接上源码

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
  throws ServletException, IOException {

  HttpServletRequest requestToUse = request;

  if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {
    String paramValue = request.getParameter(this.methodParam);
    if (StringUtils.hasLength(paramValue)) {
      String method = paramValue.toUpperCase(Locale.ENGLISH);
      if (ALLOWED_METHODS.contains(method)) {
        requestToUse = new HttpMethodRequestWrapper(request, method);
      }
    }
  }

  filterChain.doFilter(requestToUse, response);
}
```

检查当前请求参数是否为`POST`，并且当前请求是否存在错误

```java
if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {...}
```

获取指定参数，`this.methodParam`其实就是`_method`

```java
String paramValue = request.getParameter(this.methodParam);
```

检查该参数是否不为空

```
if (StringUtils.hasLength(paramValue)) {...}
```

无论是小写还是大些，全部转为大些

```java
String method = paramValue.toUpperCase(Locale.ENGLISH);
```

检查是否兼容当前请求方式

```java
if (ALLOWED_METHODS.contains(method)) {...}
```

```java
private static final List<String> ALLOWED_METHODS =
      Collections.unmodifiableList(Arrays.asList(HttpMethod.PUT.name(),
            HttpMethod.DELETE.name(), HttpMethod.PATCH.name()));
```

这里重新包装了一下`HttpServletRequest`

```java
requestToUse = new HttpMethodRequestWrapper(request, method);
```

因为它是以`POST`的方式请求进来的，所以`HttpServletReqest`提供的`getMethod`方法获取的是一个`POST`而不是对应的`PUT`、`DELETE`，所以需要重新将`method`改成对应的请求。

```java
private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {

  private final String method;

  public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
    super(request);
    this.method = method;
  }

  @Override
  public String getMethod() {
    return this.method;
  }
}
```

最后放行了过滤器链`wrapper`，以后我们开发使用`Rest`风格请求时，`HttpServletRequest`其实是`SpringBoot`给我们包装的。

`filterChain.doFilter(requestToUse, response);`





#### 请求映射原理

SpringBoot处理请求是在`DispatcherServlet` `FrameworkServlet` `HttpServlet`

在`processRequest`方法中处理请求

```java
/**
	 * Process this request, publishing an event regardless of the outcome.
	 * <p>The actual event handling is performed by the abstract
	 * {@link #doService} template method.
	 */
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {

  long startTime = System.currentTimeMillis();
  Throwable failureCause = null;

  LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
  LocaleContext localeContext = buildLocaleContext(request);

  RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
  ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
  asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

  initContextHolders(request, localeContext, requestAttributes);

  try {
    doService(request, response);
  }
  catch (ServletException | IOException ex) {
    failureCause = ex;
    throw ex;
  }
  catch (Throwable ex) {
    failureCause = ex;
    throw new NestedServletException("Request processing failed", ex);
  }

  finally {
    resetContextHolders(request, previousLocaleContext, previousAttributes);
    if (requestAttributes != null) {
      requestAttributes.requestCompleted();
    }
    logResult(request, response, failureCause, asyncManager);
    publishRequestHandledEvent(request, response, startTime, failureCause);
  }
}
```

在`try`中又调用了`doService`方法

```java
/**
	 * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
	 * for the actual dispatching.
	 */
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
  logRequest(request);

  // Keep a snapshot of the request attributes in case of an include,
  // to be able to restore the original attributes after the include.
  Map<String, Object> attributesSnapshot = null;
  if (WebUtils.isIncludeRequest(request)) {
    attributesSnapshot = new HashMap<>();
    Enumeration<?> attrNames = request.getAttributeNames();
    while (attrNames.hasMoreElements()) {
      String attrName = (String) attrNames.nextElement();
      if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
        attributesSnapshot.put(attrName, request.getAttribute(attrName));
      }
    }
  }

  // Make framework objects available to handlers and view objects.
  request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
  request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
  request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
  request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

  if (this.flashMapManager != null) {
    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
      request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
  }

  RequestPath previousRequestPath = null;
  if (this.parseRequestPath) {
    previousRequestPath = (RequestPath) request.getAttribute(ServletRequestPathUtils.PATH_ATTRIBUTE);
    ServletRequestPathUtils.parseAndCache(request);
  }

  try {
    doDispatch(request, response);
  }
  finally {
    if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
      // Restore the original attribute snapshot, in case of an include.
      if (attributesSnapshot != null) {
        restoreAttributesAfterInclude(request, attributesSnapshot);
      }
    }
    ServletRequestPathUtils.setParsedRequestPath(previousRequestPath, request);
  }
}

```

在这个方法中又有一个`doDispatch`方法，这个才是最核心的

```java

/**
	 * Process the actual dispatching to the handler.
	 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
	 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
	 * to find the first that supports the handler class.
	 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
	 * themselves to decide which methods are acceptable.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @throws Exception in case of any kind of processing failure
	 */
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // Determine handler for the current request.
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }

      // Determine handler adapter for the current request.
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

      // Process last-modified header, if supported by the handler.
      String method = request.getMethod();
      boolean isGet = "GET".equals(method);
      if (isGet || "HEAD".equals(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
          return;
        }
      }

      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
      }

      // Actually invoke the handler.
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

      if (asyncManager.isConcurrentHandlingStarted()) {
        return;
      }

      applyDefaultViewName(processedRequest, mv);
      mappedHandler.applyPostHandle(processedRequest, response, mv);
    }
    catch (Exception ex) {
      dispatchException = ex;
    }
    catch (Throwable err) {
      // As of 4.3, we're processing Errors thrown from handler methods as well,
      // making them available for @ExceptionHandler methods and other scenarios.
      dispatchException = new NestedServletException("Handler dispatch failed", err);
    }
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  }
  catch (Exception ex) {
    triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
  }
  catch (Throwable err) {
    triggerAfterCompletion(processedRequest, response, mappedHandler,
                           new NestedServletException("Handler processing failed", err));
  }
  finally {
    if (asyncManager.isConcurrentHandlingStarted()) {
      // Instead of postHandle and afterCompletion
      if (mappedHandler != null) {
        mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
      }
    }
    else {
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
        cleanupMultipart(processedRequest);
      }
    }
  }
}
```

> org.springframework.web.servlet.DispatcherServlet -> doDispatch()

查找哪个`Controller`能处理当前请求，`com.upyou.hall.controller.HelloController#saveUser()`

```java
mappedHandler = getHandler(processedRequest);
```

进到这个方法中

```java
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
  if (this.handlerMappings != null) {
    for (HandlerMapping mapping : this.handlerMappings) {
      HandlerExecutionChain handler = mapping.getHandler(request);
      if (handler != null) {
        return handler;
      }
    }
  }
  return null;
}
```

发现有一个`this.handlerMappings`，这个属性保存的是请求的映射规则。其中`RequestMappingHandlerMapping` 就保存了所有`@RequestMapping`和`handler`的映射规则。![handlerMapping](http://qiniu-note-image.ctong.top/note/images/202112271049137.png)

![所有请求路径](http://qiniu-note-image.ctong.top/note/images/202112271049148.png)

进入`mapping.getHandler(request);` ->> `getHandlerInternal(request);` ->> `super.getHandlerInternal(request);`

```java

/**
	 * Look up a handler method for the given request.
	 */
@Override
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
  String lookupPath = initLookupPath(request);
  this.mappingRegistry.acquireReadLock();
  try {
    HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
    return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
  }
  finally {
    this.mappingRegistry.releaseReadLock();
  }
}
```

拿到当前请求的路径`/user`

```java
String lookupPath = initLookupPath(request);
```

防止并发请求查询`mappingRegistry`

```java
this.mappingRegistry.acquireReadLock();
```

进入`HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);`

```java
@Nullable
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
  List<Match> matches = new ArrayList<>();
  List<T> directPathMatches = this.mappingRegistry.getMappingsByDirectPath(lookupPath);
  if (directPathMatches != null) {
    addMatchingMappings(directPathMatches, matches, request);
  }
  if (matches.isEmpty()) {
    addMatchingMappings(this.mappingRegistry.getRegistrations().keySet(), matches, request);
  }
  if (!matches.isEmpty()) {
    Match bestMatch = matches.get(0);
    if (matches.size() > 1) {
      Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
      matches.sort(comparator);
      bestMatch = matches.get(0);
      if (logger.isTraceEnabled()) {
        logger.trace(matches.size() + " matching mappings: " + matches);
      }
      if (CorsUtils.isPreFlightRequest(request)) {
        for (Match match : matches) {
          if (match.hasCorsConfig()) {
            return PREFLIGHT_AMBIGUOUS_MATCH;
          }
        }
      }
      else {
        Match secondBestMatch = matches.get(1);
        if (comparator.compare(bestMatch, secondBestMatch) == 0) {
          Method m1 = bestMatch.getHandlerMethod().getMethod();
          Method m2 = secondBestMatch.getHandlerMethod().getMethod();
          String uri = request.getRequestURI();
          throw new IllegalStateException(
            "Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
        }
      }
    }
    request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.getHandlerMethod());
    handleMatch(bestMatch.mapping, lookupPath, request);
    return bestMatch.getHandlerMethod();
  }
  else {
    return handleNoMatch(this.mappingRegistry.getRegistrations().keySet(), lookupPath, request);
  }
}
```

通过`lookupPath`在`mappingRegistry`中找能够处理这个路径的`handler`

```java
List<T> directPathMatches = this.mappingRegistry.getMappingsByDirectPath(lookupPath);
```

发现它找到了两个，同路径但不同的请求类型

![mappingRegistry查询结果](http://qiniu-note-image.ctong.top/note/images/202112271049142.png)

其实就是通过请求方式和路径参数在`RequestMapping`中找到对应的`Mapping`添加到`matches`中

```java
addMatchingMappings(directPathMatches, matches, request);
```

如果无法在`RequestMapping`中找到，再通过请求方式和路径参数找所有HandlerMapping，如果找到则将其添加到`matches`中，这所有包括了资源路径。

```java
addMatchingMappings(this.mappingRegistry.getRegistrations().keySet(), matches, request);
```

拿到第一个`Mapping`,因为这极有可能是最适合处理当前请求的`Mapping`

```java
Match bestMatch = matches.get(0);
```

当匹配到多个结果之后，检查是不是同个请求方式、路径注册了两个处理器，如果是则抛出异常

```java
if (matches.size() > 1) {...}
```



- SpringBoot默认配置了`RequestMappingHandlerMapping`





### 普通参数与基本注解



#### 注解
  `@PathVariable` `@RequestHeader` `@ModelAttribute` `@RequestParam` `@MatrixVariable` `@CookieValue` `@RequestBody`

#### @PathVariable
获取动态径上的数据
  `@RequestParam("id")、@RequestParam("token")`分别是获取 `{id}` `{token}`位置的值，也就是说他们是动态的，

  ```java
  @RequestMapping("/index/{id}/get/{token}/info")
  public Map<String, String> handleHallo(@PathVariable("id") String id,
                                         @PathVariable("token") String token) {
    Map<String, String> map = new HashMap<>(2);
    map.put("map", id);
    map.put("token", token);
    return map;
  }
  ```

  请求`localhost:8888/index/1024/get/2048/info`

  结果

  ```json
  {
      "map": "1024",
      "token": "2048"
  }
  ```

  

#### @RequestHeader

获取指定请求头

  ```java
  @RequestMapping("/index")
  public String handleHallo(@RequestHeader("User-Agent") String userAgent) {
    return userAgent;
  }
  ```

  输出`PostmanRuntime/7.26.10`

  如果被标注的类型为`java.util.Map、org.springframework.util.MultiValueMap、org.springframework.http.HttpHeaders`那么SpringBoot可以给你拿到所有的请求头

#### @RequestParam 
获取请求参数

  ```java
  @RequestMapping("/index")
  public String handleHallo(@RequestParam("userName") String userName) {
    return userName;
  }
  ```

  请求`localhost:8888/index?userName=UpYou`

  输出：**UpYou**

  如果`@RequestParam`没有指定获取那个参数，并且被标注的类型为`Map<String, String>`，该变量将拿到所有的请求参数

#### @CookieValue

获取`cookie`
  ```java
      @RequestMapping("/index")
      public String handleHallo(@CookieValue("token") String token) {
          return token;
      }
  ```

  如果被标注的类型是`Cookie`，那么它将获取指定cookie的所有信息

  ```java
  @CookieValue("token") Cookie token
  ```

#### @RequestBody
获取球体「POST」

  ```java
  @PostMapping("/index")
  public String handleHallo(@RequestBody String content) {
      return content;
  }
  ```

  ```json
  {
      "name": "UpYou",
      "age": 18
  }
  ```

#### @RequestAttribute

获取到当前请求的域对象中的指定属性

  做一个请求，在这个请求中设置域对象的属性，然后转发到`/success`请求中处理

  ```java
  
  @Controller
  public class HelloController implements Serializable {
  
    private static final long serialVersionUID = -8638687587976398521L;
  
  
    public HelloController() { }
  
    @GetMapping("/index")
    public String handleHallo(HttpServletRequest request) {
      request.setAttribute("name", "UpYou");
      return "forward:/success";
    }
  
    @ResponseBody
    @GetMapping("/success")
    public String toDo(@RequestAttribute("name") String name) {
      return name;
    }
  
  }
  
  ```

  输出结果`UpYou`

#### @MatrixVariable
矩阵变量，它绑定在路径变量中。
  `/xxx/{path}?xxx=xxx&xxx=xxx`这个叫做查询字符串，可以使用`@RequestParam`获取

  `/xxx/{path;low=34;brand=byd,audi,yd}`这个是矩阵变量

  > 例如前端cookie被禁用了，session里面的内容怎么使用 ，这时候可以使用矩阵变量进行传递：`/xxx;jsessionid=xxx`

  ```java
  @GetMapping("/get/sell")
  public Map<String, Object> index(@MatrixVariable("age") Integer age,
                                   @MatrixVariable("name") List<String> name) {
      Map<String, Object> map = new HashMap<>(2);
      map.put("age", age);
      map.put("brand", name);
      return map;
  }
  ```

  请求`/get/sell;age=18;name=UpYou,ctong`

  报错了，原因是SpringBoot默认禁用了矩阵变量，需要手动开启。

  这种情况下需要定制化开启，SpringBoot提供了两种方式

  **第一种**
  声明`WebMvcConfigurer`改变默认底层组件

  ```java
  @Configuration
  public class WebConfig
    implements Serializable, WebMvcConfigurer {
  
    private static final long serialVersionUID = -3281050128601582429L;
  
    public WebConfig() { }
  
    /**
       * 开启矩阵变量
       * @param configurer 路径匹配规则配置
       */
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
      UrlPathHelper urlPathHelper = new UrlPathHelper();
      // 设置不移除分号后边的内容
      urlPathHelper.setRemoveSemicolonContent(false);
      configurer.setUrlPathHelper(urlPathHelper);
    }
  
  }
  ```

  **第二种**

  组册`WebMvcConfigurer`类型的组件

  ```java
  @Bean
  public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {
      @Override
      public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
      }
    };
  }
  ```

---

  开启矩阵变量之后，重新请求发现变成了404，原因是`@GetMapping("/get/sell")`**不能直接写sell**，需要写成路径变量的形式。

  ```java
  @GetMapping("/get/{path}")
  public Map<String, Object> matrix(@MatrixVariable("age") Integer age,
                                   @MatrixVariable("name") List<String> name,
                                   @PathVariable("path") String path) {
      Map<String, Object> map = new HashMap<>(3);
      map.put("age", age);
      map.put("brand", name);
      map.put("path", path);
      return map;
  }
  ```

  输出`{"brand":["UpYou","ctong"],"path":"sell","age":18}`

  如果你遇到了这种路径：`/get/1;age=18/2;age=18`一个路径中有多个相同矩阵变量。这时候需要使用`@MatrixVariable`提供的`pathVar`参数指定路径

  ```java
  @GetMapping("/get/{path}/{boss}")
  public Map<String, Object> matrixTwo(@PathVariable("path") String path,
                                       @PathVariable("boss") String boss,
                                       @MatrixVariable(value = "age", pathVar = "path") Integer pathAge,
                                       @MatrixVariable(value = "age", pathVar = "boss") Integer bossAge) {
      Map<String, Object> map = new HashMap<>(4);
      map.put("pathAge", pathAge);
      map.put("bossAge", bossAge);
      map.put("path", path);
      map.put("boss", boss);
      return map;
  }
  ```

  输出： `{"pathAge":18,"path":"path","boss":"boss","bossAge":48}`



#### 各种参数类型解析原理

> 如果需要查看路径请求处理原理，可以到` org.springframework.web.servlet.DispatcherServlet`下查看，这是处理请求的入口

走到`DispatcherServlet`==> `doDispatch` 

找到能处理该请求的方法

```java
mappedHandler = getHandler(processedRequest);
```

由于SpringBoot底层需要使用反射机制去调用这个方法，而方法中有各种各样的注解、请求参数，例如`PathVariable`，`@RequestParam`等，由于该步骤非常麻烦，所以需要为当前Handler找一个适配器`HandlerAdapter`

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

在getHandlerAdapter方法中，SpringBoot默认为我们提供了四种适配器。分别用来完成不同的功能。

1. 支持`@RequestMapping`
2. 支持函数式编程

![适配器](http://qiniu-note-image.ctong.top/note/images/202112271049154.png)

匹配支持的`Handler`

```java
for (HandlerAdapter adapter : this.handlerAdapters) {
   if (adapter.supports(handler)) {
      return adapter;
   }
}
```

最后通过适配器去处理这个方法

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

#### 参数处理器

进入`ha.handle`，这一段，执行目标方法

```java
mav = invokeHandlerMethod(request, response, handlerMethod);
```

再进入`invokeHandlerMethod` 

这是一个参数解析器，将来目标参数值是什么，全都是由这些参数解析器来决定，例如参数标注了`@RequestParam`，那么就使用图中第一个解析器

```java
if (this.argumentResolvers != null){...}
```

SpringBootMVC目标方法能写多少种参数类型，全部取决于参数解析器。

![参数解析器](http://qiniu-note-image.ctong.top/note/images/202112271049158.png)

`supportsParameter`检查是否支持解析这种参数，如果支持就调用`resolveArgument`进行解析处理

```java
public interface HandlerMethodArgumentResolver {

   boolean supportsParameter(MethodParameter parameter);

   @Nullable
   Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
         NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```

#### 返回值处理器

这是得到返回值支持的处理器，也就是说，SpringBoot会工具反射去调用对应的方法，这会得到一个返回值，那么会通过这个返回值进行处理，处理完之后返回给客户端。

```java
if (this.returnValueHandlers != null) {
  invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
}
```



#### 执行处理器

在这开始执行对应的方法，可以在我们目标方法与`invokeAndHandle`方法中打一个断点，当`invokeAndHandle`执行放行之后开始执行目标方法

```java
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

进到`invokeAndHandle`，在这才是真正的执行，

```java
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
```

进`invokeForRequest`方法中

```java
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
                               Object... providedArgs) throws Exception {

  Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
  if (logger.isTraceEnabled()) {
    logger.trace("Arguments: " + Arrays.toString(args));
  }
  return doInvoke(args);
}
```

获取方法所有参数的值

```java
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
```

![方法所有参数值](http://qiniu-note-image.ctong.top/note/images/202112271049358.png)



##### 如何确定目标方法每一个参数值

`org.springframework.web.method.support.InvocableHandlerMethod`

```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
                                           Object... providedArgs) throws Exception {

  MethodParameter[] parameters = getMethodParameters();
  if (ObjectUtils.isEmpty(parameters)) {
    return EMPTY_ARGS;
  }

  Object[] args = new Object[parameters.length];
  for (int i = 0; i < parameters.length; i++) {
    MethodParameter parameter = parameters[i];
    parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
    args[i] = findProvidedArgument(parameter, providedArgs);
    if (args[i] != null) {
      continue;
    }
    if (!this.resolvers.supportsParameter(parameter)) {
      throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
    }
    try {
      args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
    }
    catch (Exception ex) {
      // Leave stack trace for later, exception may actually be resolved and handled...
      if (logger.isDebugEnabled()) {
        String exMsg = ex.getMessage();
        if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
          logger.debug(formatArgumentError(parameter, exMsg));
        }
      }
      throw ex;
    }
  }
  return args;
}
```

获取到所有参数的详细信息

```java
MethodParameter[] parameters = getMethodParameters();
```

查找是否有解析器支持解析当前这种类型

```java
if (!this.resolvers.supportsParameter(parameter)) {
  throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
}
```

进入`supportsParameter` ==> `getArgumentResolver`

判断方式是通过`for`循环来进行逐个对比得到结果

```java
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
   HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
   if (result == null) {
      for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
         if (resolver.supportsParameter(parameter)) {
            result = resolver;
            this.argumentResolverCache.put(parameter, result);
            break;
         }
      }
   }
   return result;
}
```



##### 解析这个参数的值

通过得到的解析器来解析这个参数

```java
args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
```

进入`resolveArgument`

```java
@Override
@Nullable
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

  HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
  if (resolver == null) {
    throw new IllegalArgumentException("Unsupported parameter type [" +
                                       parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
  }
  return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

获取参数解析器

```java
HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
```

使用了`resolver.resolveArgument`进行解析

进入`resolveArgument`

拿到当前参数的基本信息

```java
NamedValueInfo namedValueInfo = getNamedValueInfo(parameter);
MethodParameter nestedParameter = parameter.nestedIfOptional();
```

通过对应的解析器获取对应值

```java
Object arg = resolveName(resolvedName.toString(), nestedParameter, webRequest);
```

进入`resolveName`

> 注意，这是一个`PathVariableMethodArgumentResolver`解析器，其它解析器基本也是这种操作

由于之前已经将请求参数放进请求域中，所以这里直接从请求域中拿了。

```java
protected Object resolveName(String name, MethodParameter parameter, NativeWebRequest request) throws Exception {
  Map<String, String> uriTemplateVars = (Map<String, String>) request.getAttribute(
    HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE, RequestAttributes.SCOPE_REQUEST);
  return (uriTemplateVars != null ? uriTemplateVars.get(name) : null);
}
```

最后将设置好的值全部返回，执行`Handler`



### Servlet API

WebRequest、ServletRequest、MultipartRequest、HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、Zoneld

> `ServletRequestMethodArgumentResolve`解析器支持解析以上大部分参数



### 复杂参数

Map、Model、Errors/BindingResult、RedirectAttributes、ServletResponse、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

- 如果给`Map、Model`放数据，会被默认存放在`request`请求域中。

  ```java
  request.setAttribute();
  ```
  
  ```java
  @GetMapping("/params")
  public String params(Map<String, Object> map,
                       Model model,
                       HttpServletRequest request,
                       HttpServletResponse response) {
    map.put("name", "UpYou");
    model.addAttribute("age", 18);
    return "forward:/success";
  }
  
  @ResponseBody
  @GetMapping("/success")
  public Map<String, Object> success(HttpServletRequest request) {
    Object name = request.getAttribute("name");
    Object age = request.getAttribute("age");
    HashMap<String, Object> stringObjectHashMap = new HashMap<>();
    stringObjectHashMap.put("name", name);
    stringObjectHashMap.put("age", age);
    return stringObjectHashMap;
  }
  ```
  
  输出`{"name":"UpYou","age":18}`
  
  > Map类型的参数使用MapMethodProcessor解析器解析。这将会返回mavContainer.getModel() ==>> BindingAwareModelMap，它既是Model也是Map
  >
  > Model类型与Map类型的处理都是一致的，最终调用的都是mavContainer.getModel()
  
- `RedirectAttributes`重定向携带的数据





### 自定义类型参数绑定原理


```java
/**
 * 数据绑定：页面提交的请求数据（GET、POST、PUT、DELETE）都可以和对象属性进行绑定
 * @param person 请求数据
 * @return 请求数据
 */
@PutMapping("/user")
public Person saveUser(Person person) {
  return person;
}

```


```java
@Data
public class Person implements Serializable {

  private static final long serialVersionUID = 7514130876563463491L;

  public Person() { }

  private String name;

  private String[] interest;

  private int age;

  private Pet pet;

}

@Data
public class Pet implements Serializable {

  private static final long serialVersionUID = 2248757905577331865L;

  private String name;

  public Pet() { }

  public Pet(String name) {
    this.name = name;
  }

}
```



![请求结果](http://qiniu-note-image.ctong.top/note/images/202112271049496.png)



#### POJO 封装过程

自定义类型参数是通过`ServletModelAttributeMethodProcessor`参数解析器来进行解析的。

判断方式：判断当前类型是否为简单类型，如果不是，则支持

```java
@Override
public boolean supportsParameter(MethodParameter parameter) {
  return (parameter.hasParameterAnnotation(ModelAttribute.class) ||
          (this.annotationNotRequired && !BeanUtils.isSimpleProperty(parameter.getParameterType())));
}

public static boolean isSimpleProperty(Class<?> type) {
  Assert.notNull(type, "'type' must not be null");
  return isSimpleValueType(type) || type.isArray() && isSimpleValueType(type.getComponentType());
}

public static boolean isSimpleValueType(Class<?> type) {
  return Void.class != type && Void.TYPE != type && (ClassUtils.isPrimitiveOrWrapper(type) || Enum.class.isAssignableFrom(type) || CharSequence.class.isAssignableFrom(type) || Number.class.isAssignableFrom(type) || Date.class.isAssignableFrom(type) || Temporal.class.isAssignableFrom(type) || URI.class == type || URL.class == type || Locale.class == type || Class.class == type);
}
```

解析器调用`resolveArgument`方法对这个参数进行解析

```java
public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                    NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

  Assert.state(mavContainer != null, "ModelAttributeMethodProcessor requires ModelAndViewContainer");
  Assert.state(binderFactory != null, "ModelAttributeMethodProcessor requires WebDataBinderFactory");

  String name = ModelFactory.getNameForParameter(parameter);
  ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
  if (ann != null) {
    mavContainer.setBinding(name, ann.binding());
  }

  Object attribute = null;
  BindingResult bindingResult = null;

  if (mavContainer.containsAttribute(name)) {
    attribute = mavContainer.getModel().get(name);
  }
  else {
    // Create attribute instance
    try {
      attribute = createAttribute(name, parameter, binderFactory, webRequest);
    }
    catch (BindException ex) {
      if (isBindExceptionRequired(parameter)) {
        // No BindingResult parameter -> fail with BindException
        throw ex;
      }
      // Otherwise, expose null/empty value and associated BindingResult
      if (parameter.getParameterType() == Optional.class) {
        attribute = Optional.empty();
      }
      else {
        attribute = ex.getTarget();
      }
      bindingResult = ex.getBindingResult();
    }
  }

  if (bindingResult == null) {
    // Bean property binding and validation;
    // skipped in case of binding failure on construction.
    WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
    if (binder.getTarget() != null) {
      if (!mavContainer.isBindingDisabled(name)) {
        bindRequestParameters(binder, webRequest);
      }
      validateIfApplicable(binder, parameter);
      if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
        throw new BindException(binder.getBindingResult());
      }
    }
    // Value type adaptation, also covering java.util.Optional
    if (!parameter.getParameterType().isInstance(attribute)) {
      attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
    }
    bindingResult = binder.getBindingResult();
  }

  // Add resolved attribute and BindingResult at the end of the model
  Map<String, Object> bindingResultModel = bindingResult.getModel();
  mavContainer.removeAttributes(bindingResultModel);
  mavContainer.addAllAttributes(bindingResultModel);

  return attribute;
}
```

如果当前`request`容器中存在这个属性，则从容器中获取

```java
Object attribute = null;

if (mavContainer.containsAttribute(name)) {
  attribute = mavContainer.getModel().get(name);
}
```

否则就将这个对象`new`出来`Person`

```java
attribute = createAttribute(name, parameter, binderFactory, webRequest);
```

![截屏2021-04-02 00.43.17](http://qiniu-note-image.ctong.top/note/images/202112271049672.png)

`WebDataBinder`是一个web数据绑定组件，将请求参数的值绑定到指定的`JavaBean`中，指定的`JavaBean`就是`attribute`

```java
WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
```

`WebDataBinder`利用它里面的`converters`将请求数据转换成指定的数据类型，再次封装到`JavaBean`

![WebDataBinder结果](http://qiniu-note-image.ctong.top/note/images/202112271049686.png)

将请求中的数据映射到`binder`中的`target`，也就是`attribute`

```java
if (!mavContainer.isBindingDisabled(name)) {
   bindRequestParameters(binder, webRequest);
}
```

![数据映射到attribute](http://qiniu-note-image.ctong.top/note/images/202112271049036.png)

映射值的源码在`org.springframework.beans.AbstractPropertyAccessor`

```java
@Override
public void setPropertyValues(PropertyValues pvs, boolean ignoreUnknown, boolean ignoreInvalid)
  throws BeansException {

  List<PropertyAccessException> propertyAccessExceptions = null;
  List<PropertyValue> propertyValues = (pvs instanceof MutablePropertyValues ?
                                        ((MutablePropertyValues) pvs).getPropertyValueList() : Arrays.asList(pvs.getPropertyValues()));

  if (ignoreUnknown) {
    this.suppressNotWritablePropertyException = true;
  }
  try {
    for (PropertyValue pv : propertyValues) {
      // setPropertyValue may throw any BeansException, which won't be caught
      // here, if there is a critical failure such as no matching field.
      // We can attempt to deal only with less serious exceptions.
      try {
        setPropertyValue(pv);
      }
      catch (NotWritablePropertyException ex) {
        if (!ignoreUnknown) {
          throw ex;
        }
        // Otherwise, just ignore it and continue...
      }
      catch (NullValueInNestedPathException ex) {
        if (!ignoreInvalid) {
          throw ex;
        }
        // Otherwise, just ignore it and continue...
      }
      catch (PropertyAccessException ex) {
        if (propertyAccessExceptions == null) {
          propertyAccessExceptions = new ArrayList<>();
        }
        propertyAccessExceptions.add(ex);
      }
    }
  }
  finally {
    if (ignoreUnknown) {
      this.suppressNotWritablePropertyException = false;
    }
  }

  // If we encountered individual exceptions, throw the composite exception.
  if (propertyAccessExceptions != null) {
    PropertyAccessException[] paeArray = propertyAccessExceptions.toArray(new PropertyAccessException[0]);
    throw new PropertyBatchUpdateException(paeArray);
  }
}
```

 

#### 自定义Converter原理

使用**WebMVC**拓展的方式添加/修改配置

```java

@Configuration
public class WebConfig implements Serializable {

  private static final long serialVersionUID = -3281050128601582429L;

  public WebConfig() { }

  @Bean
  public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {

      @Override
      public void addFormatters(FormatterRegistry registry) {}

    };
  }

}
```

`WebMvcConfigurer`实现`addFormatters`方法添加`Converter`

使用`FormattersRegister`提供的`addConverter`方法来添加

`String source`这是前端传过来的值

```java
@Configuration
public class WebConfig implements Serializable {

  private static final long serialVersionUID = -3281050128601582429L;

  public WebConfig() { }

  @Bean
  public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {
      @Override
      public void addFormatters(FormatterRegistry registry) {
        System.out.println("registry");
        registry.addConverter(new Converter<String, Pet>() {

          @Override
          public Pet convert(String source) {
            ...
          }
        });
      }
    };
  }

}
```

例如前端传了个`name=xxx,xxx,xxx`，其中`xxx,xxx,xxx`是数组，你需要根据你规定规则来进行解析，例如有这么一个实体：

```java
@Data
public class Pet implements Serializable {

    private static final long serialVersionUID = 2248757905577331865L;

    private String name;
  
  	private Integer age;

    public Pet() { }

    public Pet(String name) {
        this.name = name;
    }

}
```

其中`name`是数组的第一位，`age`是数组的第二位，而程序是不知道你的规则的，需要你自定义`Converter`。

```java
@Override
public Pet convert(String source) {
  if (StringUtils.isEmpty(source)) return null;
	Pet pet = new Pet();
  String[] split = source.split(",");
  pet.setName(split[0]);
  pet.setAge(split[1]);
  return pet;
}
```



### 数据响应与内容协商

![数据响应流程](http://qiniu-note-image.ctong.top/note/images/202112271049041.png)



#### 响应JSON

如果想要响应JSON，可以使用web场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

web场景默认帮我们引入了`json`场景`spring-boot-starter-json`

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-json</artifactId>
  <version>2.4.3</version>
  <scope>compile</scope>
</dependency>
```

当然，有了以上场景就不代表我们每个请求都有了处理JSON的能力，需要配合`@ResponseBody`来进行使用。例如：

```java

@Controller
public class ResponseBodyController implements Serializable {

  private static final long serialVersionUID = 3392266298591310673L;

  public ResponseBodyController() {}

  @GetMapping("/get-person")
  @ResponseBody
  public Person getPerson() {
		String[] likes = {"女"};
    Person person = new Person();
    Pet pet = new Pet();
    pet.setName("小黄");

    person.setAge(18);
    person.setInterest(likes);
    person.setName("UpYou");
    person.setPet(pet);

    return person;
  }

}
```



##### 返回值解析器

在`org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod()`方法中，使用返回值解析起来解析我们方法的返回值。

![返回值解析器](http://qiniu-note-image.ctong.top/note/images/202112271049264.png)

在`org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle()`方法中执行目标方法，得到该方法的返回值。

```java
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
```

若返回值不是一个**String/null**类型，则使用`this.returnValueHandlers.handleReturnValue`方法来进行处理返回值。

```java
this.returnValueHandlers.handleReturnValue(
      returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
```

断点走到`this.returnValueHandlers.handleReturnValue`方法中...

```java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                              ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

  HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
  if (handler == null) {
    throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
  }
  handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
}
```

通过`selectHandler`来获取受支持的返回值处理器

```java
HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
```

走到`selectHandler`方法中

```java
private HandlerMethodReturnValueHandler selectHandler(@Nullable Object value, MethodParameter returnType) {
  boolean isAsyncValue = isAsyncReturnValue(value, returnType);
  for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
    if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
      continue;
    }
    if (handler.supportsReturnType(returnType)) {
      return handler;
    }
  }
  return null;
}
```

检查是否为异步返回值

```java
boolean isAsyncValue = isAsyncReturnValue(value, returnType);
```

遍历每个返回值处理器，检查哪个支持这种返回值类型

```java
for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
  if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
    continue;
  }
  if (handler.supportsReturnType(returnType)) {
    return handler;
  }
}
```

若是异步返回值，SpringBoot为提供能为其处理的处理器。

```java
if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
  continue;
}
```

判断当前返回值处理器是否支持当前返回值，若当前处理器能够处理，则将其返回

```java
handler.supportsReturnType(returnType)
```

拿到支持的处理器之后，调用该处理器的`handleReturnValue`方法处理返回值

```java
handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
```

最终找到了`RequestResponseBodyMthodProcessor`。这个返回值解析器支持解析被标注了`@ResponseBody`的方法。

```java
@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                              ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
  throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

  mavContainer.setRequestHandled(true);
  ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
  ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

  // Try even with null return value. ResponseBodyAdvice could get involved.
  writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
}
```

拿到返回值解析器之后，使用消息转换器进行输出到浏览器

```java
writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
```

在这个方法中，通过请求头`Accept` 匹配服务端能不能生产浏览器能处理的内容，浏览器通过指定请求头发送浏览器能处理的类型，其中`*/*`表示所有内容都能尝试处理。服务器会根据浏览器能处理的数据类型进行匹配，如果找到多个能处理的类型，则通过浏览器提供的类型权重进行匹配。

```java
protected <T> void writeWithMessageConverters(@Nullable T value, MethodParameter returnType,
			ServletServerHttpRequest inputMessage, ServletServerHttpResponse outputMessage)
			throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {...}
```

![浏览器能支持的内容](http://qiniu-note-image.ctong.top/note/images/202112271049460.png)





####  消息转换器原理（HttpMssageConverter）



##### MessageConverter 规范

`HttpMessageConverter`是一个规范消息转换器的一个接口

![HttpMssageConverter规范](http://qiniu-note-image.ctong.top/note/images/202112271049533.png)

`canRead`方法检查当前转换器是否能支持读取这个类型

`canWrite`检查当前转换器是否支持写入这种类型



##### 默认MessageConverter

![SpringBoot默认的MessageConverter](http://qiniu-note-image.ctong.top/note/images/202112271049700.png)

0. 只支持Byte类型

1. 只支持String类型

2. 只支持String类型

3. 只支持Resource类型

4. 只支持ResourceRegion类型

5. Source支持以下类型

   ```
   DOMSource.class
   SAXSource.class
   StAXSource.class
   StreamSource.class
   Source.class
   ```

6. AllEncompassing只支持MultiValueMap类型
7. 支持所有类型，也就是`*/*`
8. 同上
9. 支持注解XmlRootElement方式的xml



当拿到匹配的转换器后，调用转换器中的`write`方法

```java
@Override
public final void write(final T t, @Nullable final Type type, @Nullable MediaType contentType,
                        HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {

  final HttpHeaders headers = outputMessage.getHeaders();
  addDefaultHeaders(headers, t, contentType);

  if (outputMessage instanceof StreamingHttpOutputMessage) {
    StreamingHttpOutputMessage streamingOutputMessage = (StreamingHttpOutputMessage) outputMessage;
    streamingOutputMessage.setBody(outputStream -> writeInternal(t, type, new HttpOutputMessage() {
      @Override
      public OutputStream getBody() {
        return outputStream;
      }
      @Override
      public HttpHeaders getHeaders() {
        return headers;
      }
    }));
  }
  else {
    writeInternal(t, type, outputMessage);
    outputMessage.getBody().flush();
  }
}
```

当前转换器是一个支持Json的转换器，所以数据返回值时需要将请求头设置为`content-type: 'application/json'`

```java
final HttpHeaders headers = outputMessage.getHeaders();
addDefaultHeaders(headers, t, contentType);
```

最后调用`writeInternal`方法，将返回数据写出去。

在`writeInternal`中，`objectWriter.writeValue`将数据转为json，`generator.flush();`将数据写到`outputMessage`中

```java
objectWriter.writeValue(generator, value);
writeSuffix(generator, object);
generator.flush();
```

![flush之后的Response](http://qiniu-note-image.ctong.top/note/images/202112271049909.png)





#### 内容协商

> 根据客户端的接收能力不同，返回不同媒体类型的数据



##### 引入xml依赖

**千万别指定版本，启动会报错**

```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
  <!-- <version>2.12.3</version> -->
</dependency>
```



##### 测试返回json和xml

只需要改变请求头中Accept字段，该字段是http协议中规定的，用于告诉服务器当前客户端可以接受处理的数据类型。



##### 内容协商

主要逻辑在`org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters`方法中

1. 检查当前响应头中是否已经有媒体类型`MediaType`，这个媒体类型可能是在自定义拦截器中被设置的。

   ```java
   MediaType selectedMediaType = null;
   MediaType contentType = outputMessage.getHeaders().getContentType();
   ```

   

2. 获取客户端支持的类型，也就是请求头`Accept`中的内容例如`application/xml`

   ```java
   List<MediaType> acceptableTypes = getAcceptableMediaTypes(request);
   ```

   

3. 循环当前系统所有的`MessageConverter`，检查哪个支持解析当前返回值类型。最后将这些支持的媒体类型进行统计

   ```java
   List<MediaType> producibleTypes = getProducibleMediaTypes(request, valueType, targetType);
   ```

   

4. 匹配处理当前返回值最佳的`MessageConverter`

   ```java
   List<MediaType> mediaTypesToUse = new ArrayList<>();
   for (MediaType requestedType : acceptableTypes) {
      for (MediaType producibleType : producibleTypes) {
         if (requestedType.isCompatibleWith(producibleType)) {
            mediaTypesToUse.add(getMostSpecificMediaType(requestedType, producibleType));
         }
      }
   }
   ```

   

5. 将匹配出来的`MessageConverter`进行排序

   ```java
   MediaType.sortBySpecificityAndQuality(mediaTypesToUse);
   ```

   

6. 进行最佳匹配，最后拿到的只能有一个

   ```java
   for (MediaType mediaType : mediaTypesToUse) {
     if (mediaType.isConcrete()) {
       selectedMediaType = mediaType;
       break;
     }
     else if (mediaType.isPresentIn(ALL_APPLICATION_MEDIA_TYPES)) {
       selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
       break;
     }
   }
   ```

   

7. 最后再进行内容协商的最佳匹配，再使用匹配出来的最佳converter，再调用它进行转换。

   ```java
   for (HttpMessageConverter<?> converter : this.messageConverters) {
     GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
                                                     (GenericHttpMessageConverter<?>) converter : null);
     if (genericConverter != null ?
         ((GenericHttpMessageConverter) converter).canWrite(targetType, valueType, selectedMediaType) :
         converter.canWrite(valueType, selectedMediaType)) {
       body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
                                          (Class<? extends HttpMessageConverter<?>>) converter.getClass(),
                                          inputMessage, outputMessage);
   ```

   

8. 最后响应出去，就得到了它
   ![内容协商响应的结果](http://qiniu-note-image.ctong.top/note/images/202112271049978.png)



##### 开启浏览器参数方式内容协商功能

为了方便内容协商，开启基于请求参数的内容协商功能。在`application.yaml`中开启

```yaml
spring:
  mvc:
    # 开启参数方式的内容协商
    contentnegotiation:
      favor-parameter: true
```

开启之后只需要通过`format`参数就能更改返回值类型，例如

`{{api}}/get-person?format=xml`

`{{api}}/get-person?format=xml`

开启基于参数的内容协商功能之后，底层的内容协商管理器多了一个基于参数的内容协商策略，它只支持两种类型：`json`和`xml`

![内容协商管理器](http://qiniu-note-image.ctong.top/note/images/202112271049008.png)



##### 自定义MessageConverter

> 若需要自定义任何`SpringMVC`的配置，去`WebMvcAutoConfiguration`类中参考



###### 根据请求头处理的MessageConverter

若需要添加自定义`MessageConverter`需要实现`WebMvcConfigurer`接口中的`extendMessageConverters`方法，这个方法是用来拓展`MessgeConverter`的

```java
@Configuration
public class WebConfig implements Serializable {

  private static final long serialVersionUID = -3281050128601582429L;

  public WebConfig() { }

  @Bean
  public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {
      @Override
      public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new MyConverter());
      }

    };
  }

}
```

通过往**SpringMVC**提供的`List<HttpMessageConverter<?>> converters`中添加数据，这是一个`MessageConverter`集合

```java
converters.add(new MyConverter());
```

实现一个`MessageConverter`

- `canRead(Class<?> clazz, MediaType mediaType)` 是否支持读
  - 底层通过调用这个方法来判断当前`MessageConverter`是否支持处理读取
- `canWrite(Type type, Class<?> clazz, MediaType mediaType)`是否支持写
  - 底层通过这个方法判断是否支持处理浏览器支持的类型
- `getSupportedMediaTypes(Class<?> clazz)`获取支持的类型
  - 底层通过请求过来的请求头`Accept`对比，当前`MessageConverter`是否支持该类型，例如:`application/json`
- `write(Person person, Type type, MediaType contentType, HttpOutputMessage outputMessage)`将数据处理并写入`outputMessage`
  - 如果当前`MessageConverter`被匹配到了，则用这个`MessageConverter`处理数据

```java
class MyConverter implements GenericHttpMessageConverter<Person> {

  @Override
  public void write(Person person, Type type, MediaType contentType, HttpOutputMessage outputMessage) throws
    IOException,
  HttpMessageNotWritableException {
    String data = person.toString();
    byte[] bytes = data.getBytes(StandardCharsets.UTF_8);
    OutputStream body = outputMessage.getBody();
    body.write(bytes);
    body.flush();
  }

  @Override
  public boolean canRead(Class<?> clazz, MediaType mediaType) {
    return false;
  }

  @Override
  public boolean canWrite(Type type, Class<?> clazz, MediaType mediaType) {
    // 如果返回值为Persion，则支持处理
    return clazz.isAssignableFrom(Person.class);
  }

  @Override
  public List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
    return MediaType.parseMediaTypes("application/c-tong");
  }

}
```

> 注意实现的是`GenericHttpMessageConverter`而不再是`HttpMessageConverter`
>
> 我按照教程走，发现我定义的无效，原来是底层改成了`GenericHttpMessageConverter`
>
> ![匹配MessageConverter代码](http://qiniu-note-image.ctong.top/note/images/202112271049481.png)



重新发送请求之后结果就是我们自定义的`MessageConverter`处理返回的结果

![使用自定义MessageConverter处理结果](http://qiniu-note-image.ctong.top/note/images/202112271049488.png)



###### 根据参数处理的MessageConverter



```java
@Configuration
public class WebConfig implements Serializable {

  private static final long serialVersionUID = -3281050128601582429L;

  public WebConfig() { }

  @Bean
  public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {

      @Override
      public void configureContentNegotiation(ContentNegotiationConfigurer configurer) { ... }
    };
  }
}
```

添加`strategies`

```java
@Override
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {

  WebMvcConfigurer.super.configureContentNegotiation(configurer);
  Map<String, MediaType> mediaTypes = new HashMap<>(3);
  mediaTypes.put("ctong", MediaType.parseMediaType("application/c-tong"));
  mediaTypes.put("json", MediaType.parseMediaType("application/json"));
  mediaTypes.put("xml", MediaType.parseMediaType("application/xml"));
  /// 基于参数的Strategy
  ContentNegotiationStrategy parameterStrategy = new ParameterContentNegotiationStrategy(mediaTypes);
  /// 基于请求头的Strategy
  HeaderContentNegotiationStrategy headerStrategy = new HeaderContentNegotiationStrategy();

  List<ContentNegotiationStrategy> strategy = Arrays.asList(parameterStrategy, headerStrategy);
  /// 全部添加到Strategies
  configurer.strategies(strategy);
}
```

如果没有添加`Header`的`MessageConverter`，则无法根据请求头中的`Aeecpt`进行匹配，最后无论`Aeecpt`是什么，都将当作`*/*`来进行使用。

```java
HeaderContentNegotiationStrategy headerStrategy = new HeaderContentNegotiationStrategy();
```

> 如果我们添加了自定义的功能，则可能会将SpringBoot底层定义好的功能覆盖，导致默认功能失效。



## 拦截器

处理请求时，若是需要判断用户是否登录，若是没有登录，就禁止用户访问。这种操作可以使用拦截器来处理，或者也可以使用原生servlet的filter来处理。

在SpringBoot中，`HandlerInterceptor`是请求拦截器接口，它有三个方法可实现

1. `preHandle` 预先处理
2. `postHandle`目标方法执行完成后处理
3. `afterCompletion` 页面渲染后处理



### 定义拦截器

```java
public class LoginInterceptor implements Serializable, HandlerInterceptor {

  private static final long serialVersionUID = 7221439597788131747L;

  public LoginInterceptor() {}

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {
    HttpSession session = request.getSession();
    // 是否登录
    return session.getAttribute("loginUser") != null;
  }

}
```

定义拦截器后，需要将拦截器放到容器中

```java
@Configuration
public class WebConfig implements Serializable, WebMvcConfigurer {

  private static final long serialVersionUID = -4663219618421154965L;

  public WebConfig() {}

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoginInterceptor())
      .addPathPatterns("/**")
      .excludePathPatterns("/login", "/login.html","/css/**", "/js/**", "/fonts/**", "/images/**");
    WebMvcConfigurer.super.addInterceptors(registry);
  }

}

```

- `addPathPatterns` 过滤路径，`/**`表示过滤所有路径
- `excludePathPatterns` 放行路径，对指定的路径进行放行，也就是不拦截。



### 拦截器原理

当发送一个请求到服务端后，spring的`org.springframework.web.servlet.DispatcherServlet#doDispatch`方法会根据这个请求进行匹配一个**HandlerExecutionChain**。里面包含了可以处理当前请求的handler，其中有一个interceptorList属性，它是一个拦截器链。可以看到，spring中有两个默认拦截器，而`LoginInterceptor`是我们自定义的拦截器。



![截屏2021-07-02 下午3.46.37](http://qiniu-note-image.ctong.top/note/images/202112271049629.png)

在`org.springframework.web.servlet.DispatcherServlet#doDispatch`中有这么几行代码。` mappedHandler.applyPreHandle` 执行所有拦截器的`preHandler`方法，`ha.handle`是执行执行目标方法，`applyPostHandle`是执行所有拦截器中的`postHandle`方法

```java
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
  return;
}

// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

mappedHandler.applyPostHandle(processedRequest, response, mv);
```

在`mappedHandler.applyPreHandle`方法中，通过循环容器中的所有拦截器，并执行所有`preHandler()`

```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
  for (int i = 0; i < this.interceptorList.size(); i++) {
    HandlerInterceptor interceptor = this.interceptorList.get(i);
    if (!interceptor.preHandle(request, response, this.handler)) {
      triggerAfterCompletion(request, response, null);
      return false;
    }
    this.interceptorIndex = i;
  }
  return true;
}
```

> `org.springframework.web.servlet.HandlerExecutionChain#applyPreHandle`

如果有拦截器没有放行，则会调用`triggerAfterCompletion(request, response, null);`方法，这个方法中会执行所有已经执行`preHandler`的拦截器中的`afterCompletion()`方法。

```java
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) {
  for (int i = this.interceptorIndex; i >= 0; i--) {
    HandlerInterceptor interceptor = this.interceptorList.get(i);
    try {
      interceptor.afterCompletion(request, response, this.handler, ex);
    }
    catch (Throwable ex2) {
      logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
    }
  }
}
```

> `org.springframework.web.servlet.HandlerExecutionChain#triggerAfterCompletion`

当目标方法执行完毕之后也就是`ha.handle`方法执行完毕后开始执行`mappedHandler.applyPostHandle`方法

```java
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
  throws Exception {

  for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
    HandlerInterceptor interceptor = this.interceptorList.get(i);
    interceptor.postHandle(request, response, this.handler, mv);
  }
}
```

> `org.springframework.web.servlet.HandlerExecutionChain#applyPostHandle`

值得注意的是，无论请求发生任何异常，都会执行所有已执行拦截器的`afterCompletion`

![截屏2021-07-02 下午5.02.17](http://qiniu-note-image.ctong.top/note/images/202112271049188.png)

当请求执行结束后，执行所有已执行拦截器的`afterCompletion`

![截屏2021-07-02 下午11.00.16](http://qiniu-note-image.ctong.top/note/images/202112271049191.png)



## 文件上传

若需要上传文件，请求头中的`Content-Type`必须为`multipart/form-data`。可以使用`@RequestPart`解析`multipart/form-data`中的参数，`@RequestParam`也可以完成此操作。

```html
<form role="form" action="/upload" method="post" enctype="multipart/form-data">
  <div class="form-group">
    <label for="exampleInputEmail1">邮箱</label>
    <input type="email" name="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
  </div>
  <div class="form-group">
    <label for="exampleInputPassword1">名字</label>
    <input type="text" name="userName" class="form-control" id="exampleInputPassword1" placeholder="Password">
  </div>
  <div class="form-group">
    <label for="exampleInputFile">头像</label>
    <input type="file" name="headerImg" id="exampleInputFile">
  </div>
  <div class="form-group">
    <label for="exampleInputFile">生活照</label>
    <input type="file" name="photos" multiple>
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox"> Check me out
    </label>
  </div>
  <button type="submit" class="btn btn-primary">提交</button>
</form>
```



```java
@PostMapping("/upload")
public String upload(@RequestParam("email") String email,
                     @RequestParam("userName") String userName,
                     @RequestPart("headerImg") MultipartFile headerImg,
                     @RequestPart("photos") MultipartFile[] photos) throws IOException {
  log.info("上传的信息：email={}, userName={}, headerImg={}, photos={}",
           email, userName, headerImg.getSize(), photos.length);
  String srcPath = "/Users/ctong/Desktop/";

  /// 保存图片到本地
  if (!headerImg.isEmpty()) {
    headerImg.transferTo(new File(srcPath+headerImg.getOriginalFilename()));
  }

  if (photos.length > 0) {
    for (MultipartFile photo : photos) {
      if (!photo.isEmpty()) {
        photo.transferTo(new File(srcPath + photo.getOriginalFilename()));
      }
    }
  }
  return "/index";
}
```

如果你上传的图片大小超过了默认的`1MB`就会出现以下错误。需要在**application.yaml配置文件**中自定义文件大小

```java
Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.web.multipart.MaxUploadSizeExceededException: Maximum upload size exceeded; nested exception is java.lang.IllegalStateException: org.apache.tomcat.util.http.fileupload.impl.FileSizeLimitExceededException: The field photos exceeds its maximum permitted size of 1048576 bytes.] with root cause
```

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
```



### 文件上传原理

文件上传自动配置类 => `org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration`

- 在`org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration#multipartResolver`中自动配置好了文件上传解析器「**StandardServletMultipartResolver**」。

- 在`org.springframework.web.servlet.DispatcherServlet#doDispatch`方法中判断当前请求是否是文件上传请求。

  ```java
  processedRequest = checkMultipart(request);
  ```

  在`checkMultipart`中通过判断请求头的`ContentType`是否为`multipart/`开头，如果是则当前请求为文件上传请求。

  ```java
  @Override
  public boolean isMultipart(HttpServletRequest request) {
    return StringUtils.startsWithIgnoreCase(request.getContentType(), "multipart/");
  }
  ```

  > `org.springframework.web.multipart.support.StandardServletMultipartResolver#isMultipart`

  最后通过`this.multipartResolver.resolveMultipart(request);`封装一个`MultipartHttpServletRequest`

  > `org.springframework.web.multipart.support.StandardServletMultipartResolver#resolveMultipart`

  `checkMultipart`完成后判断`processedRequest`和`request`是否相等，如果不相等则视为文件上传请求。因为在`checkMultipart`方法中，如果是一个文件上传请求，他会将request包装成一个**StandardServletMultipartResolver**

  ```java
  multipartRequestParsed = (processedRequest != request);
  ```

  

- 通过`org.springframework.web.multipart.support.MultipartResolutionDelegate#resolveMultipartArgument` 获取`MultiValueMap<String, MultipartFile>`中的参数。这个**Map**在一开始解析文件上传请求的时候就已经将文件流封装为`MultipartFile`保存在`MultiValueMap`中，而`RequestPartMethodArgumentResolver`解析器只是负责从`MultiValueMap`中把参数拿出来。



## 错误处理

- 默认情况下，SpringBoot提供`/error`处理所有错误的映射

- 对于机器客户端，它将生成JSON响应，其中包含错误Http状态和异常消息的详细信息，对于浏览器客户端，则响应一个**whitelabel**错误视图，以HTML格式呈现相同的数据。
  ![截屏2021-07-03 下午4.49.23](http://qiniu-note-image.ctong.top/note/images/202112271049199.png)

  

- 要对其进行自定义，需要添加`View`解析为`error`
  要完全替换默认行为，可以实现`ErrorController`并注册该类型的Bean定义，或添加`ErrorAttributes`类型的组件以使用现有机制替换其内容。

- `error/`下的4xx、5xx页面会被自动解析
  ![截屏2021-07-03 下午4.53.08](http://qiniu-note-image.ctong.top/note/images/202112271049536.png)



### 定制错误处理

- 自定义错误页
  - error/404.html
  - error/5xx.html
- @ControllerAdvice + @ExceptionHandler处理异常
- 实现Handler Exception Resolver处理异常



### 异常处理自动配置原理

自动配置异常处理规则在这个类中`org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration` 

这个类就是一个普通的`Controller`，如果没有在配置文件中指定错误路径`server.error.path`，那么则使用默认的错误路径`/error`

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {}
```

在这个类中，有两个方法响应这个请求

1. `org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController#error`

   ```java
   @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
   public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
     HttpStatus status = getStatus(request);
     Map<String, Object> model = Collections
       .unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
     response.setStatus(status.value());
     ModelAndView modelAndView = resolveErrorView(request, response, status, model);
     return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
   }
   ```

   如果当前请求是普通浏览器客户端，则使用这个方法处理，否则使用下面那个方法。当一个请求过来的时候，通过`resolveErrorView`方法获取视图解析器，默认解析器只有一个`DefaultErrorViewResolver`，而在这个视图解析器中通过`org.springframework.boot.autoconfigure.web.servlet.error.DefaultErrorViewResolver#resolve`方法得到一个视图，在这个方法内部，已经钉死了模版位置

   `String errorViewName = "error/" + viewName;`

   下一步获取可解析这个模板的模板解析器`this.templateAvailabilityProviders.getProvider`如果在这一步找不到可以解析这个模板的解析器，那么则返回一个可以通过**html**解析器进行解析的`ModelAndView`。

   > 这个`ModelAndView`并不是一个真正的视图，它是一个视图“模型”

   得到这些信息之后，其它步骤就是普通的解析。

   ![截屏2021-07-05 21.45.01](http://qiniu-note-image.ctong.top/note/images/202112271049663.png)

2. `org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController#mediaTypeNotAcceptable`

   ```java
   @RequestMapping
   public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
     HttpStatus status = getStatus(request);
     if (status == HttpStatus.NO_CONTENT) {
       return new ResponseEntity<>(status);
     }
     Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
     return new ResponseEntity<>(body, status);
   }
   ```

   这个其实和第一个一样，只不过返回类型变成一个普通的实体`ResponseEntity`，这个试图最终会以JSON的形式展示。

   

   ### 异常处理流程

   **“没搞清楚/error在哪转定义”**

   

   

   ## Web原生组件注入（Servlet、Filter、Listener）

   ### 使用ServletApi

   spring boot支持原生servlet的`@WebFilter`、`@WebServlet` 、`@WebListener`，它需要通过`@ServletComponentScan`注解对指定包进行扫描。

   

   #### @WebServlet

   ```java
   @WebServlet(urlPatterns = "/my")
   public class MyServlet extends HttpServlet {
   
     public MyServlet() {
       // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
     }
   
     @Override
     protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
       try {
         PrintWriter writer = resp.getWriter();
         writer.println("Hello World!");
       } catch (IOException e) {
         e.printStackTrace();
       }
     }
   
   }
   ```

   **注意，使用原生Servlet之后，无法被SpringBoot的拦截器所拦截**

   ![截屏2021-07-08 14.48.34](http://qiniu-note-image.ctong.top/note/images/202112271049875.png)

   

   #### @WebFilter

   使用原生servlet的filter

   ```java
   @WebFilter
   public class MyFilter implements Filter {
   
       public MyFilter() {
           // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
       }
   
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
           log.info("原生过滤器初始化");
           Filter.super.init(filterConfig);
       }
   
       @Override
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
               throws IOException, ServletException {
           log.info("原生过滤器开始执行");
           /// 放行
           chain.doFilter(request, response);
       }
   
       @Override
       public void destroy() {
           log.info("原生过滤器销毁");
           Filter.super.destroy();
       }
   
   }
   ```

   **原生servlet的filter也无法拦截spring boot的请求**

   

   

   #### @WebListener

   监听servlet是否启动成功

   ```java
   @WebListener
   @Slf4j
   public class MyServletContextListener implements ServletContextListener {
   
     public MyServletContextListener() {
       // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
     }
   
     @Override
     public void contextInitialized(ServletContextEvent sce) {
       log.info("MyServletContextListener监听到项目初始化完成");
     }
   
     @Override
     public void contextDestroyed(ServletContextEvent sce) {
       log.info("MyServletContextListener监听到项目被销毁");
     }
   
   }
   ```

   

   

   #### @ServletComponentScan

   ```java
   @ServletComponentScan(basePackages = "com.ctong.learnspringboot") // 指定包进行扫描，将原生Servlet注入到Spring
   @SpringBootApplication
   public class LearnSpringBootApplication {
   
      public static void main(String[] args) {
         SpringApplication.run(LearnSpringBootApplication.class, args);
      }
   
   }
   ```

   

   

   ### 使用RegistrationBean

   除了通过`@WebServlet`、`@WebFilter`、`@WebListener`外，Springboot还提供了它自己的注册方式，`ServletRegistrationBean`、`FilterRegistrationBean`、`ServletListenerRegistrationBean`。使用`RegistrationBean`无需通过`ServletComponentScan`进行扫描，因为它本身就是一个`Component`

   ```java
   @Configuration
   public class MyRegisterConfig {
   
     public MyRegisterConfig() {
       // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
     }
   
     /**
        * SpringBoot提供的多种原生Servlet注入方式之一
        *
        * @return ServletRegistrationBean
        */
     @Bean
     public ServletRegistrationBean<MyServlet> servletRegistrationBean() {
       MyServlet myServlet = new MyServlet();
       // ServletRegistrationBean 第一个参数代表需要注入的servlet， 第二个参数是一个数组，表示这些url都通过这个servlet进行处理
       return new ServletRegistrationBean<>(myServlet, "/my");
     }
   
     /**
        * SpringBoot 提供的多种原生Filter注入方式之一
        *
        * @return FilterRegistrationBean
        */
     @Bean
     public FilterRegistrationBean<MyFilter> filterRegistrationBean() {
       MyFilter myFilter = new MyFilter();
       // 拦截指定servlet
       FilterRegistrationBean<MyFilter> filter = new FilterRegistrationBean<>(myFilter, servletRegistrationBean());
       List<String> patterns = Arrays.asList("/css/*", "/js/*", "/fonts/*", "/images/*");
       filter.setUrlPatterns(patterns);
       return filter;
     }
   
     /**
        * SpringBoot 提供的多种原生Listener注入方式之一
        * @return ServletListenerRegistrationBean
        */
     @Bean
     public ServletListenerRegistrationBean<MyServletContextListener> servletContextListenerRegistrationBean() {
       MyServletContextListener listener = new MyServletContextListener();
       return new ServletListenerRegistrationBean<>(listener);
     }
   
   }
   ```

   

   ### 原理

   在SpringBoot底层中，也是通过原生Servlet进行编写，StringBoot中默认Servlet处理的路径是`/`，源码在`org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration.DispatcherServletConfiguration#dispatcherServlet`。

   其实和我们注入原生Servlet是一样的`org.springframework.web.servlet.DispatcherServlet#doService`，他这个`doService`等价于`javax.servlet.http.HttpServlet#service(javax.servlet.http.HttpServletRequest, javax.servlet.http.HttpServletResponse)`方法

   ![Shot_20210708_153831](http://qiniu-note-image.ctong.top/note/images/202112271049332.png)

   在tomcat中，若是存在多个servlet，它会进行**精确匹配**，例如有两个servlet，A处理`/`路径，B处理`/my`路径。用户端发送一个`/my`请求过来例如：127.0.0.1:80/my/login。那么tomcat就会使用B的规则去处理这个请求。

   ![截屏2021-07-08 15.22.34](http://qiniu-note-image.ctong.top/note/images/202112271049327.png)

   

   

   ## 嵌入式Servlet容器

   [官网文档传送门](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.embedded-container.application-context)

   

   ### 切换嵌入式Servlet容器

   Spring Boot 内嵌了一个tomcat服务器，除了tomcat，它还支持其它的`WebServlet`，例如`Jetty`、`Undertow`。SpringBoot是通过`org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext`容器启动并寻找`org.springframework.boot.web.servlet.server.ServletWebServerFactory`

   

   ### 切换servlet服务器

   由于SpringBoot的web场景内嵌了tomcat，所以需要将tomcat剔除然后导入我们需要的servlet服务器。在导入之前，需要排除springboot默认引入的tomcat服务器，因为tomcat服务器的优先级最高。如果不排除会导致我们指定的服务器无法被添加进容器

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
     <exclusions>
       <!-- 排除默认导入的tomcat -->
       <exclusion>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-tomcat</artifactId>
       </exclusion>
     </exclusions>
   </dependency>
   ```

   排除tomcat依赖后导入我们需要的服务器即可

   ```xml
   <!-- 导入undertow服务器 -->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-undertow</artifactId>
   </dependency>
   ```

   

   

   ### 原理

   - spring boot应用启动发现当前是web应用。web场景包会自动导入tomcat
   - web应用会创建一个web版的ioc容器`org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext`
   - 当这个容器启动的时候会自动寻找`org.springframework.boot.web.servlet.server.ServletWebServerFactory`
   - Spring Boot 底层有很多WebServlet工厂：`TomcatServletWebServerFactory`, `JettyServletWebServerFactory`, `UndertowServletWebServerFactory` 他们分别对应`tomcat`、`jetty`、`undertow`
   - 在底层有一个自动配置类，里面导入了默认的WebServlet
     `org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration`。在这个自动配置类中导入了`org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryConfiguration`配置类，在这个类中动态判断系统中到底导入了哪个个Web服务器的包（默认tomcat）`TomcatServletWebServerFactory`
   - 在`org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory`中创建出tomcat服务器并启动。

   

   ### 定制servlet容器

   1. 实现`WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>`接口，通过`customize`方法将配置文件的值和`ServletWebServerFactory`中相关的属性进行绑定。
   2. 修改配置文件，`server.xxx`，与`servlet`相关的配置都在`server.xxx`下，具体可以查看`org.springframework.boot.autoconfigure.web.ServerProperties`
   3. 添加`org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory`组件到容器中，这种方式相当于

   

   ## 定制化原理

    ### 定制化的常见方式

   - 修改配置文件

   - 实现xxx Customizer类

   - 编写自定义的配置类xxxConfiguration，通过`@Bean`添加或替换容器中默认的组件

   - web应用实现`WebMvcConfiguration`即可定制化web功能。

     ```java
     @Configuration
     public class WebConfig implements WebMvcConfigurer {}
     ```
     
   - `@EnableWebMvc` + `WebMvcConfiguration`。这种方式将全面接管SpringMvc，所有规则全部自己重新配置。实现定制和功能的拓展

     - 原理

       1. `WebMvcAutoConfiguration`是SpringMvc默认的自动配置类，其中配置了静态资源规则、欢迎页等规则...

       2. `@EnableWebMvc`中会导入`DelegatingWebMvcConfiguration`，它继承了`WebMvcConfigurationSupport`
       3. `DelegatingWebMvcConfiguration`的作用是只保证SpringMvc最基本的使用
          1. 这个配置类会把系统中所有的`WebMvcConfiguration`拿过来。所有的功能定制都是这些`WebMvcConfiguration`合起来的结果
          2. 自动配置了一些非常底层的组件。`RequestMapping`、`HandlerMapping`，这些组件依赖的组件都是从容器中获取
       4. 在`org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration`中，有一个生效条件`@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`当`WebMvcConfigurationSupport.class`不存在时生效

   

   ### 原理分析

   导入所需场景，例如`starter-web` ->> 

   之后这个starter场景会导入xxxAutoConfiguration自动配置类 ->>

   在这个自动配置类中会使用@Bean导入一系列组件 ->>

   在组件中的默认配置都会跟xxxProperties进行绑定 ->>

   最后xxProperties又和配置文件进行绑定

   

## 指标监控

[官网相关文档](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)

### SpringBoot Actuator

未来每一个微服务在云上部署后，需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我门每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```



#### 使用

- 引入actuator场景

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

- 访问http://localhost:8080/actuator/\*\*

- Web下默认暴露的端点只有`health`和`info`，可以在配置中暴露所有端点信息

  ```yaml
  # actuator 监控指标
  management:
    endpoints:
      enabled-by-default: true # 默认开启所有监控端点
      web:
        exposure:
          include: '*' # 以web方式暴露所有监控端点
  ```



#### 可视化

https://github.com/codecentric/spring-boot-admin



### Actuator Endpoint

常用的Endpoint有

- Health
  监控状况
- Metrics
  运行时指标
- Loggers
  日志记录



#### Health Endpoint

健康检查端点，一般用在云平台，平台会定时检查应用的健康状况。需要Health Endpoint为平台返回当前应用的一系列组件健康状况集合。

- health endpoint 返回的结果，应该是一系列健康检查后的一个汇总报告
- 很多的健康检查默认已经自动配好了，比如：数据库，redis等
- 可以自定义添加健康检查机制



开启指定端点详细信息

```yaml
# actuator 监控指标
management:
  endpoint:
    health:
      show-details: always # 显示详细
```

![截屏2021-07-14 15.04.50](http://qiniu-note-image.ctong.top/note/images/202112271049555.png)





#### Metrics Endpoint

提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到：

- 通过Metrics对接多种监控系统
- 简化核心Metrics开发
- 添加自定义Metrics或者扩展已有Metrics



![截屏2021-07-14 15.10.09](http://qiniu-note-image.ctong.top/note/images/202112271049574.png)



![截屏2021-07-14 15.11.13](http://qiniu-note-image.ctong.top/note/images/202112271049845.png)



  ### 管理Endpoint



#### 开启与禁用Endpoint

默认情况下，endpoint是全部开启的，若需要指定开启，需要全部关闭后再开启指定端点

```yaml
# actuator 监控指标
management:
  endpoints:
    enabled-by-default: false # 关闭全部端点
  endpoint:
    health:
      show-details: always # 显示详细信息
      enabled: true # 开启端点
    info:
      enabled: true
    metrics:
      enabled: true
```





### 定制Endpoint



#### 定制Health信息

实现`HealthIndicator`接口或者`AbstractHealthIndicator`抽象类。

```java
@Component
public class ComponentHealthIndicator extends AbstractHealthIndicator implements Serializable {

  private static final long serialVersionUID = 5066467685531996717L;

  public ComponentHealthIndicator() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @Override
  protected void doHealthCheck(Health.Builder builder) throws Exception {
    Map<String, Object> map = new HashMap<>();
    if (false) {
      builder.status(Status.UP);
      map.put("count", 1);
      map.put("ms", 100);
    } else {
      builder.status(Status.OUT_OF_SERVICE);
      map.put("err", "连接超时");
      map.put("ms", 3000);
    }
    builder.withDetail("code", 200).withDetails(map);
  }

}
```

![image-20210714154234818](http://qiniu-note-image.ctong.top/note/images/202112271049017.png)



#### 定制info信息

定制info有两种方法，一种是在application.yaml配置文件中定义 

```yaml
info:
  version: @project.version@ # 获取pom文件中的版本信息
  autor: Clover
  age: 19
```

![截屏2021-07-14 15.45.20](http://qiniu-note-image.ctong.top/note/images/202112271049192.png)



第二种方式，可以实现`InfoContributor`接口的方式实现，第二种优先级高于配置文件方式。

```java
@Component
public class AppInfoContributor implements Serializable, InfoContributor {

  private static final long serialVersionUID = -2489956740668578470L;

  public AppInfoContributor() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @Override
  public void contribute(Info.Builder builder) {
    Map<String, Object> info = new HashMap<>();
    info.put("author", "Clover.You");
    info.put("age", 19);
    info.put("version", "1.0.0.1");
    builder.withDetails(info);
  }

}
```



#### 定制Metrics信息

##### 增加定制Metrics

```java
@Slf4j
@Controller
public class LoginController implements Serializable {

  private static final long serialVersionUID = -2783841528321793923L;

  // 计数指标
  private Counter counter;

  public LoginController(MeterRegistry meter) {
    // 注册自定义计数指标，记录登录方法调用了几次
    counter = meter.counter("com.ctong.learnspringboot.controller.LoginController.login");
  }

  @PostMapping("/login")
  public String login(HttpSession session, User user) {
    // 增加计数
    counter.increment();
    session.setAttribute("loginUser", user);
    return "redirect:/index.html";
  }

}
```

注册成功后，访问localhost:8080/actuator/metrics就可以看见我们自定义的新指标

![image-20210714162110608](http://qiniu-note-image.ctong.top/note/images/202112271049497.png)



计数会随着`counter.increment();`方法的调用而增加

![截屏2021-07-14 16.22.11](http://qiniu-note-image.ctong.top/note/images/202112271049795.png)



#### 定义Endpoint

SpringBoot给我们定义了非常多的监控端点，一旦我们引入 了复杂场景，可能需要自定义监控端点。

```java
@Component
@Endpoint(id = "diy")
@Slf4j
public class MyEndpoint implements Serializable {

  private static final long serialVersionUID = 1370368576988354134L;

  public MyEndpoint() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @ReadOperation
  public Map getDockerInfo() {
    return Collections.singletonMap("docker", "docker started...");
  }

  @WriteOperation
  public void stopDocker() {
    log.info("docker stopped....");
  }

}
```






   >    SpringBoot由多种解析器组成

   