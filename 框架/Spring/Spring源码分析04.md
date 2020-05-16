## 注册 BeanDefinitions

在上一篇已经介绍了 `Spring` 中如何通过`doLoadDocument`方法将XML文件解析为`Document`对象。

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
    throws BeanDefinitionStoreException {
    try {
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

实现了 `Document` 不算完，后面紧跟着就来了一个`registerBeanDefinitions` 盲猜一下是 注册定义的Bean，

那么`BeanDefinition`是什么?

`BeanDefinition` 是一个接口，描述了一个 `bean` 实例，它具有属性值，构造函数参数值以及具体实现提供的更多信息。