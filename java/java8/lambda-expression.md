Lambdas表达式
-----
 Java 8 用Lambdas表达式来支持函数式编程。Lambda表达式可以用来支持高阶函数（高阶函数是指将函数作为参数或者返回结果的函数）。

下面是一个`Comparator<String>`类型Lambda表达式的例子：

```java
List<String> names = Arrays.asList("shekhar", "rahul", "sameer");
Collections.sort(names, (first, second) -> first.length() - second.length());//按照名称name长度排序
```

* `(first, second)`是 `Comparator`中`compare`方法的两个参数。
* `first.length() - second.length()` 是比较两个name的函数体。
* `->` 是把参数和函数体分开的Lambda操作符。

还是先讲一下Lambda表达式吧。

## Lambda的历史

Lambda表达式源于Lambda演算。Lambda演算说白了是一种计算的表示方法，它是函数式编程的基础，高阶函数的概念也源于Lambda演算。

Lambda演算表示如下：

```
<expression> := <variable> | <function>| <application>
```

* **variable** -- 变量
* **function** -- 函数定义，例如`λx.x*x` 表示一个数的平方。
* **application** -- 这是真正用于计算的参数。例如我们计算10^2的时候Lambda演算为`(λx.x*x) 10`，而`(λx.x*x) (λz.z+10)`表示`λz.(z+10)*(z+10)`。

了解了Lambda演算及对函数式编程语言的影响后，我们再回到Java 8 中。

## Passing behavior before Java 8

在Java 8 之前只能使用匿名类来传递行为。下面的例子展示了在用户注册后新启动一个线程发送邮件：

```java
sendEmail(new Runnable() {
            @Override
            public void run() {
                System.out.println("Sending email...");
            }
        });
```

`sendEmail` 的方法签名：

```java
public static void sendEmail(Runnable runnable)
```

写匿名类会有很多没用的代码，这写没用的代码会混淆编程人员的真实意图。


## Java 8 Lambda 表达式

上面的例子在Java 8 的Lambda表达式中这么写：

```java
sendEmail(() -> System.out.println("Sending email..."));
```

这种写法就很简洁了，并且代码的意图也很明显。`()`表示这个lambda表达式不需要参数，`->`是参数和函数体之间的分隔符。

下面看一下`Comparator` 中的 `sort` 方法

```java
Comparator<String> comparator = (first, second) -> first.length() - second.length();
```

方法签名：

```java
int compare(T o1, T o2);
```
在Lambda表达式中不需要显式的指定类型 -- String。Java编译器会从上下文中推断出类型信息。Java 8 中将类型推断系统改进得更加健壮来支持Lambda表达式。

> 多数情况下，javac可以正确的推断出类型。如果不能再上下文中推断出类型的话代码将无法编译。下面的例子中去掉了`String`就无法通过编译。

```java
Comparator comparator = (first, second) -> first.length() - second.length(); // compilation error - Cannot resolve method 'length()'
```

## Lambda表达式在 Java 8中是怎么实现的？

不是任意的接口都可以使用Lambda表达式。**只有一个抽象方法的接口才能使用Lambda表达式**。这种接口被称为**功能接口**，可以加上`@FunctionalInterface`注解。`Runnable`接口就是一个功能接口。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`@FunctionalInterface` 可以用来标识功能接口，但不是必须的。 如果被标记为`@FunctionalInterface`的接口有多个抽象方法的话，编译的时候会报***Multiple non-overriding abstract methods found*** 的错误；如果标记在没有任何方法的接口（marker interface）上，将返回***No target method found*** 的错误

那么，***lambda 表达式只是Java 8 中提供的一个匿名内部类的语法糖么？它是怎么翻译成字节码的？***

其实不是的，java中不使用匿名内部类的原因有以下两个：



1. **性能影响**: 如果使用匿名类，那么的编译的时候就得生成一个class文件，会拖慢JVM的启动速度。

2. **未来的扩展性**: 使用匿名类以后扩展起来比较困难。

### 使用动态方法调用

在`javac`编译时，将Lambda表达式生成动态调用点（称为Lambda工厂）。在调用动态调用点时，将lambda转换成一个功能接口实例。下面是`Collections.sort`生成的字节码，注意第23行。
```
public static void main(java.lang.String[]);
    Code:
       0: iconst_3
       1: anewarray     #2                  // class java/lang/String
       4: dup
       5: iconst_0
       6: ldc           #3                  // String shekhar
       8: aastore
       9: dup
      10: iconst_1
      11: ldc           #4                  // String rahul
      13: aastore
      14: dup
      15: iconst_2
      16: ldc           #5                  // String sameer
      18: aastore
      19: invokestatic  #6                  // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
      22: astore_1
      23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;
      28: astore_2
      29: aload_1
      30: aload_2
      31: invokestatic  #8                  // Method java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
      34: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      37: aload_1
      38: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      41: return
}
```
然后将lambda表达式中的方法体转换成函数，通过动态调用方法来调用。这一步中JVM虚拟机可以自行选择策略。

## 选择lambda表达式还是匿名类？

下面列出了几点区别：

1. 在匿名类中`this`指的是匿名类自身，而Lambda表达式中`this`指包含lambda表达式的类。
2. 在匿名类中可以使用包裹类的变量，而在Lambda表达式中不行。
3. Lambda表达式的类型是上下文决定的，二匿名类的类型是在创建匿名类的时候明确指定的。

## 功能接口要自己写么?

Java 8 提供了一些功能接口，他们在`java.util.function` 包中。

### java.util.function.Predicate<T>

`Predicate`接口用来进行判断。接口中包含一个`test`方法，返回一个boolean值。下面是一个判断字符串是否以 **s** 开头的例子：

```java
Predicate<String> namesStartingWithS = name -> name.startsWith("s");
```

### java.util.function.Consumer<T>

没有返回结果的功能接口。接口中有已 **T** 作为参数，没有返回值的`accept`方法。下面是例子：

```java
Consumer<String> messageConsumer = message -> System.out.println(message);
```

### java.util.function.Function<T,R>

一个参数一个返回值的功能接口，里面有个`accept`方法，输入类型T，返回类型R。

```java
Function<String, String> toUpperCase = name -> name.toUpperCase();
```

### java.util.function.Supplier<T>

没有输入只有输出。看例子吧：

```java
Supplier<String> uuidGenerator= () -> UUID.randomUUID().toString();
```

## 方法引用

可以认为是Lambda表达式的一种简写。如

`Function<String, Integer> strToLength = str ->
str.length();`

可以简写成

`Function<String, Integer>
strToLength = String::length;`

前面的`String`是目标引用，`::` 是分隔符，`length`是目标引用上调用的方法。

### 静态方法引用

`Function<List<Integer>, Integer> maxFn = (numbers) ->
Collections.max(numbers);`

### 实例方法引用

`BiFunction<String,
String, String> concatFn = String::concat;
"shekhar".concat("gulati")`.

[![Analytics](https://ga-beacon.appspot.com/UA-89268287-1/Zhaolinq/blog/blob/master/java/java8/lambda-expression.md?pixel)](https://github.com/igrigorik/ga-beacon)