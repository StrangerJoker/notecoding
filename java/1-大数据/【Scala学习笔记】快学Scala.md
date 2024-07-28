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

### 3.1、定长数组

长度不变的数组，使用Scala中 Array

### 3.2、变长数组：数组缓冲

如果长度按需变化，使用ArrayBuffer 。常用方法：

-  += : 在尾端添加一个或多个元素
- ++= : 追加任何集合
- trimEnd(n) ：移除最后n个元素
- insert(index, vals)：在下标index前插入 任意多个元素vals
- remove(index, cnt)：从下标index开始，移除cnt个元素，cnt参数可选， 不选cnt就默认值为1 。
- ArrayBuffer.toArray()：转变为数组
- Array.toBuffer()：转变为数组缓冲

```scala
object CH3_Array {
  def main(args: Array[String]): Unit = {
    val nums = new Array[Int](10)
    val strs = Array("hello", "scala")
    strs(0) = "bye"
    println(strs(0))

    val b = ArrayBuffer[Int]()
    b += 1

    b += (2, 3, 4, 5)
    println(b)
    b ++= Array(6, 7, 8, 9)
    println(b)
    b.trimEnd(3)
    println(b)

    val arr = Array(1, 2, 3, 4, 5, 6, 7)
    for (i <- arr.indices by -1) {
      println(arr(i))
    }

    val tran = for (ele <- arr) yield 3 * ele
    tran.foreach(t => print(t + " "))
    println()
    val tran1 = for (ele <- arr if ele % 2 == 0) yield 3 * ele
    tran1.foreach(t => print(t + " "))
    val tran2 = arr.filter(_ % 2 == 0).map(_ * 3)
    println()
    tran2.foreach(t => print(t + " "))

    //  原地删除数组中的 满足条件的元素
    println()
    val ab = ArrayBuffer(1, -1, 2, -2, 3, -3, 4, -5)
    println(ab)
    val posToKeep: IndexedSeq[Int] = for (i <- ab.indices if ab(i) >= 0) yield i
    for (j <- posToKeep.indices) ab(j) = ab(posToKeep(j))
    ab.trimEnd(ab.length - posToKeep.length)
    println(ab)
  }
}
```

### 3.3、遍历数组和数组缓冲

for循环遍历下标：

```scala
for (i <- 0 until a.length)
for (i < 0 until a.length by step)
```

- until 关键字跟to差不多。to 是闭区间，until是左闭右开。
- 后接 by 表示步长。by的值如果为 -1，表示逆序。

获取下标的方法：

- a.indices：顺序下标
- a.indices.reverse ：逆序下标

for 循环直接获取数组中的元素，类似java增强for循环：

```scala
for(ele <- a) {}
```

### 3.4、数组转换

数组转换动作不会修改原始数组，而是交出一个全新的数组。

for/yield 循环创建了一个类型与原始集合相同的新集合。

### 3.5、常用算法

- 求和：sum
- 最值：max/min
- 排序：sorted、sortWith(比较器)、Sorting.quickSort。 （能对数组进行排序，不能对数组缓冲区排序）
- mkString：将每一个元素按照分隔符拼接起来，形成字符串。此外有重载方法可以指定拼接结果的前缀和后缀

### 3.6、解读Scaladoc

### 3.7、多维数组

和java一样，多维数组是通过数组的数组来实现的。

使用 Array.ofDim 方法定义一个多维数组。

```scala
val matrix = Array.ofDim[Double](3)(4) // 定义一个三行四列的double类型的二维数组
```

访问元素 用两对括号。

```scala
matrix(row)(col)
```

### 3.8、与Java互操作

## 4、映射和元祖

在Scala中，映射是**对偶**的集合，对偶可以理解为两个值构成的组。这两个值可以是任意类型。

### 4.1、构造映射

使用 `->` 操作符来创建对偶：

```scala
"Alice " -> 10
// 上述的输出是
("Alice", 10)
```

使用Map方法创建映射，方法参数是多个对偶

```scala
val map = Map("Alice" -> 10, "Bob" -> 3)
// 或者
val map = Map(("Alice" -> 10), ("Bob" -> 3))
```

### 4.2、获取映射中的值

- 使用 `()`获取，参数为key
- `映射.get(key)` 返回的是一个Option，要么是Some （找到了），要么是None（找不到）
- `contains(key)`：判断key是否存在

### 4.3、更新映射中的值



- 在 `=` 左侧使用 `()`， `map(key) = newValue`
- += ：添加多个关系，`map += ("Alice", 1)`
- -=：移除某个key，`map -= "Alice"`

**不能更新一个不可变的映射。**下述操作都会生成一个新的映射

- `+`：
- `-`：

### 4.4、迭代映射

for循环迭代 key 和 value

```scala
for ((k,v) <- 映射) {}
```

只需要获取 key 或者 value

- `keySet()`方法
- `values()`方法

### 4.5、已排序的映射

使用SortedMap

### 4.6、与Java互操作

### 4.7、元祖

元祖是不同类型的值的聚集。对偶是元祖最简单的形态。

元祖的值是通过将单个的值包含在圆括号中构成的：`(1,"2",3.0)`，可以使用 `_1 _2 _3` 来通过下标访问组成部件，下标（组成部件）从1开始。

元祖可以用于函数返回值不止一个的情况下。

通常通过模式匹配来获取元祖的组成部件：

```scala
val t  = (1,2,3)
// 获取元祖的部件
val (first, second, _) = t // 不需要关注的值，用 _ 替代变量名
```

### 4.8、拉链操作

- 使用元祖的原因是将多个值绑定在一起，以便于他们能够被统一处理。通常用`symbols.zip`方法来完成。
- 使用 `toMap`方法可以将对偶的集合转换为Map

```scala
object CH4_MAP {
  def main(args: Array[String]): Unit = {
    val m1:Map[String, String] = Map(("k1", "v1"), ("k2", "v2"))
    val m2 = Map("l1" -> "b1")

    val m3:mutable.Map[String, String] = mutable.SortedMap("k1" -> "v1", "k2" -> "v2")
    val m4 = mutable.LinkedHashMap("k3" -> "v3", "k1" -> "v1", "k2" -> "v2")
    println(m1)
    println(m2)
    println(m3)
    println(m4)
    
    val  symbols = Array("<", "-", ">")
    val counts = Array(2,10,2)
    val pairs = symbols.zip(counts)
    println(pairs)
    for(pair <- pairs) {
      println(pair)
    }
    println(pairs.toMap)
  }
}
/**
Map(k1 -> v1, k2 -> v2)
Map(l1 -> b1)
TreeMap(k1 -> v1, k2 -> v2)
Map(k3 -> v3, k1 -> v1, k2 -> v2)
[Lscala.Tuple2;@3c09711b
(<,2)
(-,10)
(>,2)
Map(< -> 2, - -> 10, > -> 2)
*/
```



## 5、类

### 5.1、简单类和无参方法

以下是一个简单类：

```scala
class Counter {
  private var value = 0

  def incr(): Unit = {
    value += 1
  }

  def current() = value  // 定义时都可以不带 () 
}

object CH5_Class {
  def main(args: Array[String]): Unit = {
    val counter = new Counter  // 或者new Counter()
    counter.incr()
    println(counter.current)  // 取值方法通常省略 ()
  }
}
```

- Scala中类不声明为 public ，Scala源文件可以包含多个类，所有这些类都具有公有可见性。
- 对于取值方法：调用时可以省略 `()`，甚至在定义时都可以省略（此时调用时必须没有括号）

### 5.2、带getter和setter的属性

属性：有一堆getter和setter。

Scala对每个字段都提供 getter 和 setter 方法。

```scala
class Human {
  var age = 0
}


// use
    val human = new Human
    println(human.age) // call age()
    human.age = 2 // 调用age_=
    println(human.age)
```

在scala中，getter和setter分别叫做 `age `和` age_=`，反编译字节码如下：

```shell
javap -private Human 
����: �������ļ�Human����cm.uestc.edu.scalaimpatient.ch5.Human
Compiled from "CH5_Class.scala"
public class cm.uestc.edu.scalaimpatient.ch5.Human {
  private int age;
  public int age();
  public void age_$eq(int);
  public cm.uestc.edu.scalaimpatient.ch5.Human();
}
```

>Scala对每个字段生成的getter和setter的规则：
>
>- 如果字段是私有的，生成的getter和setter是私有的
>- 如果字段是val，只有getter生成
>- 如果不需要getter和setter，可以将字段声明为 `private[this]`

**Note：getter和setter 可以自己定义实现**

### 5.3、只带getter的属性

定义只读属性使用 `val`，Scala会生成一个私有的final字段和一个getter方法，但没有setter

### 5.4、对象私有字段

Scala中，方法可以访问该类的所有对象的私有字段，但是Scala允许更加严格的访问限制：

- private[this] ：类只能访问当前对象的所有字段，其他对象的对象私有字段不能访问
- private[类名]：指定类可以访问给定的字段。类名必须是当前定义的类，或者包含该类的外部类。

private[this]  修饰的字段是对象私有字段，**Scala 不会生成对象私有字段的getter和setter**

### 5.5、Bean属性

对Scala字段标注为 `@BeanProperty` 时，以age字段为例，会自动生成四个方法：

- age: Int
- age_=(newAge:Int) :Unit
- getAge() ：Int
- setAge(newAge:Int): Unit

**生成方法总结**

| Scala字段                  | 生成的方法                                                   | 何时使用                                     |
| -------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| val/var name               | 公有的name_= （仅针对var）<br />name                         | 可以公开访问其属性                           |
| @BeanProperty val/var name | 公有的 name<br />getName()<br />name_= （仅针对var）<br />setName(...) （仅针对var） | 与JavaBean互操作                             |
| private val/var name       | 私有的 name <br />name_= ( 仅针对var)                        | 与Java的习惯一样                             |
| private[this] val/var name | 无                                                           | 用于将字段访问限制在同一个对象上调用的方法。 |
| private[类名] val/var name | 依赖具体实现                                                 | 将访问权赋予外部类。                         |

>主构造器上定义了字段，并且当前类是JavaBean，则对应字段直接用注解@BeanProperty 修饰即可。

### 5.6、辅助构造器

Scala拥有一个主构造器和任意多的辅助构造器。

辅助构造器：

- 方法名为` this`
- 方法体第一行必须调用 主构造器 或者 其他先前已经定义好的辅助构造器

```scala
class Person {
  var age1 = 0 // 生成 public getter setter
  private var age2 = 0 // 生成 private getter setter
  val age3 = 0 // 生成  getter 字段是final
  private[this] val age4 = 0 // 不会生成  getter setter ；只有当前对象能访问字段，不能访问其他对象的这个字段
  private[Person] val age5 = 0 // 只有当前类的对象才能访问此类的所有对象的这个字段


  private var name = ""
  private var age = 0

  def this(name: String, age: Int) {
    this() // 调用主构造器
    this.name = name
    this.age = age
  }

  def this(name: String) {
    this(name, 0)
  }
}
```

### 5.7、主构造器

主构造器：

- 不已this方法定义，而是与类的定义交织在一起。
- 主构造器的参数直接放置在类名之后
- 主构造器会执行类定义中的所有语句
- 如果没有显示定义主构造器，则会自动拥有一个无参的主构造器
- 如果不带 var、val的参数至少被一个方法调用，则参数自动升格为字段；否则仅作为主构造器中代码能够访问到的普通参数。

**总结：针对主构造器参数生成的字段和方法**

| 主构造器参数                      | 生成字段的方法                                        |
| --------------------------------- | ----------------------------------------------------- |
| name:String                       | 对象私有字段，如果没有方法使用name，则没有该字段      |
| private val/var name:String       | 私有字段，私有 getter setter方法                      |
| val/var name:String               | 私有字段，公有 getter setter方法                      |
| @BeanProperty val/var name:String | 私有字段，公有的Scala版本JavaBean的 getter setter方法 |

如果想要主**构造器为私有**的，则可以放置 private 关键字

```scala
class Person private(val name:string) {}
```



### 5.8、嵌套类

Scala允许在函数中定义函数，在类中定义类

```scala
class NetWork(val name: String) {
  outer => //外部类的引用(this)

  private val members: ArrayBuffer[Member] = new ArrayBuffer[Member]()

  def join(name: String) = {
    val m = new Member(name)
    members += m
    m
  }

  class Member(val name: String) {
    def description = s"$name inside ${outer.name}"
    // def description = s"$name inside ${NetWork.this.name}"
  }
}
```

- 上述代码中，每个实例都有自己的Member类。 network1.Member 和 network2.Member 是不同的类
- 在嵌套类中，通过 `外部类.this` 获取访问外部类的this引用。获取外部类的this还可以通过 上述代码 outer 的语法获取。

## 6、对象

### 6.1、单例对象

Scala没有静态方法和静态字段，但是可以用 object 关键字达到同样的目的。

对象定义了某个类的单个实例。对象本质上可以拥有类的所有特性，甚至可以扩展其他类 或者 特质。

不能给对象提供构造函数。

```scala
object Accounts {
  private var lastNo = 0
  def newUniqNo() = {
    lastNo += 1
    lastNo
  }
}
```

在Scala中对象的作用：

- 存放工具函数或常量
- 高效共享单个不可变实例
- 单例模式

### 6.2、伴生对象

Java中一个类既有成员方法又有静态方法，在Scala中使用通过**类和与类同名的 伴生对象**来实现同样的效果。

伴生对象的功能特性并不在类的作用域内部。Account类必须使用 Accounts.newUniqNo() 才能调用方法。

```scala
class Accounts {
  var id = Accounts.newUniqNo()
  private var balance = 0.0

  def deposit(amount: Double) = balance += amount

  def this(id: Int, balance: Double) {
    this()
    this.id = id
    this.balance = balance
  }
}

object Accounts {
  private var lastNo = 0

  def newUniqNo() = {
    lastNo += 1
    lastNo
  }
}
```



### 6.3、扩展类或特质的对象

一个object可以扩展类以及多个特质，其结果就是 扩展类以及多个特质 的对象。

```scala
abstract class UndoableAction(val desc: String) {
  def undo(): Unit

  def redo(): Unit
}

object DoNothing extends UndoableAction("do Nothing") {
  override def undo(): Unit = {}

  override def redo(): Unit = ???
}
```



### 6.4、apply方法

当使用 `Object(p1,p2...)`代码时，object的apply方法会被调用，apply方法通常返回的是伴生类的对象

```scala
class Accounts {
  var id = Accounts.newUniqNo()
  private var balance = 0.0

  def deposit(amount: Double) = balance += amount

  def this(id: Int, balance: Double) {
    this()
    this.id = id
    this.balance = balance
  }
}

object Accounts {
  private var lastNo = 0

  def newUniqNo() = {
    lastNo += 1
    lastNo
  }

  def apply(balance: Double): Accounts = {
    new Accounts(newUniqNo(), balance)
  }
}



// run
object CH6_object {
  def main(args: Array[String]): Unit = {
    val account: Accounts = Accounts(0.1)
    println("id=" +account.id)
  }
}

```



### 6.5、应用程序对象

可以扩展App特质，然后将程序代码放入构造器内部。

可以通过 args 获取命令行参数

```scala
object CH6_App extends App {
  println("app")
  if (args.length <= 0) {
    println("hello world")
  } else {
    println(s"hello ${args(0)}")
  }
}
```



### 6.6、枚举

Scala没有枚举类型，不过标准库提供了一个Enumeration助手类用于产出枚举。

```scala
object CH6_App extends App {
  val en:TrafficLightColor.Value = TrafficLightColor.GREEN
  if (en == TrafficLightColor.GREEN) println("1")
  println(en.id)
  println(en.toString)
  for (elem <- TrafficLightColor.values) {
    println(elem.toString)
  }
}


object TrafficLightColor extends Enumeration {
  type TrafficLightColor = Value // 起类型别名
  val RED, GREEN, YELLOW = Value
}

```

注意：

-  枚举的类型是：TrafficLightColor.Value
- 通过id方法获取枚举的ID
- 通过toString方法获取枚举的名称
- 通过 TrafficLightColor.values 获取所有的枚举

## 7、包和引入（doing）

### 7.1、包

管理大型程序中的名称，类似于c++的namespace和java中包

不同的是：

- 源文件目录跟包之间没有强制关联关系
- 可以在同一个文件中为多个包贡献内容

### 7.2、作用域规则

### 7.3、串联式包语句

### 7.4、文件顶部标记法

### 7.5、包对象

### 7.6、包可见性

### 7.7、引入

### 7.8、任何地方都可以声明引入

### 7.9、重命名和隐藏方法

### 7.10、隐式引入

## 8、继承

### 8.1、扩展类

extends 和 final关键字的用法和java一样

### 8.2、重写方法

重写一个非抽象方法必须使用 override 修饰符

调用超类的方法和java一样，使用  super.method() 

### 8.3、类型检查和转换

- 判断某个对象是否是某个类的实例，用`obj.isInstanceOf[类名]`，然后使用`obj.asInstanceOf[类名]`强制转换引用

- 如果想测试p执行某个类的对象，但是又不是其子类

  ```scala
  if (p.getClass == classOf[类名])  // 类型检查通常使用模式匹配来写
  ```

### 8.4、受保护字段和方法

将字段和方法声明为 `protected`，被修饰的方法和属性只能被子类访问。

### 8.5、超类的构造

- 只有主构造器可以调用超类的主构造器。
- Scala扩展Java类，则主构造器必须实现某一个Java类的构造方法

```scala
class Person(name: String, age: Int) {
}
// 继承 父类，并且调用父类的主构造器
class Employee(name: String, age: Int, salery: Double) extends Person(name, age) {
}

// 继承一个Java类，调用一个构造方法就可以
class OtherEmployee(name: String, age: Int, salaery: Double) extends JavaPerson(name, age) {
}
```

### 8.6、重写字段

scala的字段是由一个私有字段和取值器/改值器方法构成。可以用另一个同名的val重写一个val字段。子类有一个私有字段和一个公有的getter方法，而这个getter方法重写了超类的getter方法。

```scala
class Person(val name: String) {
  override def toString: String = s"${getClass.getName}[name=$name]"
}

class SecretAgent(codename: String) extends Person(codename) {
  override val name: String = "secret" // name保密

  override def toString: String = "secret"
}
```

上述写法优点复杂，更常见的写法如下：

```scala
abstract class Person1 {
  def id: Int  // 这是一个抽象方法
}

class Student1 (override val id:Int) extends Person1 {
}
```

**总结**

|         | 用val                                                    | 用def  | 用var                                              |
| ------- | -------------------------------------------------------- | ------ | -------------------------------------------------- |
| 重写val | （1） 子类有一个私有字段<br />（2）getter方法重写超类    | 错误   | 错误                                               |
| 重写def | （1）子类有一个私有字段<br />（2）getter方法重写超类方法 | 同java | var可以重写getter/setter。<br />只重写getter会报错 |
| 重写var | 错误                                                     | 错误   | 仅当超类的var是抽象的才可以                        |



### 8.7、匿名子类

跟java的匿名类有点像。

创建一个结构类型。

### 8.8、抽象类

使用abstract关键字修饰的类。如果一个类至少存在一个抽象方法，这个类必须是抽象类。

**重写抽象方法，可以不加 override** 

```scala
abstract class Person2(val name: String) {
  def id: Int
}

class Emp(name: String) extends Person2(name) {
  override def id: Int = ???
}
```



### 8.9、抽象字段

类还可以有抽象字段：一个没有初始值的字段。

**重写抽象字段，可以不加 override** 

```scala
abstract class Person3 {
  val id: Int // 抽象字段，带抽象的getter
  var name: String // 抽象字段，带抽象的getter和setter
}

class Emp1(val id:Int) extends Person3 {
  // override var name: String = "111"
  var name: String = "111"
}
```



### 8.10、构造顺序和提前定义

构造顺序：超类的构造器优先于子类的构造器执行

提前定义：在超类的构造器执行之前初始化子类的val字段。

```scala
object ch8_10 {
  def main(args: Array[String]): Unit = {
    val ant = new Ant
    println(ant.env.mkString("Array(", ", ", ")"))
  }
}

class Creature {
  val range: Int = 10
  val env: Array[Int] = new Array[Int](range)
}

// class Ant extends Creature {
//   override val range: Int = 2
// }

// 提前定义的语法
class Ant extends {
  override val range = 2 
} with Creature {

}
```

### 8.11、Scala类继承关系

![](https://i-blog.csdnimg.cn/blog_migrate/db138cf2f7bda2c16d60dfee4baf2713.png)

**与Java基本类型 相关的类 以及 Unit ，都扩展自 AnyVal；其他类都是AnyRef的子类。**AnyVal 和 AnyRef 扩展自Any类

类中的方法：

**Any：**

- isInstanceOf
- asInstanceOf

**AnyVal**：无方法，起标记作用

**AnyRef**：追加了来自Object类的方法

- wait
- notify
- notifyAll
- synchronized

**所有的Scala类都实现 ScalaObject接口**

**Null类型的唯一实例是null，null可以赋值给任何引用，但是不能赋值给值类型的引用。**

**Nothing类型没有实例**，通常用于泛型，例如列表Nil的类型是List[Nothing]，他是List[T]的子类型

**`???方法`被声明为返回类型是Nothing，但是它从不返回，被调用时直接抛出 scala.NotImplementedError**

> Nothing和c++中的void是两个概念。
>
> scala中的 void 是 Unit，该类型只有一个值，即 `()`。

### 8.12、对象相等性

在scala中，**`== `是判断两个对象内容相等的方法**，判断地址是否相等的是AnyRef中的`eq`方法。

针对引用类型，== 操作符会先判空，再调用equals方法。

- 重写equals方法：确保参数类型是Any
- 重写hashcode方法：`##`方法是hashcode的null值安全版本，对于null值返回0而不是异常

```scala
object User {
  def main(args: Array[String]): Unit = {
    val wade = new User("wade", 1)
    val wade1 = new User("wade", 1)
    print(wade1 == wade)
  }
}

class User (val name:String, val id:Int) {
  override def hashCode(): Int = (name, id).##  // 将两个字段的值结合起来

  override def equals(obj: Any): Boolean = this.hashCode() == obj.hashCode()
}
```

### 8.13、值类

值类：定义内联的类。具备以下性质：

- 扩展自AnyVal
- 主构造器有且仅有一个参数，该参数是一个val，且没有方法体
- 没有其他字段或者构造器
- 自动提供equals 和 hashcode方法，针对的是背后的值

当new一个值类的时候，编译器不会分配一块内存，而是使用背后的值。

```scala
object MilTime {
  def main(args: Array[String]): Unit = {
    val lunch = new MilTime(1230)
    println(lunch.minute)
    println(lunch.hours)
    println(lunch)
  }

  def apply(time: Int): MilTime = new MilTime(time)
}

class MilTime private(val time: Int) extends AnyVal {
  def minute: Int = time % 100

  def hours: Int = time / 100

  override def toString: String = f"$time"
}
```

值类的设计是为了做高效的隐式转换，可以用他们实现无额外开销的小微类型。

## 9、文件和正则表达式

### 9.1、读取行

使用以下方法读取文件的所有行，返回的是一个Iterator

```scala
Source.fromFile("filepath").getLines()
```

读取之后要将source.close

### 9.2、读取字符

直接将Source当成迭代器，

```scala
for (c <- source) {}
```

### 9.3、读取词法单元和数字

### 9.4、从URL或其他资源读取

- 读URL：Source.fromURL("url", "encode")
- 读字符串：Source.fromString("hello world")
- 读命令行：Source.stdin

### 9.5、读取二进制文件

Scala不提供，借助Java的库读取。

### 9.6、写文件

Scala不提供，借助Java的 PrintWriter

### 9.6、访问目录

Scala不提供，借助Java

### 9.7、序列化

Scala类需要继承 Serializable，并且使用注解 @SerialVersionUID(123L) 指定序列化的UID

```scala
@SerialVersionUID(123L)
class Person extends Serializable {

}
```

scala中的集合都是可以序列化的。

### 9.8、进程控制（doing）

### 9.9、正则表达式（doing）

### 9.10、正则表达式组（doing）

## 10、特质

**特质：同时可以提供抽象方法和具体方法，以及状态。**

### 10.1、why没有多继承

### 10.2、当作接口使用的特质

- 特质中未实现的方法都是抽象的，无序写abstract关键字

- 子类重写特质抽象方法无需写 override 关键字

### 10.3、带具体实现的特质

就是有具体方法的特质

### 10.4、带有特质的对象

就是new出来一个带有特质的类的对象

### 10.5、叠加在一起的特质

特质可以多with，类似于java中的多实现。但是有一些区别。

```scala
trait Logger {
  def log(msg: String): Unit
}

trait ConsoleLogger extends Logger {
  override def log(msg: String): Unit = {
    println(msg)
  }
}

trait TimestampLogger extends ConsoleLogger {
  override def log(msg: String): Unit = {
    super.log(s"${java.time.Instant.now()} $msg")
  }
}
```

对特质而言 ，`super.log` 并不像类的一样。**实际上`super.log`调用的是另一个特质的log方法，具体是哪一个特质取决于特质被添加的顺序。**

对于下面的代码，那么方法将从Logger3.log开始，super就是调 logger2.log方法。**“从后往前”**调用

```scala
new Account  with Logger1 with logger2 with Logger3
```

如果需要控制具体哪一个特质的方法被调用，则可以在方括号中给出名称： `super[类].someMethod(...)`

### 10.6、在特质中重写抽象方法

如果调 super.log 时发现 父类没有实现，则会报错。解决方法是 重写特质中的抽象方法，要加上 abstract 关键字，如下：

```scala
trait Logger {
  def log(msg: String): Unit
}

trait TimestampLogger extends Logger {
  abstract override def log(msg: String): Unit = {
    super.log(s"${java.time.Instant.now()} $msg")
  }
}

```

### 10.7、当作富接口使用的特质

### 10.8、特质中的具体字段

特质中的字段可以是抽象的，也可以是**具体的（字段给了初始值**）。

对于特制里的每一个具体字段，，使用该特质的类都会获得一个字段与之对应。这些字段不是被继承的，而是简单的加到了子类当中，属于子类。

```scala
trait ShortLogger extends ConsoleLogger {
  private val maxLen = 15
  override def log(msg: String): Unit = {
    if (msg.length > 15) {
      super.log(s"${msg.substring(0, maxLen)}")
    } else {
      super.log(s"$msg") // 打印的是被截断的msg
    }
  }
}
```



### 10.9、特质中的抽象字段

特质中没有给初始值的字段是抽象的。在子类中**必须重写**（无需写override）。

```scala
trait ShortLogger1 extends ConsoleLogger {
  val maxLen:Int
  override def log(msg: String): Unit = {
    if (msg.length > 15) {
      super.log(s"${msg.substring(0, maxLen)}")
    } else {
      super.log(s"$msg") // 打印的是被截断的msg
    }
  }
}

class Acc extends ShortLogger1 {
  override val maxLen: Int = 11
}
```



### 10.10、特质构造顺序

构造器的执行顺序：

- 首先调用超类的构造器
- 特质构造器在超类之后、类构造器之前执行
- 特质由左到右被构造
- 在每个特质中，父特质先被构造
- 如果多个特征公有一个父特质，并且父特质已经被构造，则不会重复构造
- 所有特质构造完毕，子类被构造

### 10.11、初始化特质中的字段

特质不能有构造器参数，每个特质由一个无参的构造器。**缺少构造器参数是 特质 跟 类 唯一的区别**

解决特质中字段初始化的问题的方法：（1）提前定义；（2）懒值

### 10.12、扩展类的特质

特质也可以扩展类，这个类会自动成为所有混入该特质的超类。

```scala
import java.io.IOException


trait LoggerException extends Exception with ConsoleLogger {
  def log(): Unit = {
    log(getMessage)
  }
}

// 特质的超类自动成为当前类的超类
class UnhappyException extends LoggerException {
  override def getMessage: String = "arggh"
}

// 可以再混入一个跟特质相关的类
class HappyException extends IOException with LoggerException {

}

// error 扩展一个不相关的类，则不能混入整个特质
class UnhappyFrame extends JFrame with LoggerException {
  
}
```

### 10.13、自身类型

特质扩展类时，编译器能够确保：所有混入该特质的类都将这个类作为超类。Scala还有一套机制能够保证这一点：自身类型（self type）。

语法：

```scala
this: 类型 => 
```

在特质的方法中，可以调用 该自身类型 的任何方法。

```scala
trait LoggerException extends ConsoleLogger {
  this: Exception => // 当前特质不扩展类Exception，具有一个自身类型Exception，表示只能混入Exception的子类
  def log(): Unit = {
    log(getMessage) // getMessage 是 Exception 的方法，可以调用自身类型的方法
  }
}
```

自身类型也可以处理“结构类型”：这种类型只给出类必须要有的方法，而不是类的名称

```scala
trait LoggerException extends ConsoleLogger {
  this: {def getMessage(): String} => // 只有类有 getMessage 方法，则可以混入
  def log() {
    log(getMessage())
  }
}
```



### 10.14、背后发生了什么（doing）

## 11、操作符

### 11.1、标识符

变量、函数、类等名称。

Scala中的标识符保留字：[Scala 关键字/保留字_scala中声明单列对象需要使用什么关键字-CSDN博客](https://blog.csdn.net/cauchy8389/article/details/104462138)

使用反引号可以包含几乎任何字符序列。 

```scala
val `val` = 4
Thread.`yeild`()
```



### 11.2、中置操作符

中置操作符（二元）：操作符介于两个参数之间

```scala
a 标识符 b
// 等价于
a .标识符(b)
```

其中标识符代表一个带有两个参数的方法（一个隐式参数、一个显示参数）

**定义操作符**：将想用的操作符的名称来定义一个方法

```scala
class Complex(private val real: Double, private val img: Double) {
  def +(other: Complex): Complex = {
    new Complex(this.real + other.real, this.img + other.img)
  }

  def -(other: Complex): Complex = {
    new Complex(this.real - other.real, this.img - other.img)
  }

  override def toString: String = {
    "[real:" + real + "," + "img:" + img + "]"
  }
}
```

### 11.3、一元操作符

有几个参数的操作符，就成为几元操作符？

`+ - ! ~` 可以作为前置操作符，出现在参数之前，它们被转换成队名的 `unarray_操作符`的方法调用：

```scala
-a
a.unary_-
```

出现在参数之后，为后缀操作符

```scala
    a toString 
    a.toString
```



### 11.4、赋值操作符

名称形式为 `操作符=`

```scala
a 操作符= b
// 等价于
a = a 操作符 b
```

Note：

- `>= <= !=`不是赋值操作符
- 以 = 开头的操作符不是赋值操作符 （== === =/=)
- 如果a有一个名为 `操作符=` 的方法，那么该方法会直接被调用



### 11.5、优先级

根据操作符中的第一个字符决定优先级

```scala
(一些其他特殊字符)
* / %
+ -
:
= !
< >
& 
ˆ
|
(字母)
(赋值操作符)
```

后置操作符低于中置操作符

```scala
a mid b post

(a mid b) post
```



### 11.6、结合性

操作符结合性决定了它们是从左到右求值还是从右到左求值。

在Scala中，所有操作符都是**左结合**的。除了  以`:`结尾、赋值、`::`（右结合）

### 11.7、apply方法 和 update 方法

Scala中允许 `f(arg1,arg2)` 的代码，如果`f`不是函数或方法，则等同于 `f.apply(arg1,arg2)`.

除非这代码出现在赋值语句的左侧 `f(arg1,arg2) = value` ，等价于调用 `f.update(arg1,arg2)`

apply方法经常出现在伴生对象里。

### 11.8、提取器

提取器就是一个带有 `unapply`方法的对象，unapply方法可以理解为apply的反向操作：入参是一个对象，返回值是option，包含一个元祖，内部有需要提取的值。

### 11.9、带单个参数或无参的提取器

Scala没有只带一个组件的元祖，如果unapply要提取单值，则返回一个目标类型为Option

### 11.10、unapplySeq方法

要提取任意长度的值的序列，应该用unapplySeq来命名方法，返回的是一个Option[Seq[A]]

### 11.11、动态调用(doing)

## 12、高阶函数

### 12.1、作为值的函数

### 12.2、匿名函数

### 12.3、带函数参数的函数

### 12.4、参数（类型）推断

### 12.5、一些有用的高阶函数

### 12.6、闭包

### 12.7、SAM转换

### 12.8、科里化

### 12.9、控制抽象

### 12.10、return 表达式

## 13、集合

## 14、模式匹配和样例类

## 15、注解

## 16、XML出路

## 17、 Futrue

## 18、 类型参数

## 19、 高级特性

## 20、 解析

## 21、 隐式转换和隐式参数