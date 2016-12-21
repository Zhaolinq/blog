# 接口的默认和静态方法

原文：
> https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/01-default-static-interface-methods.md

## 接口


接口是用来解耦的。看下面的例子：

```java
//接口
public interface Calculator {

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);
}
```

```java
//实现类
public class BasicCalculator implements Calculator {

    @Override
    public int add(int first, int second) {
        return first + second;
    }

    @Override
    public int subtract(int first, int second) {
        return first - second;
    }

    @Override
    public int divide(int number, int divisor) {
        if (divisor == 0) {
            throw new IllegalArgumentException("divisor can't be zero.");
        }
        return number / divisor;
    }

    @Override
    public int multiply(int first, int second) {
        return first * second;
    }
}
```

调用的时候这么写：

```java
Calculator calculator = new BasicCalculator();
int sum = calculator.add(1, 2);

BasicCalculator cal = new BasicCalculator();
int difference = cal.subtract(3, 2);
```

发现问题了么？**在调用的时候不仅使用了接口`Calculator`，还用到了实现类 `BasicCalculator`！** 这可怎么办？


## 静态工厂方法

先将`BasicCalculator`隐藏起来。

```java
class BasicCalculator implements Calculator {
  // rest remains same
}
```

写一个工厂方法来创建`Calculator`实例。

```java
public abstract class CalculatorFactory {

    public static Calculator getInstance() {
        return new BasicCalculator();
    }
}
```

隐藏了实现细节，真正的面向接口编程。但是在提供接口API的时候还要附加一个`CalculatorFactory`类……@_@

## 接口中声明静态方法

**Java 8 中允许在一个接口中声明静态方法来保持API的简洁**。
使用了Java 8 后接口就可以定义成这样：


```java
public interface Calculator {

    static Calculator getInstance() {
        return new BasicCalculator();
    }

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);

}
```

有没有感觉世界美好了一点？

## 接口的默认（防御）方法

慢慢的你可能发现需要在`Calculator`接口中增加一个`remainder` 方法，那么接口就变成这样了：

```java
public interface Calculator {

    static Calculator getInstance() {
        return new BasicCalculator();
    }

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);

    int remainder(int number, int divisor); // new method added to API
}
```
但是这样就破坏了代码的兼容性。之前实现了`Calculator`接口的用户必须增加`remainder`的实现方法，否则不能通过编译。这对API的设计来说是一个大问题，因为它使得API难以扩展。

为了解决这个问题，**Java 8允许在接口中定义默认的方法实现**。这些被称为默认或者防御方法。有了默认方法后，接口的实现类就不需要再实现这些方法（当然，如果实现了这些的话就会覆盖默认方法）。

如此一来，API就可以定义成这样：

```java
default int remainder(int number, int divisor) {
    return subtract(number, multiply(divisor, divide(number, divisor)));
}
```

万事大吉了？其实还没有……

## 多重继承

Java中类只能继承一个类,但可以实现多个接口。由于Java 8 中引入的默认方法，Java实际上就支持了多重继承。

下面是三个决定最终使用哪个方法的解析规则:

**No 1: 类中定义的方法优于接口中定义的方法。**

```java
interface A {
    default void doSth(){
        System.out.println("inside A");
    }
}
class App implements A{

    @Override
    public void doSth() {
        System.out.println("inside App");
    }

    public static void main(String[] args) {
        new App().doSth(); //inside App
    }
}
```

**No 2: 次之，选择更具体的接口**

```java
interface A {
    default void doSth() {
        System.out.println("inside A");
    }
}
interface B {}
interface C extends A {
    default void doSth() {
        System.out.println("inside C");
    }
}
class App implements C, B, A {

    public static void main(String[] args) {
        new App().doSth(); //inside C
    }
}
```


**No 3: 再次，必须明确指定实现**

```java
interface A {
    default void doSth() {
        System.out.println("inside A");
    }
}
interface B {
    default void doSth() {
        System.out.println("inside B");
    }
}
class App implements B, A {

    @Override
    public void doSth() {
        B.super.doSth();
    }

    public static void main(String[] args) {
      new App().doSth(); //inside B
    }
}
```

[![Analytics](https://ga-beacon.appspot.com/UA-89268287-1/Zhaolinq/blog/blob/master/java/java8/default-and-static-methods-for-interfaces.md?pixel)](https://github.com/igrigorik/ga-beacon)
