## 开始

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
		
        // 主要是创建beanFactory，同时加载配置文件.xml中的beanDefinition  
        // 此处比较重要会在《Spring源码分析（三）- obtainFreshBeanFactory》中单独讲解
        // Tell the subclass to refresh the internal bean factory.
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



