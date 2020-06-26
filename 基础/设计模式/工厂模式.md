# 工厂设计模式

> "Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses."
>
> 在基类中定义创建对象的一个接口，让子类决定实例化哪个类。工厂方法让一个类的实例化延迟到子类中进行。

## 分类：

![1592973411176](../../image/1592973411176.png)

- 简单工厂（Simple Factory）模式，又称静态工厂方法模式（Static Factory Method Pattern）。

- 工厂方法（Factory Method）模式，又称多态性工厂（Polymorphic Factory）模式或虚拟构造子（Virtual Constructor）模式；
- 抽象工厂（Abstract Factory）模式，又称工具箱（Kit 或Toolkit）模式。

## 优点

- **解耦** ：把对象的创建和使用的过程分开

- **降低代码重复:** 如果创建某个对象的过程都很复杂，需要一定的代码量，而且很多地方都要用到，那么就会有很多的重复代码。

- **降低维护成本** ：由于创建过程都由工厂统一管理，所以发生业务逻辑变化，不需要找到所有需要创建对象B的地方去逐个修正，只需要在工厂里修改即可，降低维护成本。

## 实例

### 简单工厂模式

```php
interface Shape
{
    public function draw();
}
```

```php
class Circle implements Shape
{
    public function draw()
    {
        echo "Draw Circle";
    }
}

class Rectangle implements Shape
{
    public function draw()
    {
        echo "Draw Rectangle";
    }
}
```

```php
class ShapeFactory
{
    public static function getShape(string $type):Shape
    {
        if ($type == 'Circle'){
            return new Circle();
        }elseif ($type == 'Rectangle'){
            return new Rectangle();
        }
        return null;
    }
}
```

```php
$shape1 = ShapeFactory::getShape('Circle');
$shape2 = ShapeFactory::getShape('Rectangle');
var_dump($shape1);
var_dump($shape2);
```



> 由于工厂类集中了所有实例的创建逻辑，违反了高内聚责任分配原则，将全部创建逻辑集中到了一个工厂类中；它所能创建的类只能是事先考虑到的，如果需要添加新的类，则就需要改变工厂类了。



![1592988548775](../../image/1592988548775.png)

### 工厂方法模式

**工厂方法模式**是在**简单工厂模式**基础上加了一个工厂接口

```java
interface Factory
{
    public function getShape();
}
```

```php
class CircleFactory implements Factory
{
    public function getShape()
    {
        return new Circle();
    }
}
```

```php
$circleFactory = new CircleFactory();
$circle = $circleFactory->getShape();
$circle->draw();
```



![1592988678064](../../image/1592988678064.png)



### 抽象工厂模式

> 假设某手机厂商有`α`和`β`两款产品，用了相同的**模具**、**供应商**和**销售渠道**，只是个别硬件配置不同。

