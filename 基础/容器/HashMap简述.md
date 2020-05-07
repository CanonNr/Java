## 基本使用

```java
{
    HashMap<Object, Object> map = new HashMap<>();
    map.put("不知妻美","刘强东");
    map.put("悔创阿里","杰克马");
    map.put("普通家庭","马化腾");
    map.put("长相一般","孙彦祖");
    System.out.println(map);
    
    // 输出结果
	{不知妻美=刘强东, 长相一般=孙彦祖, 悔创阿里=杰克马, 普通家庭=马化腾}	
}



{
    // 获取所有Key值
    map.keySet();
    // 获取所有value值
    map.values());

    // 输出结果
	[刘强东, 孙彦祖, 杰克马, 马化腾]
	[不知妻美, 长相一般, 悔创阿里, 普通家庭]
}


```



