## 三、Spring Boot 静态资源的映射规则

在前面已经提到过，如果我们需要 web 环境开发，只需要添加一个 web 环境的启动器即可，`Spring Boot` 就会自动给我们将这个环境所有的依赖以及常规的配置给我们配置好。针对于 web 环境，其实是由一个类 `WebMvcAutoConfiguration` 来进行配置的，在这个类中，就有静态资源的映射规则，不多说了，直接上码：

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache()
					.getCachecontrol().toHttpCacheControl();
    		//配置webjars资源映射规则
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod))
						.setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    		// 静态文件夹映射
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(getResourceLocations(
										this.resourceProperties.getStaticLocations()))
								.setCachePeriod(getSeconds(cachePeriod))
								.setCacheControl(cacheControl));
			}
		}
```

### 1、webjars资源

上面的这段代码提到的 `webjars` 是以 jar 包引入的静态资源。官网：http：//www.webjars.org

**在上面映射了一个规则：所有 /webjars/ ，都会去 classpath:/META-INF/resources/webjars/ 找资源。** 举个例子，假如项目中需要使用 jq ：

1.在 `webjars` 官网上找到 jq 对应的依赖：

![](images\1554097538107.png)

2.将依赖添加到 `pom.xml` 中

好了，已经能使用了。这里就验证一下上面提到的映射规则，在 `External Libraries` 中找到 `webjars` 依赖：

![1554098346342](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554098346342.png)

然后启动项目，直接通过浏览器访问这个资源以测试结果：

![1554098671395](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554098671395.png)

### 2、静态资源

结果认证了上面的规则是对的，`webjars` 的资源默认就是这样找的。下来再来看我们自己的静态资源：

![1554099097856](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099097856.png)

通过这行代码找到一个变量 `statciPathPattern` ：

![1554099004060](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099004060.png)

通过下面这行代码找到一个常量：

![1554099146701](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099146701.png)

![1554099173798](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099173798.png)

由上面的两行代码，可以总结：

访问当前项目的静态资源，都去上面提到的四个路径中找映射，这就是我们静态资源的映射规则：

```
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
```

### 3、欢迎页面

对于首页（欢迎页）也有默认的配置，同样的，在 `WebMvcAutoConfiguration` 类中找到映射处理器 `welcomePageHandlerMapping` ：

![1554099819578](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099819578.png)

进入 `getWelcomePage` 方法：

![1554099907071](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554099907071.png)

`location` 就是上面那个四个静态资源的位置，也就是说， `Spring Boot` 会在上面提到的那四个位置下找一个名为 `index.html` 的页面。下面就来验证这是说法。

假如就在 `classpath:/resources/` 下面定义一个 `index.html` （随便一个h2来个welcome)，然后通过浏览器访问：

![1554100187476](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554100187476.png)

结果是对的（差点吓到了，戏真多~~）。但是这里需要注意所谓的 `classpath` 类路径：

![1554100274218](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554100274218.png)

这个是类路径（虽然也叫 resources），上面的 `classpath:/resources/` 就会看到上图这样的结构。

### 4、小图标

最后说一个小图标，在 `WebMvcConfiguration` 类中有小图标资源的处理器（一个内部类）：

```java
		@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true) 
		public static class FaviconConfiguration implements ResourceLoaderAware {

			private final ResourceProperties resourceProperties;

			private ResourceLoader resourceLoader;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Override
			public void setResourceLoader(ResourceLoader resourceLoader) {
				this.resourceLoader = resourceLoader;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler.setLocations(resolveFaviconLocations());
				return requestHandler;
			}

			private List<Resource> resolveFaviconLocations() {
				String[] staticLocations = getResourceLocations(
						this.resourceProperties.getStaticLocations());
				List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
				Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
						.forEach(locations::add);
				locations.add(new ClassPathResource("/"));
				return Collections.unmodifiableList(locations);
			}

		}
```

同样的，会在静态资源文件夹（就是那四个路径）下找一个名为 `favicon.ico` 的资源，在`resources` 路径下添加了自己的一个头像试试：

![1554101301852](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554101301852.png)

### 5、总结

帅气，其实两句话总结上面的内容就是：

**1.对于 `webjars` 资源， /webjars/ ，都会去 classpath:/META-INF/resources/webjars/ 下找资源。**

**2.对于静态资源，/ ，都会去静态资源文件夹下寻找：**

```
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
```

