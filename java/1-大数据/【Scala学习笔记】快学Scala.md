# 【Scala学习笔记】快学Scala

## 1、基础

### 1.1、Scala解释器

Scala解释器读到一个表达式，对它进行求值，将它打印出来，接着再继续读下一个表达式。这个过程叫做“读取-求值-打印”循环（read-eval-print loop），即REPL

### 1.2、声明值和变量

声明可变的变量，使用`var`

声明不可变的变量，使用`val`，如果对象引用不可变，则优先使用`val`

一行代码可以写多句语句，语句之间用分号分割。如果不是多久语句，则不需要分号结尾。

### 1.3、常用类型

Scala有7种数值类型：Byte、Char、Short、Int、Long、Float、Double，1个Boolean类型。这些都是类，Scala中不区分基本数据类型和包装类。

```scala
1.toString() // 将交出字符串“1”
```

此外Scala还提供了 RichInt、RichDouble、RichChar等，以及BigInt、BigDecimal。

```scala
1.to(10) // 将交出 Range(1,2,3,4,5,6,7,8,9,10)
```

上述方法首先将 Int 转为 RichInt ，然后再应用 to 方法。

>在Scala中，使用方法而不是强制类型转换，来做数值类型之间的转换
>
>`99.44.toInt`

### 1.4、算术和操作符重载

算术操作符跟java，c++一样。**在Scala中所有操作符都是方法**。例如 `a + b` 为 `a.+(b)`，总结就是：

`a 方法 b` 等价为 `a.方法(b)`

Scala中没有提供`++`和`--`操作符。无法实现一个`++`方法，因为Scala中Int类是不可变的。

### 1.5、关于方法调用

**最重要的规则**：如果一个方法没有参数，调用的时候可以不写括号。

Scala中的静态方法：在单例对象中定义这些方法。

导包的时候，以`scala.`开头的包可以省略前缀scala。

通常类都有一个伴声对象（companion object），其方法就跟Java中的静态方法一样。

### 1.6、apply方法

Scala中有类似于函数调用的语法，例如有一个字符串str，程序语句为str(i)，等价于java的str.charAt(i)、c++的str[i]。类似于str调用了`()`操作符，而操作符就是方法。`()`背后的原理就是有一个`apply`方法，apply方法一般位于伴声对象内部，这时Scala中常见的写法。

> 偶尔`()`表示法会跟另一个Scala特性冲突 （隐式参数）

## 2、控制结构和函数

### 2.1、条件表达式

scala的if/else表达式跟java一样，不过if/else每个分支都有值（返回），可以理解为scala的if else等价于java的if else + 三元表达式

可以将`()`当作表示"无用的值"的占位符

将 Unit 当作Java中的void，表示**无值**的值

### 2.2、语块终止

Scala中不需要明确写分号，只需要能够从上下文中判断出这里是语句的终止。

如果一句程序要写多行，要明确第一行以一个不能用作语句结尾的符号结尾。

```scala
s = s0 + (v - v0) * t + // + 告诉解析器这里不是末尾
0.5 * (a - a0) * t * t
```

### 2.3、块表达式和赋值

Scala中`{ }` 块是一系列表达式，其结果也是一个表达式。块中最后一个表达式的值就是块的值。这个特性对val初始化分为多步完成的情况非常有用。

一个赋值语句结束的块的值是Unit类型。



### 2.4、输入和输出

**命令行输出：**

对应函数： print   println printf

打印字符串时可以使用**字符串插值**。 `$str_var` ，`${str_var}`可以直接写在字符串内部。例如： 

```scala
 print(f"this is a $var") // 格式化的字符串要以f开头。 字符串插值的变量要以 $ 开头
```

**命令行输入**：

用 scala.io.readIn 的 readLine方法读取一行。此外还有 readInt ,readDouble等方法

### 2.5、 循环

scala有用和java相同的 while循环 

scala没有fori 循环，要想使用for循环，使用这种语句：

```scala
for (i <- 表达式)
// 例如
for (i <- 1 to 10) {}
for (i <- 0 to str.length - 1)
```

scala没有提供 contine、break 语句来退出循环。解决方法：

- 使用boolean来控制

- 使用嵌套函数——从当前函数return

- 使用 Breaks 对象中的break方法

  ```scala
  object Ch2_5 {
    def main(args: Array[String]): Unit = {
      Breaks.breakable {
        for (i <- 1 to 10) {
          if (i == 9) {
            println("break")
            Breaks.break
          }
        }
      }
    }
  }
  ```

scala中存在正常的遮挡：范围小的变量遮挡范围大的变量。

### 2.6、 高级for循环

高级特性：

- 以 `<- ` 表达式的形式提供多个 **生成器**，用分号分割。
- 每个生成器可以带 **守卫**，即 if 语句，if 跟其生成器之间没有分号
- 使用任意多的定义，引入可以在循环中使用的变量
- for循环的循环体是以 yield 开始，则该循环会构造出一个集合，每次迭代生成集合中的一个值。

```scala
// 1. 2.
for (1 <- 1 to 3; j <- 1 to 3 if i != j) {}

// 1. 3.
for (i <- 1 to 3; from = 4 - i; j <- from to 3) {}

// 4.
for (i <- 1 to 10) yield i % 3 // 将返回 Vector(1,2,0,1,2,0,1,2,0,1)

```



### 2.7、函数

方法 VS 函数：

- 方法：对 对象进行操作

- 函数：不对对象进行操作

定义函数要给出：函数名称、参数、函数体。

如果函数不是递归的，则可以不给出函数返回值类型，可以通过函数体自动推导出来。代码块中最后一个表达式为返回值，则返回值不需要用到return关键字。

> return关键字：
>
> - 匿名函数中return并不会返回值给调用者，而是跳出到包含他的带名函数中。
> - 可以把return当作函数版本的break语句

### 2.8、 默认参数和带名字的参数

定义函数是可以给参数默认值。

调用函数时可以指定参数名。

### 2.9、变长参数

语法：

```scala
def func(args: Type*) = {}
```



```scala
object CH2_9 {
  def main(args: Array[String]): Unit = {
    println(decorate("wade"))
    println(decorate("wade", "<<"))
    println(decorate("wade", "<<", ">>"))
    println(decorate("wade", left = "<<", right = ">>"))
    println(sum(1, 2, 3, 4, 5))
    println(sumRecu(1 to 5: _*))
  }


  // 默认参数
  private def decorate(str: String, left: String = "[", right: String = "]"): String = {
    left + str + right
  }

  // 变长参数
  def sum(args: Int*): Int = {
    var sum = 0
    for (arg <- args) {
      sum += arg
    }
    sum
  }

  def sumRecu(args: Int*): Int = {
    if (args.isEmpty) 0
    else args.head + sumRecu(args.tail: _*) // 使用 _*  将整数区间转为序列
  }
}
```

### 2.10、过程

过程：函数体包含在花括号当中，但是没有前面的 =  号，那么返回的类型是Unit。

### 2.11、懒值

当val被声明为lazy时，它的初始化将会被推迟，只有在首次取值的时候才会初始化。

### 2.12、 异常

Scala中没有受检异常。

throws 表达式有特殊的类型 Nothing。如果一个分支的返回值是Nothing，那么另一个分支的类型就是表达式的返回类型。

**捕获异常**的语法通常使用**模式匹配**。对于不需要捕获的异常 ，可以使用`_`来替代变量名

```scala
try{
    
} catch{
    case _: xxxException => //do something
    case ex: IOException => //do something
} finally{
    // release source
}
```

## 3、数组相关操作



## 4、映射和元祖

## 5、类

## 6、对象

## 7、包和引入

## 8、继承

## 9、文件和正则表达式

## 10、特质

## 11、操作符

## 12、高阶函数

## 13、集合

## 14、模式匹配和样例类

## 15、注解

## 16、XML出路

## 17、 Futrue

## 18、 类型参数

## 19、 高级特性

## 20、 解析

## 21、 隐式转换和隐式参数