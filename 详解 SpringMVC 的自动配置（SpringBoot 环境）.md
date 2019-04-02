## 详解 SpringMVC 的自动配置（SpringBoot 环境）

------

前面的文章中也有提到过 `Spring Boot` 中的自动配置原理，在实际开发中如果对这些默认配置不熟悉的话很难做到“灵活”，什么都只能听“人家的”，所以，在此就借 `SpringMVC` 来开刀，揭开 `Spring Boot ` 自动配置的原理。

在官网上提到 `SpringMVC` 的内容其实少的可伶，有兴趣也可以过去传送过去。

https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-auto-configuration

废话不多说，下面正式来介绍自动配置。

### 1.自动配置SpringMVC的视图解析器

`Inclusion of  ContentNegotiatingViewResolver  and BeanNameViewResolver beans.`—— 摘自官网

这里主要说的是自动配置了 `ViewResolver` （视图解析器：解析视图，说白了就是根据方法的返回值得到视图对象，视图对象决定如何再去进行下一步的处理，是转发还是重定向。）

上面提到的 `ContentNegotiatingViewResolver` 的作用就是整合所有的视图解析器（后面进行验证），如果我们需要自定义视图解析器，则直接添加到容器中即可，它会给我们自动组合起来。

分析：

所有的 web 配置都是由 `WebMvcAutoConfiguration` 这个类来完成的，所以，先从这个类下手。该类中有一个方法 `viewResolver`，方法的返回值就是上面提到的 `ContentNegotiatingViewResolver` ，还要注意该方法上有一个 `@Bean` 注解：

```java
		@Bean
		@ConditionalOnBean(ViewResolver.class)
		@ConditionalOnMissingBean(name = "viewResolver", value = ContentNegotiatingViewResolver.class)
		public ContentNegotiatingViewResolver viewResolver(BeanFactory beanFactory) {
			ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
			resolver.setContentNegotiationManager(
					beanFactory.getBean(ContentNegotiationManager.class));
			// ContentNegotiatingViewResolver uses all the other view resolvers to locate
			// a view so it should have a high precedence
			resolver.setOrder(Ordered.HIGHEST_PRECEDENCE);
			return resolver;
		}
```

所以说，不用细看这个方法的逻辑就能判断这个方法的作用就是给容器中注册一个 `ContentNegotiatingViewResolver` 。既然这玩意是一个视图解析器，那么就得解析视图，所以进入这个类，找到解析视图的方法 `resolveViewName` ：

```java
	@Override
	@Nullable
	public View resolveViewName(String viewName, Locale locale) throws Exception {
		RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
		Assert.state(attrs instanceof ServletRequestAttributes, "No current ServletRequestAttributes");
		List<MediaType> requestedMediaTypes = getMediaTypes(((ServletRequestAttributes) attrs).getRequest());
		if (requestedMediaTypes != null) {
            //获取获选视图
			List<View> candidateViews = getCandidateViews(viewName, locale, requestedMediaTypes);
            //选择适合的视图对象
			View bestView = getBestView(candidateViews, requestedMediaTypes, attrs);
			if (bestView != null) {
                //返回最适合的视图对象
				return bestView;
			}
		}

		String mediaTypeInfo = logger.isDebugEnabled() && requestedMediaTypes != null ?
				" given " + requestedMediaTypes.toString() : "";

		if (this.useNotAcceptableStatusCode) {
			if (logger.isDebugEnabled()) {
				logger.debug("Using 406 NOT_ACCEPTABLE" + mediaTypeInfo);
			}
			return NOT_ACCEPTABLE_VIEW;
		}
		else {
			logger.debug("View remains unresolved" + mediaTypeInfo);
			return null;
		}
	}
```

注意上面的几行注释，也许你会有好奇，怎么去选择适合的视图对象呢？不慌，点进去 `getCandidateViews` 方法：

```java
private List<View> getCandidateViews(String viewName, Locale locale, List<MediaType> requestedMediaTypes)
			throws Exception {

		List<View> candidateViews = new ArrayList<>();
		if (this.viewResolvers != null) {
			Assert.state(this.contentNegotiationManager != null, "No ContentNegotiationManager set");
			for (ViewResolver viewResolver : this.viewResolvers) {
				View view = viewResolver.resolveViewName(viewName, locale);
				if (view != null) {
					candidateViews.add(view);
				}
				for (MediaType requestedMediaType : requestedMediaTypes) {
					List<String> extensions = this.contentNegotiationManager.resolveFileExtensions(requestedMediaType);
					for (String extension : extensions) {
						String viewNameWithExtension = viewName + '.' + extension;
						view = viewResolver.resolveViewName(viewNameWithExtension, locale);
						if (view != null) {
							candidateViews.add(view);
						}
					}
				}
			}
		}
		if (!CollectionUtils.isEmpty(this.defaultViews)) {
			candidateViews.addAll(this.defaultViews);
		}
		return candidateViews;
	}
```

这几个意思？看到一个 `List` 容器（装候选视图的），然后遍历，没错，正是逐一去试。把所有视图解析器拿过来，挨个去解析视图，看看哪个最合适。那么这些视图解析器又是从何而来的呢？找到该类中的一个方法：

```java
protected void initServletContext(ServletContext servletContext) {
		Collection<ViewResolver> matchingBeans =
            	//利用一个工具从容器中获取所有的视图解析器
				BeanFactoryUtils.beansOfTypeIncludingAncestors(obtainApplicationContext(), ViewResolver.class).values();
		if (this.viewResolvers == null) {
			this.viewResolvers = new ArrayList<>(matchingBeans.size());
			for (ViewResolver viewResolver : matchingBeans) {
				if (this != viewResolver) {
					this.viewResolvers.add(viewResolver);
				}
			}
		}
		else {
			for (int i = 0; i < this.viewResolvers.size(); i++) {
				ViewResolver vr = this.viewResolvers.get(i);
				if (matchingBeans.contains(vr)) {
					continue;
				}
				String name = vr.getClass().getName() + i;
				obtainApplicationContext().getAutowireCapableBeanFactory().initializeBean(vr, name);
			}

		}
		AnnotationAwareOrderComparator.sort(this.viewResolvers);
		this.cnmFactoryBean.setServletContext(servletContext);
	}
```

利用一个 `BeanFactoryUtils` 工具从容器中获取所有的视图解析器。哎哟呵，既然你是从容器中拿的，那么就刺激了，**如果我们需要自定义视图解析器，只需要把我们定义的视图解析器往容器一丢，完事！** 那么事实到底是不是我们猜想的这样的呢？接下来进入验证时刻（有没有一点蒋昌建的味道）：

​	1.定义一个视图解析器

```java
	//定义一个视图解析器
    private static class MyViewResolver implements ViewResolver {

        @Override
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            return null;
        }
    }
```

​	2.丢进容器中

```java
	//丢进容器中
    @Bean
    public ViewResolver viewResolver(){
        return new MyViewResolver();
    }
```

​	3.在核心控制器 `DispatcherServlet` 中找到 `doDispatch` 方法，并在方法上打上断点，

![1554210062700](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1554210062700.png)

然后 debug 随便发起一个请求，发会看到上面的结果：咱们自定义的视图解析器就给管理了。所以，需要自定义视图解析就直接将自定义的解析器丢进容器中。

### 2. 类型转换器以及格式化器

`Automatic registration of Converter, GenericConverter, and Formatter beans` —— 摘自官网

大致意思就是自动注册了转换器和日期格式化器。套路还是一样，先从最初的 `WebMvcAutoConfiguration` 开始。然后找到 `addFormatter` 方法，接着你会发现，这玩意又是在容器中取的，套路都是一样的，这里就不作详细的分析了。直接给出结论：

**无论是转换器还是格式化器，如果需要自定制，则直接丢容器里面即可。**

`SpringMVC` 给我们配置其实还不仅仅是上面提到的这两个（对于静态资源的配置前面的文章已经提到过了），还有好一大堆的呢，如果感兴趣，可以从这 `org.springframework.boot.autoconfigure.web` 刨出来看看。

### 3. 关于SpringMVC 默认配置的原则

`Spring Boot` 会先看容器中有没有用户自定义的配置，如果有，则使用用户配置的，如果没有，才使用默认配置的；如果有些组件可以有多个（比如上面提到的 `ViewResolver` ），将用户配置的和默认配置的结合起来。