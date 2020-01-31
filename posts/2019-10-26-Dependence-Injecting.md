---
title: 'Dependency Injection'
layout: post
guid: urn:uuid:4fa00e79-263c-42c2-a1ba-d8223c5b52dc
tags:
    - programing
---

## What is [Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) 

依赖注入（DI）指的通过一个对象依赖另一个对象的一种技术方式，例如在Client对象中依赖一个Service类，我们所做的方式不是在Client对象中创建一个新的Service类或者去寻找一个Service类，而是通过把Service类当作为Client对象的一个状态或者说一个属性传递给Client对象。

依赖注入（DI）的意图是想实现程序的*解藕*（SoC Separation of Concerns），也是设计模式的一种，让程序变成相对独立的部分，每个部分完成独立的功能，保持功能单一性原则。这样能够提高代码的重用性。

依赖注入（DI）属于*Ioc*（Inversion of Control）技术中的一种，主要思想就是屏蔽Client对象对如何创建Server过程的感知，Client不需要知道怎么创建Service，相应的注入实现会代替完成这项工作。将构造Service的能力从Client中分离出来。

依赖注入中存在四种角色：

 - 被依赖的服务提供者
 - 依赖服务的客户端
 - 服务提供者的统一接口
 - 创建和传递服务的注入者

使用过Spring的开发者一定很熟悉的。

Java相关的依赖注入框架[CDI](http://cdi-spec.org/)有很多，就我所知道的框架有： [Weld](http://weld.cdi-spec.org/) 新框架Quarkus的依赖注入好像用的就是这一种框架, [Spring](https://spring.io)耳熟能详的Spring全家桶，运行时的依赖注入，所以启动的时候会很慢, [Play framework](https://www.playframework.com/)scala写的web框架

## What's the benift of DI

综上所述依赖注入（DI）主要是为了解决一下问题：

 - 如何独立出应用或类的创建方式？
 - 如何复用一个已经创建好的类？
 - 如何在单独的配置文件中指定创建对象的方式？
 - 应用程序如何支持不同的配置？

以Spring开发为例,我们可以使用 **@Autowired** 注解很快的注入一个依赖的Service类

```java
public class Client{

    @Autowried
    private ServiceA serviceA;

    public void do(){
        serviceA.doService();
    }
}
```

如果我们不使用依赖注入的方式是如何实现呢？
```java
public class Client{

    private final ServiceA serviceA=new ServiceA();

    public void do(){
        serviceA.doService();
    }

}
```
看起来好像也没有什么变化，就是使用**new ServiceA()**创建了一个对象而已。但是假如创建ServiceA的方法需要更多的构造参数呢？比如需要Dao层的支持，那么代码就会变成这样：
```java
public class Client{

    private final UserRepository userRepository;
    private final ClientRepository clientRepository;

    private final ServiceA serviceA=new ServiceA(userRepository,clientRespository);
}
```
这样一来Client就会为了创建Service类引入更多与自身无关的类，让Client变得臃肿不好维护，如果需要在另一个Client中使用这个Service那么还得将上面代码拷贝过去，又会增加重复代码的坏味道。或则说当Service类的构造方法改变了，那么所有用到它的Client类都需要修改相应的代码。

所有如果使用了依赖注入的方式实现代码逻辑会有以下好处：
 - 代码更加清晰明朗，增加可读性，因为代码都一个单独的功能
 - Client类变得更加灵活，有选择性的依赖需要的Service类
 - 依赖注入隔离类Client和Service，不仅提高了Service代码的重用性，还让代码可以更好的进行单元测试

当然有好处就有坏处的：
 - 由于创建类的工作是不可见的，各个框架会有不同的实现细节。
 - 引入额外的依赖，而且会创建很多类或接口（总得有个CDI框架吧，Spring打包出来还是挺大的）
 - 会增加启动时间（Spring就是一个典型的例子）

## DI in Spring 

在我看来，在Java中实现DI不是一件困难的事情，主要是考虑实现的DI会遇到什么问题。比如使用Spring的DI：
 - Bean是Singleton的
 - 可以检查到Cycle Dependence
 - 保证每次获取的实例都是最新的（例如在Controller中获取到的HttpSession）

以上是目前使用Spring的时候能想到的关键点。

在Spring中依赖注入的方式主要有三种：
 - 构造注入，通过反射并调用类的构造函数进行创建实例，默认使用的是无参数的构造方法。
 - Set方法注入，顾名思义就是调用Set方法将需要的值注入到类中。
 - 最常用的注解注入，主要依靠的是 JSR-250 **@Resource**｜ Spring **@Autowired** ｜ JSR-330 **@Inject**  注解。

*官方文档中说明了@Autowired默认使用的ByType进行注入，@Resource默认使用的是ByName进行注入*

```java
//map的key只能是String会被注入成Bean的名字，value是能够类型匹配上的Bean实例
//这样就能后去整个应用的所有Bean了
@Autowired
private Map<String,Object> maps;
```

构造注入与Set注入的区别：
> 构造注入是强制性的依赖，set方法注入是选择性的依赖。Spring team推荐使用的是构造注入的方式（难怪IDEA中使用@Autowired注解的时候老是提示使用构造的方式），构造注入提供工参数的null检查而且会将参数作为一个不可变的对象来使用。注意不要在构造方法中使用过多的构造参数了，会产生坏味道的。


#### 首先来看一下Spring是如何扫描类的吧。

**一下代码使用的SpringBoot 2.2版本**

1.扫描basePackage下的class文件。

2.筛选是否标记为Component的类。

**ClassPathScanningCandidateComponentProvider.java**
```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
		Set<BeanDefinition> candidates = new LinkedHashSet<>();
		try {
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + '/' + this.resourcePattern;
                    //通过basePackage获取当前路径下的class文件信息
			Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
			for (Resource resource : resources) {
				if (traceEnabled) {
					logger.trace("Scanning " + resource);
				}
				if (resource.isReadable()) {
					try {
						MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
                        //筛选class
						if (isCandidateComponent(metadataReader)) {
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							...
}

```
**PathMatchingResourcePatternResolver.java**
```java
Assert.notNull(locationPattern, "Location pattern must not be null");
		if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
			// a class path resource (multiple resources for same name possible)
			if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
				// a class path resource pattern
				return findPathMatchingResources(locationPattern);
			}
			else {
				// all class path resources with the given name
				return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
			}
		}
		else {
			// Generally only look for a pattern after a prefix here,
			// and on Tomcat only after the "*/" separator for its "war:" protocol.
			int prefixEnd = (locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 :
					locationPattern.indexOf(':') + 1);
			if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
				// a file pattern
				return findPathMatchingResources(locationPattern);
			}
			else {
				// a single resource with the given name
				return new Resource[] {getResourceLoader().getResource(locationPattern)};
			}
		}
```

2.从CachingMetadataReaderFactory获取扫描到的类信息并装配成一个Configuration（表示定义类的一些行为和依赖关系）
```java
protected final void parse(@Nullable String className, String beanName) throws IOException {
		Assert.notNull(className, "No bean class name for configuration class bean definition");
		MetadataReader reader = this.metadataReaderFactory.getMetadataReader(className);
		processConfigurationClass(new ConfigurationClass(reader, beanName));
	}
```

3.不断得调用**ConfigurationClassParser**中*doProcessConfigurationClass*方法解析定义的注解的行为，并根据注解扫描加载相关的类（执行同样的步骤）
```java
// Recursively process the configuration class and its superclass hierarchy.
		SourceClass sourceClass = asSourceClass(configClass);
		do {
			sourceClass = doProcessConfigurationClass(configClass, sourceClass);
		}
		while (sourceClass != null);

```
扫描类步骤
```java
if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
			// Recursively process any member (nested) classes first
			processMemberClasses(configClass, sourceClass);
}

		// Process any @PropertySource annotations
        ...
        // Process any @ComponentScan annotations
        ...
        // Process any @Import annotations
        ...
        // Process any @ImportResource annotations
        ...
        // Process individual @Bean methods
        ...
        // Process default methods on interfaces
        ...
        // Process superclass, if any
        ...
        // No superclass -> processing is complete
```

**以上步骤完成之后还没有正式的初始化Bean**

4.把装配好的Bean按照依赖关系排好序
```java
sortPostProcessors(currentRegistryProcessors, beanFactory);
```

5.之后会执行一些一系列的postProcess操作，其中使用到了CGLIB的字节码增强处理。
```java
public Class<?> enhance(Class<?> configClass, @Nullable ClassLoader classLoader) {
		if (EnhancedConfiguration.class.isAssignableFrom(configClass)) {
			if (logger.isDebugEnabled()) {
				logger.debug(String.format("Ignoring request to enhance %s as it has " +
						"already been enhanced. This usually indicates that more than one " +
						"ConfigurationClassPostProcessor has been registered (e.g. via " +
						"<context:annotation-config>). This is harmless, but you may " +
						"want check your configuration and remove one CCPP if possible",
						configClass.getName()));
			}
			return configClass;
		}
		Class<?> enhancedClass = createClass(newEnhancer(configClass, classLoader));
		if (logger.isTraceEnabled()) {
			logger.trace(String.format("Successfully enhanced %s; enhanced class name is: %s",
					configClass.getName(), enhancedClass.getName()));
		}
		return enhancedClass;
	}
```

6.最后在准备工作都完成之后开始初始化非懒加载的Bean。
```java
// Instantiate all remaining (non-lazy-init) singletons.
beanFactory.preInstantiateSingletons();
```
**BeanFactory**是创建Bean的主要类具体的实现是在 **AbstractBeanFactory**中的 *doGetBean*方法中

```java
// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						final FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
											((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}
```

具体实例化的时候会有不同的InstanceStrategy，如果是使用默认的构造方法（就是没有注解标记的）就会使用 *SimpleInstantiationStrategy* 反射默认的构造方法，如果是使用了注解，就会使用 *CglibSubclassingInstantiationStrategy* 生产代理类。

7.最后通知容器Bean创建完成。

## Consolation

以上就是Spring创建Bean的一个大概流程。由于源码多而杂很多细节的地方都没有仔细去看，本次主要是想知道Spring是创建Bean的一个流程，从扫描类到解析类的信息，再到将类信息转化可解释的配置类，再统一将配置类按照不同的实例化策略进行实例化，再到实例化之后的后处理操作。整个过程看起来是很简单的，但是具体实现起来就有很多的细节问题需要考虑周全，比如其中为了加快类的解析过程，使用了缓存。又比如使用了CGLIB来动态代理来实现方法注入。循环依赖的检测与处理，以及大量的反射技术。内容太多了，想要了解的透彻还得化很多时间去看源码。

说了那么多最终的收获在于了解了DI技术利弊，学了DI的编程思想。这种DI技巧并不总是在Java中才能实现的，而是每一种编程语言都有。并不是与Java的注解绑定的，注解只是简化了代码和提高了代码的可读性。其实我理解的DI最主要的思想就是让调用方不关心依赖类的创建逻辑和具体实现，调用方通过组装需要的依赖类来实现自己想要实现的逻辑。是一个高层依赖底层的一种关系。这种结构让大型多层应用能够有效的维护和开发，避免了牵一发而动全身的影响。

**后序阅读更新（2019-12-25）**

	The Spring container validates the configuration of each bean as the container is created. However, the bean properties themselves are not set until the bean is actually created. Beans that are singleton-scoped and set to be pre-instantiated (the default) are created when the container is created. Scopes are defined in Bean Scopes. Otherwise, the bean is created only when it is requested. Creation of a bean potentially causes a graph of beans to be created, as the bean’s dependencies and its dependencies' dependencies (and so on) are created and assigned. Note that resolution mismatches among those dependencies may show up late — that is, on first creation of the affected bean.
	//谷歌翻译
	在创建容器时，Spring容器会验证每个bean的配置。但是，在实际创建Bean之前，不会设置Bean属性本身。创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。范围在Bean范围中定义。否则，仅在请求时才创建Bean。创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。请注意，这些依赖项之间的分辨率不匹配可能会显示得较晚-即在首次创建受影响的bean时。


## Reference

 - [The IoC Container](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans)