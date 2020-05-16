## Spring 源码分析（三）- obtainFreshBeanFactory

本文主要讲述 `obtainFreshBeanFactory` 方法及其相关类，根据方法的字面意思可以想到这个方法是用于**创建一个新的 Bean 工厂**

### 1.0 ConfigurableListableBeanFactory

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

### 1.1 refreshBeanFactory

```java
/**
* This implementation performs an actual refresh of this context's underlying
* bean factory, shutting down the previous bean factory (if any) and
* initializing a fresh bean factory for the next phase of the context's lifecycle.
*/
@Override
protected final void refreshBeanFactory() throws BeansException {
    // 第一步 : 判断是否有BeanFactory 详情查看本文1.1
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
        // 很关键的一个加载方法,详情查看本文1.3
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

### 1.2  hasBeanFactory
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



### 1.3 loadBeanDefinitions

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
    // 注意真正的读取方法,具体讲解在本文1.4
    loadBeanDefinitions(beanDefinitionReader);
}
```



### 1.4  loadBeanDefinitions

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
        // loadBeanDefinitions 具体做了什么在本文1.5
        reader.loadBeanDefinitions(configLocations);
    }
}
```

![1589619485972](../../image/1589619485972.png)



### 1.5  loadBeanDefinitions

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



### 1.6 loadBeanDefinitions

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
             // 又是一个一系列的操作,详情查看本文 1.7
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





### 1.7  getResources

```java
//---------------------------------------------------------------------
// Implementation of ResourcePatternResolver interface
// 实现 ResourcePatternResolver 接口
//---------------------------------------------------------------------

@Override
public Resource[] getResources(String locationPattern) throws IOException {
    return this.resourcePatternResolver.getResources(locationPattern);
}
```

```java
@Override
public Resource[] getResources(String locationPattern) throws IOException {
    Assert.notNull(locationPattern, "Location pattern must not be null");
    // 首先判断是不是 以 classpath*: 开头的资源路径
    if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
        // a class path resource (multiple resources for same name possible)
        // 判断:是否包含 * ? { }
        if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
            // a class path resource pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // all class path resources with the given name
            // 查找所有所有类位置资源。
            return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
        }
    }
    else {
        // Generally only look for a pattern after a prefix here,
        // and on Tomcat only after the "*/" separator for its "war:" protocol.
        // 是不是以 war: 开头
        // 如果是 则得到 资源路径 */ 首次出现位置+1  如果不存在 */ 则为0
        // 否则 : 首次出现位置 +1
        // 最终得到 前缀结尾
        int prefixEnd = (locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 :
                         locationPattern.indexOf(':') + 1);
        if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
            // a file pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // a single resource with the given name
            // 获取一个资源的对象 详情查看1.8
            return new Resource[] {getResourceLoader().getResource(locationPattern)};
        }
    }
}
```

### 1.8 getResource

```java
@Override
public Resource getResource(String location) {
    Assert.notNull(location, "Location must not be null");

    for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
        Resource resource = protocolResolver.resolve(location, this);
        if (resource != null) {
            return resource;
        }
    }
	// 这个方法就是做了一堆判断
    // location 是否 / 开头
    if (location.startsWith("/")) {
        return getResourceByPath(location);
    }
    // 是否 classpath: 开头
    else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
        return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
    }
    else {
        try {
            // Try to parse the location as a URL...
            // 如果都不是就根据 location 生成一个 URL对象
            URL url = new URL(location);
            return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
        }
        catch (MalformedURLException ex) {
            // No URL -> resolve as resource path.
            // 上面声明 URL 对象时 传入的参数如果不是 URL 则会报错
            // 所以会通过 path 获取一个资源对象 详情1.9
            return getResourceByPath(location);
        }
    }
}
```



### 1.9  getResourceByPath

```java
// Return a Resource handle for the resource at the given path.
// 返回一个 Resource 通过给定的一个资源路径
protected Resource getResourceByPath(String path) {
    // 此处也很简单,就是 new 了一个 ClassPathContextResource 对象
    // 将 path 的参数传入进去
    return new ClassPathContextResource(path, getClassLoader());
}

public ClassPathResource(String path, @Nullable ClassLoader classLoader) {
    Assert.notNull(path, "Path must not be null");
    String pathToUse = StringUtils.cleanPath(path);
    if (pathToUse.startsWith("/")) {
        pathToUse = pathToUse.substring(1);
    }
    this.path = pathToUse;
    this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
}
```





### 1.10 loadBeanDefinitions

> 🚀 在 `1.9 getResourceByPath` 我们成功获取到了一个配置文件的 `Resource` 对象
>
> 层层返回，此时我们需要回到 `1.6 loadBeanDefinitions` 中继续 加载Bean

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
            // 由 1.7 ~ 1.9 一系列操作 得到了一个 Resource 对象的数组
            Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
            // 截止到现在我们完成的仅仅是将 Spring.xml 字符串转换成了 resource 对象
            // 看下图,一个存放着 resource 对象的数组
            // loadBeanDefinitions 具体操作看本文 1.11
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

![1589649074859](../../image/1589649074859.png)



### 1.11 loadBeanDefinitions

```java
@Override
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
    Assert.notNull(resources, "Resource array must not be null");
    int count = 0;
    for (Resource resource : resources) {
        // 本方法代码不多,处理在 loadBeanDefinitions()
        count += loadBeanDefinitions(resource);
    }
    return count;
}
```

```java
/**
* Load bean definitions from the specified XML file.
* 官方文档很重要:加载一个定义的bean在指定的xml文件中
* 至于这个指定的xml文件也很清楚了,就是我们声明是的参数 - Spring.xml
*/
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    // 第一步先将 Resource 对象进行了一个编码的处理
    // 第二步才是解析 XML
    // 具体操作查看本文 1.12
    return loadBeanDefinitions(new EncodedResource(resource));
}
```





### 1.12 loadBeanDefinitions

```java
/**
* Load bean definitions from the specified XML file.
*/
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    Assert.notNull(encodedResource, "EncodedResource must not be null");
    if (logger.isTraceEnabled()) {
        logger.trace("Loading XML bean definitions from " + encodedResource);
    }
	// 通过属性记录当前已经加载的资源
    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
    // 因为什么也没做 首次肯定为 null
    if (currentResources == null) {
        // 一个初始化长度为 4 的 HashSet
        currentResources = new HashSet<>(4);
        // 将 currentResources 的值加载进去
        this.resourcesCurrentlyBeingLoaded.set(currentResources);
    }
    
    if (!currentResources.add(encodedResource)) {
        throw new BeanDefinitionStoreException(
            "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }
    try {
        // 通过 Resource 对象 获取字节流对象
        InputStream inputStream = encodedResource.getResource().getInputStream();
        try {
            // 通过字节流获取一个输入流
            InputSource inputSource = new InputSource(inputStream);
            if (encodedResource.getEncoding() != null) {
                inputSource.setEncoding(encodedResource.getEncoding());
            }
            // 看方法名:加载 定义 bean (此方法不简单)
            // 还有一个有意思的:在Spring中很多doxxxx的方法可能都是即将有实际操作的方法	
            // 具体做了什么 本文 1.13
            return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        }
        finally {
            // 不要忘记关闭输入流
            inputStream.close();
        }
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
            "IOException parsing XML document from " + encodedResource.getResource(), ex);
    }
    finally {
        currentResources.remove(encodedResource);
        if (currentResources.isEmpty()) {
            this.resourcesCurrentlyBeingLoaded.remove();
        }
    }
}
```



### 1.13  doLoadBeanDefinitions

```java
/**
* Actually load bean definitions from the specified XML file.
* 怀疑是一个划水的官方方法文档,在定义的XML文件中加载Bean对象已经说了很多次了
*/
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
    throws BeanDefinitionStoreException {
    try {
        // 首先是读取一个文档,参数有两个:输入流和资源类
        // 查看本文1.14
        Document doc = doLoadDocument(inputSource, resource);
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }
    catch (BeanDefinitionStoreException ex) {
       // 源代码中此处有很多 catch 的异常捕获,为了看得舒服我删掉了
    }
}

```



### 1.14 doLoadDocument

```java
/**
* Actually load the specified document using the configured DocumentLoader.
* 创建bean所需要的参数都是从DocumentLoader读来的
*/
protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
    // 验证偏多
    // loadDocument 方法查看本文 1.15
    return this.documentLoader.loadDocument(inputSource, getEntityResolver(),
                                            this.errorHandler,
                                            getValidationModeForResource(resource),
                                            isNamespaceAware());
}
```



```java
/**
* Determine the validation mode for the specified {@link Resource}.
* If no explicit validation mode has been configured, then the validation
* ode gets {@link #detectValidationMode detected} from the given resource.
* <p>Override this method if you would like full control over the validation
* mode, even when something other than {@link #VALIDATION_AUTO} was set.
* @see #detectValidationMode
*/
protected int getValidationModeForResource(Resource resource) {
    int validationModeToUse = getValidationMode();
    if (validationModeToUse != VALIDATION_AUTO) {
        return validationModeToUse;
    }
    // 这个是关键 
    // 检测验证模式
    int detectedMode = detectValidationMode(resource);
    if (detectedMode != VALIDATION_AUTO) {
        return detectedMode;
    }
    // Hmm, we didn't get a clear indication... Let's assume XSD,
    // since apparently no DTD declaration has been found up until
    // detection stopped (before finding the document's root tag).
    return VALIDATION_XSD;
}
```



```java
/**
* Detect which kind of validation to perform on the XML file identified by the supplied Resource
* 检测XML文件执行哪种验证由Resource决定
*  If the file has a {@code DOCTYPE} definition then DTD validation is used otherwise XSD validation is assumed.
* 如果有DOCTYPE定义则使用 DTD 否则用 XSD 验证
* <p>Override this method if you would like to customize resolution
* of the {@link #VALIDATION_AUTO} mode.
*/
protected int detectValidationMode(Resource resource) {
    if (resource.isOpen()) {
        throw new BeanDefinitionStoreException(
            "Passed-in Resource [" + resource + "] contains an open stream: " +
            "cannot determine validation mode automatically. Either pass in a Resource " +
            "that is able to create fresh streams, or explicitly specify the validationMode " +
            "on your XmlBeanDefinitionReader instance.");
    }

    InputStream inputStream;
    try {
        inputStream = resource.getInputStream();
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
            "Unable to determine validation mode for [" + resource + "]: cannot open InputStream. " +
            "Did you attempt to load directly from a SAX InputSource without specifying the " +
            "validationMode on your XmlBeanDefinitionReader instance?", ex);
    }

    try {
        return this.validationModeDetector.detectValidationMode(inputStream);
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException("Unable to determine validation mode for [" +
                                               resource + "]: an error occurred whilst reading from the InputStream.", ex);
    }
}
```



```java
/**
* Detect the validation mode for the XML document in the supplied {@link InputStream}.
* Note that the supplied {@link InputStream} is closed by this method before returning.
* @param inputStream the InputStream to parse
* @throws IOException in case of I/O failure
* @see #VALIDATION_DTD
* @see #VALIDATION_XSD
*/
public int detectValidationMode(InputStream inputStream) throws IOException {
    // Peek into the file to look for DOCTYPE.
    // 查看文件的寻找DOCTYPE
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
    try {
        
        boolean isDtdValidated = false;
        String content;
        // 下面就是验证了
        // 循环判断
        while ((content = reader.readLine()) != null) {
            content = consumeCommentTokens(content);
            if (this.inComment || !StringUtils.hasText(content)) {
                continue;
            }
            if (hasDoctype(content)) {
                isDtdValidated = true;
                break;
            }
            if (hasOpeningTag(content)) {
                // End of meaningful data...
                break;
            }
        }
        // 如果是DTD验证则返回 2  , 否则为XSD 3
        return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
    }
    catch (CharConversionException ex) {
        // Choked on some character encoding...
        // Leave the decision up to the caller.
        return VALIDATION_AUTO;
    }
    finally {
        reader.close();
    }
}
```



### 1.15 loadDocument

```java
/**
* Load the {@link Document} at the supplied {@link InputSource} using the standard JAXP-configured
* XML parser.
*/
@Override
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
                             ErrorHandler errorHandler, 
                             int validationMode, 
                             boolean namespaceAware) throws Exception {

    DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
    if (logger.isTraceEnabled()) {
        logger.trace("Using JAXP provider [" + factory.getClass().getName() + "]");
    }
    DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
    return builder.parse(inputSource);
}
```

