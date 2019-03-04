---
title: Fescar在SpringCloud中事务传播源码分析
author: [Ywind](https://icoding.live/) 
date: 2019-02-25 19:05:46
keywords: 
- Java
- Spring Cloud
- Distributed Transaction
- Fescar
---

最近一直比较关注分布式事务相关的内容，恰好阿里开源了GTS的开源实现 [Fescar](https://github.com/alibaba/fescar/) 。阿里的背书是Java届非常认可的，所以项目本身短短时间便收到了将近6k的star ⭐。抱着学习的心态对它的原理进行了解，同时因为现在的微服务框架Spring Cloud大行其道，所以分析下Fescar 整合Spring Cloud部分的源码机制。 

<!-- more -->

### Fescar简要介绍

常见的分布式事务方式有基于2PC的XA (e.g. atomikos)，从业务层入手的TCC( e.g. byteTCC)、事务消息( e.g. RocketMQ Half Message)等等。XA是需要本地数据库支持的分布式事务的协议，资源锁在数据库层面导致性能较差，而支付宝作为布道师引入的TCC模式需要大量的业务代码保证，开发维护成本较高。

分布式事务是业界比较关注的领域，这也是短短时间Fescar能收获6k Star的原因之一。Fescar 名字取自 **Fast & EaSy Commit And Rollback** ，简单来说Fescar通过对本地RDBMS分支事务的协调来完成全局事务的推进，是工作应用层的中间件。优点明显，相对于XA模式是性能较好，相对于TCC等方式开发成本较低。

类似于XA，Fescar将角色分为TC、RM、TM，事务整体过程模型如下：

![Fescar事务过程](https://camo.githubusercontent.com/6a31d57b4496e83af1a0bd76ede83657de2d9d34/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f343432303736372d343934613534643430326232643335342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

```
1. TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID。
2. XID 在微服务调用链路的上下文中传播。
3. RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖。
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议。
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。
```

其中TC是单独的进程，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚。TM RM则与应用程序工作在同一JVM进程。RM对JDBC数据源采用代理的方式对底层数据库做管理，利用语法解析，在执行事务保留快照，并生成undo log。大概的流程和模型划分就介绍到这里，下面开始对Fescar事务传播机制的分析。

### Fescar事务传播机制

Fescar事务是怎么在微服务调用链中传播的呢？从前一节的事务流程我们可以看到，当开启一个全局事务的时候，TC会创建一个全局唯一的XID，XID 是一个全局事务的唯一标识。如果想要当前的分支事务纳入全局事务的管理，只需要将该XID绑定到当前的事务上下文中即可。我们根据不同的服务框架机制，将XID在链路中传递即可实现事务的传播。

RPC请求过程分为调用方与被调用方两部分，XID在请求与响应时做处理即可。调用方即请求方将当前事务上下文中的XID取出，利用RPC协议传递给被调用方，被调用方从请求中的将XID取出，并绑定到自己的事务上下文中，纳入全局事务。微服务框架一般都有相应的Filter机制，那么Spring Cloud用的是哪个机制呢，我们来分析下Spring Cloud与Fescar的整合过程。Code： [spring-cloud-alibaba-fescar](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-fescar)

#### 微服务被调用方

由于调用方的逻辑比较多一点，我们先分析被调用方的逻辑。针对于Spring Cloud项目，默认采用的RPC传输协议时HTTP协议，所以Fescar利用了HandlerInterceptor机制来对HTTP的请求做拦截。

HandlerInterceptor 是Spring提供的接口， 它有以下三个方法可以被覆写。

```java
    /**
	 * Intercept the execution of a handler. Called after HandlerMapping determined
	 * an appropriate handler object, but before HandlerAdapter invokes the handler.
	 */
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	/**
	 * Intercept the execution of a handler. Called after HandlerAdapter actually
	 * invoked the handler, but before the DispatcherServlet renders the view.
	 * Can expose additional model objects to the view via the given ModelAndView.
	 */
	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	/**
	 * Callback after completion of request processing, that is, after rendering
	 * the view. Will be called on any outcome of handler execution, thus allows
	 * for proper resource cleanup.
	 */
	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
```

根据注释，我们可以很明确的看到各个方法的作用时间和常用用途。对于Fescar来讲，它重写了preHandle、afterCompletion方法。

FescarHandlerInterceptor 的作用是将Fescar 传递过来的 XID，绑定到本地事务的上下文中，并且在请求完成后清理相关资源。Fescar在FescarHandlerInterceptorConfiguration中配置了所有的url均进行拦截，对所有的请求过来均会执行该拦截器，进行XID的转换与事务绑定。

```java
/**
 * @author xiaojing
 *
 * Fescar HandlerInterceptor, Convert Fescar information into
 * @see com.alibaba.fescar.core.context.RootContext from http request's header in
 * {@link org.springframework.web.servlet.HandlerInterceptor#preHandle(HttpServletRequest , HttpServletResponse , Object )},
 * And clean up Fescar information after servlet method invocation in
 * {@link org.springframework.web.servlet.HandlerInterceptor#afterCompletion(HttpServletRequest, HttpServletResponse, Object, Exception)}
 */
public class FescarHandlerInterceptor implements HandlerInterceptor {

	private static final Logger log = LoggerFactory
			.getLogger(FescarHandlerInterceptor.class);

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
			Object handler) throws Exception {

		String xid = RootContext.getXID();
		String rpcXid = request.getHeader(RootContext.KEY_XID);
		if (log.isDebugEnabled()) {
			log.debug("xid in RootContext {} xid in RpcContext {}", xid, rpcXid);
		}

		if (xid == null && rpcXid != null) {
			RootContext.bind(rpcXid);
			if (log.isDebugEnabled()) {
				log.debug("bind {} to RootContext", rpcXid);
			}
		}
		return true;
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
			Object handler, Exception e) throws Exception {

		String rpcXid = request.getHeader(RootContext.KEY_XID);

		if (StringUtils.isEmpty(rpcXid)) {
			return;
		}

		String unbindXid = RootContext.unbind();
		if (log.isDebugEnabled()) {
			log.debug("unbind {} from RootContext", unbindXid);
		}
		if (!rpcXid.equalsIgnoreCase(unbindXid)) {
			log.warn("xid in change during RPC from {} to {}", rpcXid, unbindXid);
			if (unbindXid != null) {
				RootContext.bind(unbindXid);
				log.warn("bind {} back to RootContext", unbindXid);
			}
		}
	}

}

```



preHandle在handler执行前被调用，xid为当前事务上下文已经绑定的全局事务id，rpcXid为请求通过HTTP Header传递过来的全局事务id。Fescar 在preHandle方法中判断如果当前事务上下文中没有XID，且rpcXid 不为空，那么就将rpcXid绑定到当前的事务上下文。这里的处理逻辑我理解这里也对应到了Fescar文档提到的传播机制：

```
对应到 Java EE 规范和 Spring 定义的事务传播属性，Fescar 的支持如下：

PROPAGATION_REQUIRED： 默认支持
PROPAGATION_SUPPORTS： 默认支持
PROPAGATION_MANDATORY：应用通过 API 来实现
PROPAGATION_REQUIRES_NEW：应用通过 API 来实现
PROPAGATION_NOT_SUPPORTED：应用通过 API 来实现
PROPAGATION_NEVER：应用通过 API 来实现
PROPAGATION_NESTED：不支持
```

afterCompletion在handler完成后被调用，该方法用来执行资源的清理动作。Fescar 通过RootContext.unbind() 方法对事务上下文涉及到的XID进行解绑。下面if中的逻辑是为了代码的健壮性考虑，如果遇到rpcXid和 unbindXid 不相等的情况，再将unbindXid反绑回去。

对于Spring Cloud来讲，默认采用的RPC方式是HTTP的方式，所以对被调用方来讲，它的请求拦截方式不用做任何区分，只需要从Header中将XID就可以取出绑定到自己的事务上下文中即可。但是对于调用方由于请求组件的多样化，包括熔断隔离机制，所以要区分不同的情况做处理，我们来分析一下。

#### 微服务调用方

被调用方的机制非常明确简单，调用相对来讲就要复杂一点了。Fescar目前的代码将请求的情况分为RestTemplate、Feign、Feign+Hystrix，不同的组件通过Spring Boot的Auto Configuration来完成自动的配置，具体的配置类可以看spring.factories，下文也会基本上走一遍相关的配置类。

#####  RestTemplate

先来看下如果调用方如果是是基于RestTemplate的请求，Fescar是怎么传递XID的。

```java
public class FescarRestTemplateInterceptor implements ClientHttpRequestInterceptor {
	@Override
	public ClientHttpResponse intercept(HttpRequest httpRequest, byte[] bytes,
			ClientHttpRequestExecution clientHttpRequestExecution) throws IOException {
		HttpRequestWrapper requestWrapper = new HttpRequestWrapper(httpRequest);

		String xid = RootContext.getXID();

		if (!StringUtils.isEmpty(xid)) {
			requestWrapper.getHeaders().add(RootContext.KEY_XID, xid);
		}
		return clientHttpRequestExecution.execute(requestWrapper, bytes);
	}
}
```

FescarRestTemplateInterceptor实现了ClientHttpRequestInterceptor接口的 intercept 方法，对调用的请求做了包装，在发送请求时将当前事务上下文的XID取出，放到了HTTP Header中。

FescarRestTemplateInterceptor配套了一个FescarRestTemplateAutoConfiguration，用于实现将FescarRestTemplateInterceptor配置到RestTemplate中去。

```java
@Configuration
public class FescarRestTemplateAutoConfiguration {

	@Bean
	public FescarRestTemplateInterceptor fescarRestTemplateInterceptor() {
		return new FescarRestTemplateInterceptor();
	}

	@Autowired(required = false)
	private Collection<RestTemplate> restTemplates;

	@Autowired
	private FescarRestTemplateInterceptor fescarRestTemplateInterceptor;

	@PostConstruct
	public void init() {
		if (this.restTemplates != null) {
			for (RestTemplate restTemplate : restTemplates) {
				List<ClientHttpRequestInterceptor> interceptors = new ArrayList<ClientHttpRequestInterceptor>(
						restTemplate.getInterceptors());
				interceptors.add(this.fescarRestTemplateInterceptor);
				restTemplate.setInterceptors(interceptors);
			}
		}
	}

}
```

init方法遍历所有的restTemplate，并将原来restTemplate钟的拦截器取出，增加fescarRestTemplateInterceptor后置入。

##### Feign

那么再来看下Feign的相关代码，该包下面的类还是比较多的，先从AutoConfiguration入手

```java
@Configuration
@ConditionalOnClass(Client.class)
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FescarFeignClientAutoConfiguration {

	@Bean
	@Scope("prototype")
	@ConditionalOnClass(name = "com.netflix.hystrix.HystrixCommand")
	@ConditionalOnProperty(name = "feign.hystrix.enabled", havingValue = "true")
	Feign.Builder feignHystrixBuilder(BeanFactory beanFactory) {
		return FescarHystrixFeignBuilder.builder(beanFactory);
	}

	@Bean
	@Scope("prototype")
	@ConditionalOnClass(name = "com.alibaba.csp.sentinel.SphU")
	@ConditionalOnProperty(name = "feign.sentinel.enabled", havingValue = "true")
	Feign.Builder feignSentinelBuilder(BeanFactory beanFactory) {
		return FescarSentinelFeignBuilder.builder(beanFactory);
	}

	@Bean
	@ConditionalOnMissingBean
	@Scope("prototype")
	Feign.Builder feignBuilder(BeanFactory beanFactory) {
		return FescarFeignBuilder.builder(beanFactory);
	}

	@Configuration
	protected static class FeignBeanPostProcessorConfiguration {

		@Bean
		FescarBeanPostProcessor fescarBeanPostProcessor(
				FescarFeignObjectWrapper fescarFeignObjectWrapper) {
			return new FescarBeanPostProcessor(fescarFeignObjectWrapper);
		}

		@Bean
		FescarContextBeanPostProcessor fescarContextBeanPostProcessor(
				BeanFactory beanFactory) {
			return new FescarContextBeanPostProcessor(beanFactory);
		}

		@Bean
		FescarFeignObjectWrapper fescarFeignObjectWrapper(BeanFactory beanFactory) {
			return new FescarFeignObjectWrapper(beanFactory);
		}
	}

}
```

FescarFeignClientAutoConfiguration在存在Client.class时生效，且要求作用在FeignAutoConfiguration之前。且由于FeignClientsConfiguration.class是在FeignAutoConfiguration生成FeignContext生效的，所以根据依赖关系，FescarFeignClientAutoConfiguration同样早于FeignClientsConfiguration。

FescarFeignClientAutoConfiguration自定义了Feign.Builder，针对于feign.sentinel feign.hystrix 以及feign的情况做了适配，目的是自定义Feign中Client的实现为 FescarFeignClient。

```java
HystrixFeign.builder().retryer(Retryer.NEVER_RETRY)
      .client(new FescarFeignClient(beanFactory))
```

FescarFeignClient是对原来的Feign客户端代理增强：

```java
public class FescarFeignClient implements Client {

	private final Client delegate;
	private final BeanFactory beanFactory;

	FescarFeignClient(BeanFactory beanFactory) {
		this.beanFactory = beanFactory;
		this.delegate = new Client.Default(null, null);
	}

	FescarFeignClient(BeanFactory beanFactory, Client delegate) {
		this.delegate = delegate;
		this.beanFactory = beanFactory;
	}

	@Override
	public Response execute(Request request, Request.Options options) throws IOException {

		Request modifiedRequest = getModifyRequest(request);

		try {
			return this.delegate.execute(modifiedRequest, options);
		}
		finally {

		}
	}

	private Request getModifyRequest(Request request) {

		String xid = RootContext.getXID();

		if (StringUtils.isEmpty(xid)) {
			return request;
		}

		Map<String, Collection<String>> headers = new HashMap<>();
		headers.putAll(request.headers());

		List<String> fescarXid = new ArrayList<>();
		fescarXid.add(xid);
		headers.put(RootContext.KEY_XID, fescarXid);

		return Request.create(request.method(), request.url(), headers, request.body(),
				request.charset());
	}


```

方法相当明了，Feign的请求包装对应于被调用方的inteceptor的解析过程。Interceptor是从HTTP Header中将XID取出，那么Feign就是反向操作。上面的过程中我们可以看到，Fescar对原来的Request做了修改，它首先将XID从当前的事务上下文中取出，如果XID不为空的情况下，将XID放到了Header中。

FeignBeanPostProcessorConfiguration定义了3个bean，FescarContextBeanPostProcessor FescarBeanPostProcessor FescarFeignObjectWrapper。FescarContextBeanPostProcessor FescarBeanPostProcessor 实现了Spring BeanPostProcessor接口

```java
public interface BeanPostProcessor {

   @Nullable
   default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

   @Nullable
   default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

}
```

BeanPostProcessor的两个方法可以对Spring容器中的Bean做预处理，postProcessBeforeInitialization处理时机是初始化之前，postProcessAfterInitialization的处理时机是初始化之后，这2个方法的返回值可以是原先生成的实例bean，或者使用wrapper包装后的实例。

FescarContextBeanPostProcessor  将FeignContext 包装成 FescarFeignContext，FescarBeanPostProcessor  将FeignClient包装成FescarLoadBalancerFeignClient 返回。

FeignAutoConfiguration中的FeignContext并没有加ConditionalOnXXX的条件，所以Fescar采用预置处理的方式将FeignContext包装成FescarFeignContext。

```java
    @Bean
	public FeignContext feignContext() {
		FeignContext context = new FeignContext();
		context.setConfigurations(this.configurations);
		return context;
	}
```

而对于Feign Client，FeignClientFactoryBean中会获取FeignContext的实例对象。对于开发者采用@Configuration 注解的自定义配置的Feign Client对象，这里会被配置到builder，导致FescarFeignBuilder中增强后的FescarFeignCliet失效。FeignClientFactoryBean中关键代码如下：

```java
	/**
	 * @param <T> the target type of the Feign client
	 * @return a {@link Feign} client created with the specified data and the context information
	 */
	<T> T getTarget() {
		FeignContext context = applicationContext.getBean(FeignContext.class);
		Feign.Builder builder = feign(context);

		if (!StringUtils.hasText(this.url)) {
			if (!this.name.startsWith("http")) {
				url = "http://" + this.name;
			}
			else {
				url = this.name;
			}
			url += cleanPath();
			return (T) loadBalance(builder, context, new HardCodedTarget<>(this.type,
					this.name, url));
		}
		if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
			this.url = "http://" + this.url;
		}
		String url = this.url + cleanPath();
		Client client = getOptional(context, Client.class);
		if (client != null) {
			if (client instanceof LoadBalancerFeignClient) {
				// not load balancing because we have a url,
				// but ribbon is on the classpath, so unwrap
				client = ((LoadBalancerFeignClient)client).getDelegate();
			}
			builder.client(client);
		}
		Targeter targeter = get(context, Targeter.class);
		return (T) targeter.target(this, builder, context, new HardCodedTarget<>(
				this.type, this.name, url));
	}
```

FescarContextBeanPostProcessor 的存在，即使开发者对FeignClient自定义操作，依旧可以完成Fescar所需的全局事务的增强。

对于FescarFeignObjectWrapper，我们重点关注下Wrapper方法：

```java
	Object wrap(Object bean) {
		if (bean instanceof Client && !(bean instanceof FescarFeignClient)) {
			if (bean instanceof LoadBalancerFeignClient) {
				LoadBalancerFeignClient client = ((LoadBalancerFeignClient) bean);
				return new FescarLoadBalancerFeignClient(client.getDelegate(), factory(),
						clientFactory(), this.beanFactory);
			}
			return new FescarFeignClient(this.beanFactory, (Client) bean);
		}
		return bean;
	}
```

Wrapper方法中，如果bean是LoadBalancerFeignClient的实例对象，那么首先通过client.getDelegate()方法将LoadBalancerFeignClient代理的实际Client对象取出后包装成FescarFeignClient，再生成LoadBalancerFeignClient的子类FescarLoadBalancerFeignClient对象。如果bean是Client的实例对象且不是FescarFeignClient LoadBalancerFeignClient，那么bean会直接包装生成 FescarFeignClient。

上面的流程设计还是比较巧妙的，首先根据Spring boot的Auto Configuration控制了配置的先后顺序，同时自定义了Feign Builder的Bean，保证了Client均是经过增强后的FescarFeignClient 。再通过 BeanPostProcessor 对Spring 容器中的Bean做了一遍包装，保证容器内的Bean均是增强后FescarFeignClient ，避免FeignClientFactoryBean getTarget方法的替换动作。

##### Hystrix

下面我们再来看下Hystrix部分，为什么要单独把Hystrix拆出来看呢，而且Fescar代码也单独实现了个策略类。目前事务上下文RootContext的默认实现是基于ThreadLocal方式的ThreadLocalContextCore，也就是上下文其实是和线程绑定的。Hystrix本身有两种隔离状态的模式，基于信号量或者基于线程池进行隔离。Hystrix官方建议是采取线程池的方式来充分隔离，也是一般情况下在采用的模式：

```
Thread or Semaphore
The default, and the recommended setting, is to run HystrixCommands using thread isolation (THREAD) and HystrixObservableCommands using semaphore isolation (SEMAPHORE).

Commands executed in threads have an extra layer of protection against latencies beyond what network timeouts can offer.

Generally the only time you should use semaphore isolation for HystrixCommands is when the call is so high volume (hundreds per second, per instance) that the overhead of separate threads is too high; this typically only applies to non-network calls.
```

service层的业务代码和请求发出的线程肯定不是同一个，那么threadLocal的方式就没办法将XID传递给Hystrix的线程并传递给被调用方的。怎么处理这件事情呢，Hystrix提供了个机制让开发者去自定义并发策略，只需要 继承 HystrixConcurrencyStrategy重写wrapCallable方法即可。

```java
public class FescarHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {

	private HystrixConcurrencyStrategy delegate;

	public FescarHystrixConcurrencyStrategy() {
		this.delegate = HystrixPlugins.getInstance().getConcurrencyStrategy();
		HystrixPlugins.reset();
		HystrixPlugins.getInstance().registerConcurrencyStrategy(this);
	}

	@Override
	public <K> Callable<K> wrapCallable(Callable<K> c) {
		if (c instanceof FescarContextCallable) {
			return c;
		}

		Callable<K> wrappedCallable;
		if (this.delegate != null) {
			wrappedCallable = this.delegate.wrapCallable(c);
		}
		else {
			wrappedCallable = c;
		}
		if (wrappedCallable instanceof FescarContextCallable) {
			return wrappedCallable;
		}

		return new FescarContextCallable<>(wrappedCallable);
	}

	private static class FescarContextCallable<K> implements Callable<K> {

		private final Callable<K> actual;
		private final String xid;

		FescarContextCallable(Callable<K> actual) {
			this.actual = actual;
			this.xid = RootContext.getXID();
		}

		@Override
		public K call() throws Exception {
			try {
				RootContext.bind(xid);
				return actual.call();
			}
			finally {
				RootContext.unbind();
			}
		}

	}
}
```

Fescar也提供一个FescarHystrixAutoConfiguration，在存在HystrixCommand的时候生成FescarHystrixConcurrencyStrategy

```java
@Configuration
@ConditionalOnClass(HystrixCommand.class)
public class FescarHystrixAutoConfiguration {

	@Bean
	FescarHystrixConcurrencyStrategy fescarHystrixConcurrencyStrategy() {
		return new FescarHystrixConcurrencyStrategy();
	}

}
```



### 结语

本文只是介绍了下Fescar在Spring Cloud事务传递过程，更多更详细的内容请参考官方文档。总的来说，如果需要将Fescar融合到另外的三方RPC框架中，只需要找到对应的机制来传递全局事务的XID，将XID绑定在本地分支事务中即可。这两天对Fescar的了解下来，我的理解Fescar是一个工作在应用层，需要底层本地事务型数据库支撑，并对事务隔离级别等事务特性做了权衡的分布式事务中间件。

个人能力有限，不对的地方烦请指正。

### 参考文献

- [Fescar官方概览](https://github.com/alibaba/fescar/wiki/%E6%A6%82%E8%A7%88)

- [spring-cloud-openfeign原理分析](http://techblog.ppdai.com/2018/05/28/20180528/)

  

