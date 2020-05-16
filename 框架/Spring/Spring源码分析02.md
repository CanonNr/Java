## Spring 源码分析（二）-  refresh

### 2.1 ClassPathXmlApplicationContext

`ClassPathXmlApplicationContext` 作为整个 `Spring` 最先执行的文件所以首先介绍它了。

虽然是排头兵但是并没有太多的操作，主要是实现了各种构造方法。

```java
// ClassPathXmlApplicationContext
// 定义了多种不同的构造方法
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
    this(new String[] {configLocation}, true, null);
}

public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
    this(configLocations, true, null);
}


// 不管执行哪一个构造方法最后都会执行
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    // 主要对传过来的 xml 文件名做了非空判断,删除变量两侧多余空格 
    setConfigLocations(configLocations);
    if (refresh) {
        refresh(); // 关键方法
    }
}
```

```java
// AbstractApplicationContext => ClassPathXmlApplicationContex 的父类方法
/*
*	refresh方法主要为IOC容器Bean的生命周期管理提供条件，
*	在获取了BeanFactory之后都是在向该容器注册信息源和生命周期事件
* 	在创建IOC容器前，如果已经有容器存在，需要把已有的容器销毁和关闭，以保证在refresh()方法之后
* 	使用的是新创建的IOC容器。
*
*
*/
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 容器准备工作 刷新的方法，获取容器的当前时间，给容器设置同步标识等
        // Prepare this context for refreshing.
        prepareRefresh();
		
        // 主要是创建beanFactory
        // Tell the subclass to refresh the internal bean factory.
        // 告诉子类刷新内部bean工厂
        // 翻译官方文档有点不明觉厉,所以还是拆开来看,具体内容写在本文 2.2
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

### 2.2 ConfigurableListableBeanFactory

```java
/**
* 告诉子类刷新内部bean工厂
* Tell the subclass to refresh the internal bean factory.
* @return the fresh BeanFactory instance
* @see #refreshBeanFactory()
* @see #getBeanFactory()
*/
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 方法内部很简单 就两个方法
    // 字面意思:刷新一个 Bean 工厂
    refreshBeanFactory();
    // 返回则是一个 获取 Bean 工厂的方法
    return getBeanFactory();
}
```

### 2.3 refreshBeanFactory

```java
/**
* This implementation performs an actual refresh of this context's underlying
* bean factory, shutting down the previous bean factory (if any) and
* initializing a fresh bean factory for the next phase of the context's lifecycle.
*/
@Override
protected final void refreshBeanFactory() throws BeansException {
    // 第一步 : 判断是否有BeanFactory 详情查看本文2.4
    if (hasBeanFactory()) {
        // 如果已经存在则:,销毁容器中的Bean 关闭BeanFactory
        destroyBeans();
        closeBeanFactory();
    }
    try {
        // 创建一个内部的bean工厂  空的 没什么东西 初始化了一些配置
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        // 为 beanFactory 设置序列化
        beanFactory.setSerializationId(getId());
        // 制 beanfactory ，设置相关属性，如:启动参数、开启注解的自动装配等
        customizeBeanFactory(beanFactory);
        // 调用载入Bean定义的方法，此类只是定义了抽象方法，通过子类容器实现
        // 很关键的一个加载方法,详情查看本文2.5
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}
```

### 2.4 hasBeanFactory

```java
/**
* Determine whether this context currently holds a bean factory,
* i.e. has been refreshed at least once and not been closed yet.
* 判断是够已经有存在的bean工厂,至少refreshed了一次且美哦与关闭
*/
protected final boolean hasBeanFactory() {
    synchronized (this.beanFactoryMonitor) {
        return (this.beanFactory != null);
    }
}
```

```java
// 销毁 Bean 及关闭 BeanFactory
protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}


@Override
protected final void closeBeanFactory() {
    synchronized (this.beanFactoryMonitor) {
        if (this.beanFactory != null) {
            // 将 beanFactory 的序列化值设为 null
            this.beanFactory.setSerializationId(null);
            // beanFactory 也设为 null
            this.beanFactory = null;
        }
    }
}
```



### 2.5 loadBeanDefinitions

> :fire: 高能预警：接下来会出现多个不同的 loadBeanDefinitions 重载类 ，注意区分

```java
/**
* 所属类：AbstractXmlApplicationContext
* 通过 XmlBeanDefinitionReader 加载定义的 Bean
* Loads the bean definitions via an XmlBeanDefinitionReader.
* @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
* @see #initBeanDefinitionReader
* @see #loadBeanDefinitions
*/
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    // 为给定的BeanFactory创建一个新的 XmlBeanDefinitionReader
    // 主要是声明了一些配置
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's
    // resource loading environment.
    // 为 Bean 读取器设置Spring资源加载器
    // 只是声明没做什么处理
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    // 当Bean读取器读取Bean定义的xml资源文件时，启用xml的校验机制
    initBeanDefinitionReader(beanDefinitionReader);
    // Bean读取器真正实现加载的方法
    // 注意真正的读取方法,具体讲解在本文2.6
    loadBeanDefinitions(beanDefinitionReader);
}
```



### 2.6  loadBeanDefinitions

```java
/**
* Load the bean definitions with the given XmlBeanDefinitionReader.
* <p>The lifecycle of the bean factory is handled by the {@link #refreshBeanFactory}
* method; hence this method is just supposed to load and/or register bean definitions.
* 使用给定的 XmlBeanDefinitionReader 去定义 Bean
* Bean factory 的声明周期由 refreshBeanFactory 决定
* 所以该方法只用于加载和注册Bean
*/
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    Resource[] configResources = getConfigResources();
    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
    // 获取配置的路径
    // 注意:之前在 ClassPathXmlApplicationContext 方法里曾经有一个 setConfigLocations 方法,如下图
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        // 如果不为空则加载定义的Bean
        // 此时 configLocations 就等于 开始定义的XML文件名 => Spring.xml
        // read 之前声明的一个 XmlBeanDefinitionReader 对象
        // loadBeanDefinitions 具体做了什么在本文2.7
        reader.loadBeanDefinitions(configLocations);
    }
}
```

![1589619485972](../../../../Java%20Road/note/image/1589619485972.png)



### 2.7 loadBeanDefinitions

```java
@Override
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
    // 首先判断是不是为空
    Assert.notNull(locations, "Location array must not be null");
    // 此处定义了一个计数器
    int count = 0;
    for (String location : locations) {
        // += 加载定义的Bean 参数是 XML的文件名,盲猜是返回所有配置文件里Bean的个数
        count += loadBeanDefinitions(location);
    }
    return count;
}
```



### 2.8 loadBeanDefinitions

```java
/**
* Load bean definitions from the specified resource location.
* 在指定的资源路径中(也就是XML)中加载定义的Bean
* <p>The location can also be a location pattern, provided that the
* ResourceLoader of this bean definition reader is a ResourcePatternResolver.
* @param location the resource location, to be loaded with the ResourceLoader
* (or ResourcePatternResolver) of this bean definition reader
* @param actualResources a Set to be filled with the actual Resource objects
* that have been resolved during the loading process. May be {@code null}
* to indicate that the caller is not interested in those Resource objects.
* @return the number of bean definitions found
* ↑ 注意 @return : 定义的Bean的数量
*/
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
	// 获取一下资源加载器
    ResourceLoader resourceLoader = getResourceLoader();
    // 空验证
    if (resourceLoader == null) {
        throw new BeanDefinitionStoreException(
            "Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
    }
	
	if (resourceLoader instanceof ResourcePatternResolver) {
		// Resource pattern matching available.
		try {
			// 指定位置的Bean配置信息解析为Spring IOC容器封装的资源
			// 载多个指定位置的Bean配置信息
             // 又是一个一系列的操作 具体操作会写在下一文3.1
			Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
			int count = loadBeanDefinitions(resources);
			if (actualResources != null) {
				Collections.addAll(actualResources, resources);
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
			}
			return count;
		}
		catch (IOException ex) {
            throw new BeanDefinitionStoreException(
                "Could not resolve bean definition resource pattern [" + location + "]", ex
            );
		}
	}else {
        // Can only load single resources by absolute URL.
        Resource resource = resourceLoader.getResource(location);
        int count = loadBeanDefinitions(resource);
        if (actualResources != null) {
            actualResources.add(resource);
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
        }
        return count;
    }
}
```



