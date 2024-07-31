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

可以在变量中存放函数：

- 调用函数
- 作为参数传递函数

```scala
object ch12_1 {
  def main(args: Array[String]): Unit = {
    val num = 3.14
    val func = math.ceil _ // _ 运算符 将 ceil方法变成了函数
    val func1: (Double) => Double = math.ceil // 不需要 _
    val doubles = Array(1.1, 2.2, 3.4).map(func1)
    println(doubles.mkString("Array(", ", ", ")")) // Array(2.0, 3.0, 4.0)
  }
}
```



### 12.2、匿名函数

有时候无需给出函数名

```scala
    val ints = Array(1, 2, 3, 4, 5).map((x: Int) => x * 3 + 1)
    println(ints.mkString("Array(", ", ", ")")) // Array(4, 7, 10, 13, 16)
```

> 任何以def定义的都是方法，不是函数

### 12.3、带函数参数的函数

**函数作为函数的参数。**函数的类型可以写为：

```scala
(参数类型) => 结果类型
```

```scala
object ch12_3 {
  def main(args: Array[String]): Unit = {
    println(valueAtOneQuarter(math.ceil))
  }

  def valueAtOneQuarter(f: (Double) => Double) = f(0.25)
}
```

`valueAtOneQuarter`的类型为 `((Double) => Double) => Double`

**此外函数还可以作为函数的返回值**

```scala
def multiBy(factor: Double) = (x: Double) => factor * x

//call
  println(multiBy(3)(20))
```

其返回类型为`(Double) => (Double => Double)`

### 12.4、参数（类型）推断

参数类型推断有助于节省代码量。 不必写 `(x:Double) => x * 3` ，直接写 `(Double) => _ * 3` 或则`(x) => x * 3`

### 12.5、一些有用的高阶函数

map、filter等

### 12.6、闭包

闭包：函数/变量 在不再处于作用域范围内的地方被调用/使用。

```scala
def multiBy(factor: Double) = (x: Double) => factor * x
// call
    val triple = multiBy(3)
    val half = multiBy(0.5)
    println(s"${triple(33)} ${half(10)}")
```

上述代码，第一次调用时 factor为 3，第二次为0.5。每一个返回的函数都有自己的factor设置。

### 12.7、SAM转换

Single Abstract Method，类似于Java 的函数式接口

### 12.8、科里化

函数科里化：将原来接收两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回原有第二个参数作为参数的函数。

```scala
object ch12_8 {
  def main(args: Array[String]): Unit = {
    val mul = (x: Int, y: Int) => x * y
    val mulOneAtOneTime = (x: Int) => ((y: Int) => x * y)
    println(s"计算两个数的乘积: ${mul(3, 4)}")
    println(s"计算两个数的乘积: ${mulOneAtOneTime(3)(4)}")
    println(s"计算两个数的乘积: ${mulOneAtOneTime1(3)(4)}")
    // 定义函数
    val mulOneAtOneTime2 = (x:Int) => (y:Int) => x * y  // 函数类型是  Int => (Int  => Int)
    println(s"计算两个数的乘积: ${mulOneAtOneTime2(3)(4)}")


    val a = Array("Hello", "World")
    val b = Array("Hello", "World")
    println(a.corresponds(b)(_.equalsIgnoreCase(_)))
  }

  // 定义方法 简写形式
  def mulOneAtOneTime1(x: Int)(y: Int) = x * y  // 方法类型是： (x:Int)(y:Int)Int
}
```

上述代码的 corresponds 方法是典型的科里化，其定义为：

```scala
def corresponds[B](that: _root_.scala.collection.GenSeq[B])(p: (A, B) => _root_.scala.Boolean): _root_.scala.Boolean = ???
```



### 12.9、控制抽象(doing)

### 12.10、return 表达式(doing)

## 13、集合(doing)

### 13.1、主要的集合特质

![](https://i-blog.csdnimg.cn/blog_migrate/e4fdea6b73d4869b9d28aee38658679e.png)

所有Scala都扩展自 Iterable：

- Seq：又先后次序的值的序列
- Set：一组没有先后次序的值
- Map：一组对偶

每个Scala集合都有一个伴生对象，里面有一个apply方法， apply方法可以用来构建该集合中的实例。

### 13.2、可变和不可变集合

Scala同时支持可变和不可变集合，推荐使用不可变集合。对不可变集合的更新操作（如插入值，remove值）都会生成一个新的集合。

Predef对象里还有指向不可变特质的类型别名List，Set，Map。

**不可变集合继承类图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/7e69849f50d14d2eb791f79aee3d22ba.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6I-c6I-c55qE5aSn5pWw5o2u5byA5Y-R5LmL6Lev,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)



**可变集合继承类图**

![](https://img-blog.csdnimg.cn/3812e608086441f8ba830b2d22c02b43.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6I-c6I-c55qE5aSn5pWw5o2u5byA5Y-R5LmL6Lev,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 13.3、序列

vector是ArrayBuffer的不可变版本：一个带下标的序列，快速支持随机访问，vector是以树结构的形式实现的。

range是一个整数序列。range只是存储序列的起始值，接收值和步长，不会存储数据。 可以用to 和until生成range

### 13.4、列表

在Scala中，列表要么是Nil（空表），要么是一个head元素加上一个tail，而tail又是一个列表。

`::`操作符从给定的头和尾创建一个新的列表。 `::`是右结合的，列表从末尾开始创建。

可变 的列表：ListBuffer，由链表支撑的数据结构。

### 13.5、集

Set

LinkedHashSet

SortedSet

方法：

- contains
- subsetOf
- union ， ++
- intersect
- diff，--

### 13.6、用于添加或去除元素的操作符 （doing）



### 13.7、常用方法（doing）

### 13.8、将函数映射到集合

map：将函数应用到集合中的每个元素，并交出其结果。

flatMap：和map一样，不过每个元素返回的是多个值。

transfer：和map一样，不过是当场执行。应用与可变结合

collect：用于偏函数

groupBy：返回一个映射，key是函数求值后的值，value是函数求值得到给定键的元素的集合。

### 13.9、化简、折叠和扫描（doing）

### 13.10、拉链操作

zip：将 左右两个集合 组合成一个个对偶的列表

zipAll：和zip一样，不过当两个集合长度不一样时，则最终长度是取最小值

### 13.11、迭代器

`集合.iterator`方法从集合获取一个迭代器，然后使用 next 和 hashNext 方法遍历集合中的元素。

在调用 map, filter, count, sum,  length 方法后，迭代器将位于集合尾端。

### 13.12、流

流是一个尾部被懒计算的不可变的列表。

`#::` 操作符用于构建一个流，其中流的尾部是未求值的。

- 使用 `.tall()`强制对下一个元素求值
- 想要得到多个答案，使用`.take(n)`方法，然后调用`force`，切勿直接调用force，否则oom

```scala
object ch13_12 {
  def main(args: Array[String]): Unit = {
    val tenOrMore = numsForm(10)
    println(tenOrMore)
    println(tenOrMore.tail)
    println(tenOrMore.tail.tail)


    println(tenOrMore.take(5).force)
  }
  def numsForm(n: Int): Stream[BigInt] = n #:: numsForm(n + 1)
}
```



### 13.13、懒试图

对集合应用`view`方法也能得到流式处理的效果。

- 试图不会缓存任何值，再次使用未求值的集合会重新计算。

```scala
object ch13_13 {
  def main(args: Array[String]): Unit = {
    val pal = (1 to 1000000).view.map(x => x * x).filter(x => x.toString == x.toString.reverse)
    val col = pal.take(10).mkString(",") 
    println(col)
  }
}
```



### 13.14、与Java集合的互操作

使用JavaConversions 提供的转换方法。

### 13.15、并行集合

`par`方法产生一个当前集合的并发实现。

par方法返回的并行集合属于扩展自ParSeq、ParSet、ParMap特质的类型。

par方法能够应用的条件：操作符必须是**结合的**，就是`(a op b) op c` 等价于 `a op (b op c)`

```scala
object ch13_15 {
  def main(args: Array[String]): Unit = {
    val coll = (1 to 30).toList
    println(coll.par.count(_ % 2 == 0))
    println(coll.par.sum)
  }
}
```

并行集合使用的是全局的fork-join线程池

## 14、模式匹配和样例类

### 14.1、更好的switch

scala的模式匹配案例：

```scala

object ch14_1 {
  def main(args: Array[String]): Unit = {
    var ch = '+'
    val sign = ch match {
      case '+' => 1
      case '-' => -1
      case _ => 0
    }
    
    var str = "0x"
    val strSign = str match {
      case "0" | "0x" | "0X" => 1
      case _ => -1
    }
  }

}
```

跟switch不同，scala的模式匹配不会意外调入下一分支。

使用 `|` 符号分隔多个选项

### 14.2、守卫

自己理解：模式匹配case后的语句是一个表达式，默认是 `ch equals 这个值`，然而也可以直接写一个表达式，整个表达式就是守卫，守卫可以是任何boolean条件。

模式总是自上而下匹配的，如果守卫整个模式不匹配，则匹配下一个模式。

```scala
object MatchDemo {
  def main(args: Array[String]): Unit = {
    var str = "3+4-22s"

    for (ch <- str) {
      var res = ch match {
        case _ if Character.isDigit(ch) => Character.digit(ch, 10) // 守卫
        case '+' => 1
        case '-' => -1
        case _ => 0
      }
      println(res )
    }
  }

}
```



### 14.3、模式中的变量

如果case表达式后接一个变量名，那么给这个变量赋值。`case _`就是一种特殊的情况，变量名是`_`。

```scala
str(i) match {
    case ch => ....
}
```



### 14.4、类型模式

对表达式的类型进行匹配。更推荐使用模式匹配，而不是 isInstanceOf 函数。

模式匹配的时候，必须要给出一个变量名。

**注意：**不要对泛型进行模式匹配，即 `case Map[Int, String]`，直接使用一个通用的匹配`case Map[_,_]`

```scala
object ch14_4 {
  def main(args: Array[String]): Unit = {
    val obj: Int = 13
    val res = obj match {
      case x: Int => x
      case s: String => s.toInt
      case _: BigInt => Int.MaxValue
      case _: => 0
    }
    println(res)
  }

}
```



### 14.5、匹配数组、列表和元祖

**数组**

- 第一个匹配数组只含有一个0的数组
- 第二个匹配任何只有两个元素的数组
- 第三个匹配任何以0开始的数组

```scala
object ch14_5 {
  def main(args: Array[String]): Unit = {
    val arr = Array(0/*, 1*//*, 2, 3, 4*/)
    val res = arr match {
      case Array(0) => "0"
      case Array(x, y) => s"$x,$y"
      case Array(0, _*) => "0 ..."
      case _ => "something else"
    }
    println(res)
  }

}
```

如果匹配到 `_*`的可变长度参数绑定到变量，可以使用 `@`表示法：

```scala
case Array(x, rest @ _*) => rest.min
```



**列表**：以数组同样的方式匹配列表

```scala
val list = List(0, 1, 2, 3, 4)
var res1 = list match {
  case 0 :: Nil => "0"
  case x :: y :: Nil => s"$x, $y"
  case 0::tail => "0 ..."
  case _ => "something else"
}
println(res1)
```



**元祖**

```scala
  val pair = (1,2)
  val res2 = pair match {
    case (0, _) => "0..."
    case (y, 0) => s"$y 0"
    case _ => "neither is 0"
  }
  println(res2)
}
```

上述代码，将变量绑定到元祖的不同部分。

**如果模式有不同的可选分支，不能使用除下划线外的其他变量名**

```scala
pair match {
    case (x,_) | (_,y) => ...   // OK
    case (x, 0) | (0, y) => .... //error  不能对可选分支做变量名绑定
}
```



### 14.6、提取器

14.5 的原理背后是提取器：带有从对象中提取值的`unapply`方法和`unapplySeq`方法对象。unapply 提取一个值，unapplySeq提取多个值（序列）

Array的伴生对象就是一个提取器。

对于14.5的模式匹配代码，首先是调用提取器，提取一个序列的值，然后再按照规则进行匹配。

### 14.7、变量声明中的模式

在变量声明的时候利用模式

```scala
object ch14_7 {
  def main(args: Array[String]): Unit = {
    val (x, y) = (1,2) 
    val (q, r) = BigInt(10) /% 3  // /%  操作符返回商和余
    val array = (1 to 10).toArray
    val Array(first , second, rest @ _*) = array  // 第一个赋值给first，第二个赋值为second，剩下的赋值给rest
  }

}
```

### 14.8、for表达式中的模式

在for表达式内，使用带变量的模式，对每一个遍历的值，这些变量都会被绑定。

````scala
for ((k ,v) <- System.getProperties()) {} 
````

甚至可以在之后加上守卫

```scala
for ((k ,v) <- System.getProperties() if v == "") {} 
//等价于
for ((k ,"") <- System.getProperties()) {} 
```



### 14.9、样例类

一种特殊的类，经过优化以被用于模式匹配。

```scala
object ch14_9 {
  def main(args: Array[String]): Unit = {
    val amt: Amount = Dollar(11.1)

    val str = amt match {
      case Dollar(v) => s"$$$v"
      case Currency(_, u) => s"Oh noes, I got $u"
      case Nothing => ""
    }
    println(str)
  }

}

abstract class Amount

case class Dollar(value: Double) extends Amount

case class Currency(value: Double, unit: String) extends Amount

case object Nothing extends Amount
```

> 样例类要使用 `()`
>
> 样例对象不适用`()`

声明样例类的时候，会自动发生几件事情：

- 构造器中的每个参数都成为val，除非显示地声明为var
- 在伴生对象中提供 apply 方法，生成对象是不适用new
- 提供unapply方法让模式匹配能够自动工作
- 将生成 toString、equals、hashcode 和 copy方法——除非重写这些方法

除了上面几点，样例类跟类完全一样。

### 14.10、copy方法和带名参数

样例类的copy方法创建一个跟现有对象完全一样的新对象。可以在copy的时候指定属性名来修改新对象的属性。

```scala
val newCurr = curr.copy(unit = "new curr")
println(newCurr)
```

### 14.11、case语句中的中置表示法（doing）

如果unapply方法返回的是一个对偶，则可以将case语句按照中置表达式表示法来写：

```scala
object ch14_11 {
  def main(args: Array[String]): Unit = {
    val amt: Amount = Dollar(11.1)
    amt match {
      case a Currency u =>  s"$a,$u" // 等价于 case Currency(a,u)
    }
  }

}
```



### 14.12、匹配嵌套结构

模式匹配可以匹配到特定的嵌套：

```scala
package cm.uestc.edu.scalaimpatient.ch14

object ch14_12 {
  def main(args: Array[String]): Unit = {
    val bundle = Bundle("outer", 11.22,
      Article("scala", 22.33),
      Bundle("inner", 10.0,
        Article("fast scala", 11.11),
        Article("deep java", 22.22))
    )
    val str = bundle match {
      // case Bundle(_, _, Article(desc, _), _*) => s"$desc" // 将desc绑定到Bundle的第一个article上
      case Bundle(_, _, art@Article(_, _), rest@_*) => s"${art.desc}" // 将art绑定到Bundle的第一个article上 ,将剩下的绑定到rest上
    }
    println(str)
  }
}

abstract class Item

case class Article(desc: String, price: Double) extends Item

case class Bundle(desc: String, price: Double, items: Item*) extends Item

```

### 14.13、样例类是邪恶的吗

样例类适用于标记不会更改的类。

### 14.14、密封类

用样例类做模式匹配时，如果想要确保自己已经列出了所有可能的选择，则需要将样例类的超了声明为 `sealed`——密封类。

密封类要求所有样例类必须在 密封类所在的文件中。

### 14.15、模拟枚举

样例类的作用之一是：在Scala中模拟出枚举的效果。

```scala
object ch14_15 {
  def main(args: Array[String]): Unit = {
    val color:TrafficLightColor = Red

    val str = color match {
      case Red => "Red"
      case Green => "Green"
      case Yellow => "Yellow"
    }
    println(str)
  }
}


sealed abstract class TrafficLightColor

case object Red extends TrafficLightColor
case object Green extends TrafficLightColor
case object Yellow extends TrafficLightColor
```

### 14.16、Option类型

Option用样例类表示就是可能存在 可能也不存在的值。即Some[T] 和 None

Map.get 方法 返回的是就是一个Option， 处理Option：

```scala
object ch14_16 {
  def main(args: Array[String]): Unit = {
    val map = Map(1 -> 1, 2 -> 2)
    val opt: Option[Int] = map.get(1)
    opt match {
      case Some(value) => println(value)
      case None => println("no score")
    }

    if (opt.isEmpty) println("no score") else println(opt.get)

    println(opt.getOrElse("no score")) // 推荐使用

    for (ele <- opt) println(ele)
  }

}
```

Option 还可以接 filter map等方法，调用的前提是 不是None

### 14.17、偏函数

被包在花括号内的一组case语句是一个**偏函数**(partial func)——一个并非对所有输入值都有定义的函数。

偏函数是 `PartitialFunction[A,B]`的一个实例（A是参数，B是返回值），包含两个方法：

- `apply`：从匹配到的模式计算返回值
- `isDefinedAt`：在输入至少匹配其中一个模式时返回true

```scala
object ch14_17 {
  def main(args: Array[String]): Unit = {
    val f:PartialFunction[Char, Int] = {
      case '+' => 1;
      case '-' => -1;
    }
    println(f('-')) // -1
    println(f('+')) // 1
    println(f('*')) // matchError
    println(f.isDefinedAt('+')) // true
    println(f.isDefinedAt('*')) // false
  }

}
```

偏函数表达式必须是在一个编译器能够推断出返回类型的上下文里。当将偏函数赋值给一个有类型的变量 或者 作为参数传递时，就属于这种场景。

**PartitialFunction 和 Function**：

- PartitialFunction[P, R].list 方法，将转换成Funciton 类型是：(P) => (R)
- Function.unlift 方法将转换成偏函数

> try {} catch {} 中的catch其实就是一个偏函数。

## 15、注解（doing）

### 15.1、什么是注解

### 15.2、什么可以被注解

和Java一样：类、方法、字段、局部变量、参数；可以同时多个注解，没有顺序要求

### 15.3、注解参数

### 15.4、注解实现

### 15.5、针对Java特性的注解

### 15.6、用于优化的注解

### 15.7、用于错误和警告的注解

## 16、XML处理（doing）

## 17、 Futrue

### 17.1、在future中运行任务

### 17.2、等待结果

### 17.3、Try类

### 17.4、回调

### 17.5、组合future任务

### 17.6、其他future变换

### 17.7、Future对象中的方法

### 17.8、Promise

### 17.9、执行上下文

## 18、 类型参数

## 19、 高级特性

## 20、 解析

## 21、 隐式转换和隐式参数