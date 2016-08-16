# Scala破冰之旅

> 即使水墨丹青，何以绘出半妆佳人。

`Scala`是一门优雅而又复杂的程序设计语言，初学者很容易陷入细节而迷失方向。这也给我的写作带来了挑战，如果从基本的控制结构，再深入地介绍高级的语法结构，难免让人生厌。

为此，本文另辟蹊径，尝试通过一个简单有趣的例子，概括性地介绍`Scala`常见的语言特性。它犹如一个迷你版的`Scala`教程，带领大家一起领略`Scala`的风采。

### 问题的提出

有一名体育老师，在某次离下课还有五分钟时，决定玩一个游戏。此时有`100`名学生在上课，游戏的规则如下：

1. 老师先说出三个不同的特殊数(都是个位数)，比如`3, 5, 7`；让所有学生拍成一队，然后按顺序报数；
2. 学生报数时，如果所报数字是「第一个特殊数(`3`)」的倍数，那么不能说该数字，而要说`Fizz`；如果所报数字是「第二个特殊数(`5`)」的倍数，要说`Buzz`；如果所报数字是「第三个特殊数(`7`)」的倍数，要说`Whizz`。
3. 学生报数时，如果所报数字同时是「两个特殊数」的倍数，也要特殊处理。例如，如果是「第一个(`3`)」和「第二个(`5`)」特殊数的倍数，那么也不能说该数字，而是要说`FizzBuzz`。以此类推，如果同时是三个特殊数的倍数，那么要说`FizzBuzzWhizz`。
4. 学生报数时，如果所报数字包含了「第一个特殊数」，那么也不能说该数字，而是要说`Fizz`。例如，要报`13`的同学应该说`Fizz`。
5. 如果数字中包含了「第一个特殊数」，需要忽略规则`2`和`3`，而使用规则`4`。例如要报`35`，它既包含`3`，同时也是`5`和`7`的倍数，要说`Fizz`，而不能说`BuzzWhizz`；
6. 否则，就要说对应的数字。

##### 形式化

以`3, 5, 7`为例，该问题可形式化地描述为：

```scala
r1: times(3) => Fizz || 
    times(5) => Buzz ||
    times(7) => Whizz

r2: times(3) && times(5) && times(7) => FizzBuzzWhizz ||
    times(3) && times(5) => FizzBuzz  ||
    times(3) && times(7) => FizzWhizz ||
    times(5) && times(7) => BuzzWhizz

r3: contains(3) => Fizz

rd: others => string of others

spec: r3 || r2 || r1 || rd
```

其中，`times(3) => Fizz`表示：当输入为`3`的倍数时，输出`Fizz`，其他以此类推。

##### 



### 建立测试环境

首先建立测试框架，建立反馈系统。这里使用`scalatest`的测试框架，它也是作者最喜欢的测试框架之一。

```scala
package scalaspec.fizzbuzz

import org.scalatest.Matchers._
import org.scalatest._

class RuleSpec extends FunSpec {
  describe("World") {
    it ("should not be work" ) {
      true should be(false)
    }
  }
}
```

运行测试用例，用于验证环境。

### 第一个测试用例

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz").apply(3 * 2) should be("Fizz")
}
```

它先建立了一个规则：`new Times(3, "Fizz")`，表示如果是`3`的倍数，则输出`Fizz`。此时，如果输入数字`3*2`，断言预期的结果为`Fizz`。

##### 如果使用Java

为了快速通过测试，可以做简单实现。如果使用`Java`，`Times`实现大致如下。

```java
public class Times(int n, String word) {
  private final int n;
  private final String word;

  public Times(int n, String word) {
    this.n = n;
    this.word = word;
  }

  public apply(int m) {
    return word;
  }
}
```

##### 构造参数

从上述`Java`实现可以看出，当定义一个私有字段时，需要构造函数对它进行初始化。类似重复的「样板代码」在`Scala`中，可以在「主构造函数」中使用「构造参数」代替，彻底消除重复代码。

```scala
class Times(n: Int, word: String) {
  def apply(m: Int): String = word
}
```

##### 类型的后缀修饰

`Scala`将类型的修饰放在后面，以便实现风格的「一致性」，包括：

- 变量的类型修饰
- 函数返回值的类型修饰

```scala
def apply(m: Int): String = word
```

##### 类型推演

关于`Scala`类型推演，需要注意几点：

- 函数原型后面不能略去`=`

`apply`函数原型后面的`=`不能略去，因为`Scala`将函数也看成普通的表达式。否则会限制函数的类型推演的能力，编译器将一律推演函数返回值类型为`Unit`(等价于`Java`中的`void`)。

- 函数返回值常常略去`return`

例如，`def apply(m: Int) = { return word }`将产生编译错误，其需要明确地声明函数返回值的类型。所以，显式的`return`语句的使用得不偿失，除非`return`用于明确的提前中断。

- 借助于类型推演的机制，变量、函数返回值的类型都可以略去；但是，当逻辑较为复杂时，代码的表达力将大打折扣

```scala
val i: Int = 0  // 类型修饰显得冗余
```

事实上，此处也可以略去`apply`方法的返回值的类型修饰，它依然能够推演为`String`类型。

```scala
def apply(m: Int) = word
```

##### apply方法

`apply`方法是一个特殊的方法，它可以简化方法调用的表现形式，使其行为更贴近函数的语义。在特殊的场景下，能够改善代码的表达力。

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz").apply(3 * 2) should be("Fizz")
}
```

等价于：

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz")(3 * 2) should be("Fizz")
}
```

### 实现Times

至此，测试通过了，但`apply`实现被写死了。因为`Times`的逻辑较为简单，可以快速实现它。

```scala
class Times(n: Int, word: String) {
  def apply(m: Int): String = 
    if (m % n == 0) word else ""
}
```

##### 万物皆是对象

`Scala`并没有像`Java`一样，针对「基本类型」(例如`int`)，「数组类型」(例如`int[]`)定义特殊的语法，它将世间万物都看成对象。

其中，`m % n`等价于`m.%(n)`，而`%`只不过是`Int`的一个普通方法而已。

```scala
package scala

final abstract class Int private extends AnyVal {
  def %(x: Int): Int
  ...
}
```

注意，`Int`的实现由编译器完成，后续章节将讲述`Int`的修饰语法。

##### 面向表达式

`Scala`是一门面向表达式的语言，它所有的程序结构都具有值，包括`if-else`，函数调用等。其中，`if (m % n == 0) word else ""`类似于`Java`中的三元表达式：`m % n == 0 ? word : ""`。

因为两者功能重复，因此`Scala`并没有提供三元表达式的特性。

### 使用样本类

可以将`Times`设计为「样本类」。

```scala
case class Times(n: Int, word: String) {
  def apply(m: Int): String =
    if (m % n == 0) word else ""
}
```

当构造一个`Times`实例时，可以使用其「伴生对象」提供的工厂方法，从而略去`new`关键字，简化代码实现。

```scala
it ("times(3) -> fizz" ) {
  Times(3, "Fizz")(3 * 2) should be("Fizz")
}
```

##### 揭秘样本类

使用`case class`定义的类称为「样本类」，它默认具有字段的`Getter`方法，并天然地拥有`equals, hashCode`等方法。

另外，「样本类」在其「伴生对象」中自动生成`apply`的工厂方法。当生产对象时，可以略去`new`关键字，使得语义更加简洁。

也就是说，`case class Times...`等价于：

```scala
class Times(val n: Int, val word: String) {
  def apply(m: Int): String =
    if (m % n == 0) word else ""

  override def equals(obj: Any): Boolean = ???
  override def hashCode(): Int = ???
  ...
}

object Times {
  def apply(n: Int, word: String) = new Times(n, word)
}
```

>
> 在本书中，除非特别说明，否则`???`表示函数实现的占位表示，仅仅为了方便举例。
> 
> 其中，`???`定义在`scala.Predef`中；因为`Predef`的所有成员被编译器默认导入，它对所有程序公开。
> 
> ```scala
> def ??? = throw new NotImplementedError
> ```
> 

##### 伴生对象

`object Times`常常称为`class Times`的「伴生对象」。事实上，「伴生对象」中的方法，类似于`Java`中的`static`方法。但`Scala`摒弃了`static`的关键字，将面向对象的语义进行统一。

如果用`Java`设计`case class Times`，其实现类似于：

```java
public class Times {
  private final int n;
  private final String word;

  public Times(int n, String word) {
    this.n = n;
    this.word = word;
  }
  
  public apply(int m) {
    return word;
  }
  
  // Getter方法
  public int n() {
    return n;
  }
  
  public String word() {
    return word;
  }
  
  // 自动生成equals, hashCode方法  
  @Override
  public boolean equals(Object obj) {
    ...
  }
  
  @Override
  public int hashCode() {
    ...
  }
  
  // 静态工厂方法
  public static Times apply(int n, String word) {
    return new Times(n, word);
  }
  
  ...
}
```

### 实现Contains

有了`Times`实现的基础，可以很轻松地实现`Contains`的测试用例。

```scala
it ("contains(3) -> fizz" ) {
  Contains(3, "Fizz")(13) should be("Fizz")
}
```

依次类推，`Contains`可以快速实现为：

```scala
case class Contains(n: Int, word: String) {
  def apply(m: Int): String =
    if (m.toString.contains(n.toString)) word else ""
}
```

恭喜，测试通过了。

##### 省略括号

`m.toString`等价于`m.toString()`。按照惯例，如果函数没有副作用，则可以略去小括号；相反，如果产生副作用，则显式地加上小括号用于警示。

如果函数定义时就没有使用小括号，用于表达函数无副作用；此时用户不能画蛇添足，添加多余的小括号；否则与函数定义的语义相驳了。

### 实现默认规则

对于默认规则，它只是简单地将输入的数字转变为字符串表示形式。

```scala
it ("default rule" ) {
  Default()(2) should be("2")
}
```

其中，`Default`可以快速实现为：

```scala
case class Default() {
  def apply(m: Int): String = m.toString
}
```

注意，`case class Default()`，及其调用点`Default()(2)`，不能略去`()`。

### 提取抽象

至此，发现`Times, Contains, Default`都具有相同的结构，可抽象出`Rule`的概念。

```scala
trait Rule {
  def apply(n: Int): String
}
```

##### 特质的功效

此处使用`trait`定义了一个接口。事实上，`trait`不仅仅等价于`Java`的`interface`，它是`Scala`实现对象组合的重要机制。

##### 实现特质

例如`Times`实现`Rule`特质，它使用`extends Rule`语法混入该「特质」。

```scala
case class Times(n: Int, word: String) extends Rule {
  def apply(m: Int): String =
    if (m % n == 0) word else ""
}
```

以此类推，`Contains, Default`实现方式相同，不再重述。

### 实现AnyOf

接下来，实现具有两个之间具有「逻辑与」关系的复合规则。先建立一个简单的测试用例：

```scala
it ("times(3) && times(5) -> FizzBuzz" ) {
  AllOf(Times(3, "Fizz"), Times(5, "Buzz"))(3*5) should be("FizzBuzz")
}
```

为了快速通过测试，可以先打桩实现。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = "FizzBuzz"
}
```

##### 变长参数

`rules: Rule*`表示变长的`Rule`列表，表示可以向`AllOf`的构造函数传递任意多的`Rule`实例。

事实上，`rules: Rule*`的真正类型为`scala.collection.mutable.WrappedArray[Rule]`，所以`rules: Rule*`拥有普通集合类的一般特征，例如调用`map, foreach, foldLeft`等方法。

##### 快速实现`AllOf`

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = {
    var result = StringBuilder.newBuilder
    rules.foreach { r =>
      result.append(r(n))
    }
    result.toString
  }
}
```

##### 使用foldLeft

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { r => _ + r.apply(n) }
}
```

因为`r`在函数字面值中仅过一次，可以使用占位符代替。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { _ + _.apply(n) }
}
```

因为`apply`方法具有特殊的函数调用语义，可以进一步简化实现。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { _ + _(n) }
}
```

### 实现AnyOf

接下来，实现具有两个之间具有「逻辑或」关系的复合规则。先建立一个简单的测试用例：

```scala
it ("times(3) -> Fizz || times(5) -> Buzz" ) {
  AnyOf(Times(3, "Fizz"), Times(5, "Buzz"))(3*5) should be("Fizz")
}
```

为了快速通过测试，可以先打桩实现。

```scala
case class AnyOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = "Fizz"
}
```

##### 快速实现AnyOf

```scala
case class AnyOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.map(_(n))
         .filterNot(_.isEmpty)
         .headOption
         .getOrElse("")
}
```

测试用例通过了。

### 提供工厂方法

因为`Times, Contains, Default, AnyOf, AllOf`都具有相同的句法结构，是一种典型的结构性重复设计，可以通过「工厂方法」消除它们之间的重复设计。

另外，为了简单函数调用的方式，可以使用`Int => String`的一元函数代替`Rule`特质。

##### 重构测试用例

此时，可以定义一组新的测试用例集合，并使用`describe`分离用例组，并通过显示地导入所依赖的类型，与既有的用例集共存，互不干扰。

>
> 切忌删除既有的`Rule`特质，以及`Times, Contains, Default, AllOf, AnyOf`的实现，包括既有的测试用例；否则既有的测试用例失败，重构的安全网被撕破，将会让重构陷入一个极度危险的境界。
> 
> 总之，重构应该保持小步快跑的基本原则。
> 

按照`TDD`的规则，可以小步地，安全地逐一驱动实现各个工厂方法。

```scala
class RuleSpec extends FunSpec {
  ...
  describe("Rule using factory method") {
    import Rule._
    it ("times(3) -> fizz" ) {
      times(3, "Fizz")(3 * 2) should be("Fizz")
    }
  }
}
```

##### 实现工厂

`times`的工厂方法也较容易实现，可以通过搬迁`Times`的逻辑至此即可。

```scala
object Rule {
  def times(n: Int, word: String): Int => String =
    m => if (m % n == 0) word else ""
}
```

至此，`times`实现通过测试。

##### 小步快跑

以此类推，通过小步地`TDD`的微循环，将其他工厂方法驱动实现出来。

```scala
class RuleSpec extends FunSpec {
  ...

  describe("Rule using factory method") {
    import Rule._

    it ("times(3) -> fizz" ) {
      times(3, "Fizz")(3 * 2) should be("Fizz")
    }

    it ("contains(3) -> fizz" ) {
      contains(3, "Fizz")(13) should be("Fizz")
    }

    it ("default rule" ) {
      default(2) should be("2")
    }

    it ("times(3) && times(5) -> FizzBuzz" ) {
      anyof(times(3, "Fizz"), times(5, "Buzz"))(3*5) should be("FizzBuzz")
    }

    it ("times(3) -> Fizz || times(5) -> Buzz" ) {
      anyof(times(3, "Fizz"), times(5, "Buzz"))(3*5) should be("Fizz")
    }
  }
}
```

`Rule`伴生对象中的工厂方法实现如下。

```scala
object Rule {
  def times(n: Int, word: String): Int => String =
    m => if (m % n == 0) word else ""
    
  def contains(n: Int, word: String): Int => String = 
    m => if (m.toString.contains(n.toString)) word else ""
    
  def default: Int => String =
    m => m.toString
  
  def anyof(rules: (Int => String)*): Int => String = 
    m => rules.foldLeft("") { _ + _(m) }
    
  def allof(rules: (Int => String)*): Int => String = 
    m => rules.map(_(m))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")
}
```

恭喜，通过所有测试。此时可以安全地删除`Times, Contains, Default, AnyOf, AllOf`，`Rule`特质，以及相关的遗留的测试用例了。

##### 类型别名

可以对`Int => String`定义「类型别名」，消除类型的重复定义。

```scala
object Rule {
  type Rule = Int => String

  def times(n: Int, word: String): Rule =
    m => if (m % n == 0) word else ""

  def contains(n: Int, word: String): Rule =
    m => if (m.toString.contains(n.toString)) word else ""

  def default: Rule =
    m => m.toString

  def anyof(rules: Rule*): Rule =
    m => rules.foldLeft("") { _ + _(m) }

  def allof(rules: Rule*): Rule =
    m => rules.map(_(m))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")
}
```

至此，设计已经较为干净了。但发现`times, contains, default`之间存在微妙的重复结构。它们各自拥有隐晦的「匹配规则」，当匹配成功时，执行相应的「转换规则」。

其中，`default`的匹配规则、转换规则都比较特殊；因为它总是匹配成功，转换时简单地讲数字转换为字符串表示的形式。。

### 提取匹配器

先提取抽象的「匹配器」概念：`Matcher`。事实上，`Matcher`是一个「一元函数」，入参为`Int`，返回值为`Boolean`，是一种典型的「谓词」。

> 从`OO`的角度看，`always`是一个典型的`Null Object`实现模式。

```scala
object Matcher {
  type Matcher = Int => Boolean

  def times(n: Int): Matcher = _ % n == 0
  def contains(n: Int): Matcher = _.toString.contains(n.toString)
  def always(bool: Boolean): Matcher = _ => bool
}
```

### 执行器：`Action`

然后再提取抽象的「执行器」概念：`Action`。事实上，`Action`也是一个「一元函数」，入参为`Int`，返回值为`String`。其本质类似于`map`操作，将定义域映射到值域。

> 从`OO`的角度看，`nop`也是一个典型的`Null Object`实现模式。

```scala
object Action {
  type Action = Int => String

  def to(str: String): Action = _ => str
  def nop: Action = _.toString
}
```

### 原子操作

至此，可以提取`times, contains, default`三者之间的共公抽象：`atom`。

```scala
def atom(matcher: => Matcher, action: => Action): Rule =
  m => if (matcher(m)) action(m) else ""
```

它表示的语义为：给定一个整数`m`，如果与`Matcher`匹配成功，则执行`Action`转换；否则返回空字符串。

##### 新建一组用例集合

此时，新建一组用例集合，并使用`atom`的原子接口，并使用`describe`隔离新老用例集，显式地`import`所依赖的类型，保证既有测试用例可用。

> 
> 在`Rule.atom, Matcher, Action`可运行之前，切忌删除`Rule`中既有的`times, contains, default`，及其相应的测试用例。
>  

```scala
class RuleSpec extends FunSpec {
  ...
  describe("using atom rule") {
    import Rule.{anyof, anyof, atom}
    import Matcher._
    import Action._

    val r1_3 = atom(times(3), to("Fizz"))
    val r1_5 = atom(times(5), to("Buzz"))

    it ("times(3) -> fizz" ) {
      r1_3(3 * 2) should be("Fizz")
    }

    val r3 = atom(contains(3), to("Fizz"))

    it ("contains(3) -> fizz" ) {
      r3(13) should be("Fizz")
    }
    
    val rd = atom(always(true), nop)

    it ("default rule" ) {
      rd(2) should be("2")
    }

    it ("times(3) && times(5) -> FizzBuzz" ) {
      anyof(r1_3, r1_5)(3*5) should be("FizzBuzz")
    }

    it ("times(3) -> Fizz || times(5) -> Buzz" ) {
      anyof(r1_3, r1_5)(3*5) should be("Fizz")
    }
  }
}
```

测试用例通过。

### 规则库

> Composition Everywhere

此时，可以安全地删除`Rule`中`times, contains, default`的实现，及其遗留的测试用例集。`Rule`最终实现为：

```scala
object Rule {
  type Rule = Int => String

  import Matcher.Matcher
  import Action.Action

  def atom(matcher: => Matcher, action: => Action): Rule =
    n => if (matcher(n)) action(n) else ""

  def anyof(rules: Rule*): Rule =
    n => rules.map(_(n))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")

  def allof(rules: Rule*): Rule =
    n => rules.foldLeft("") { _ + _(n) }
}
```

`Rule`是`FizzBuzzWhizz`最核心的抽象，也是设计的灵魂所在。从语义上`Rule`分为`2`种基本类型，并且两者之间形成了隐式的「树型」结构，体现了「组合式设计」的强大威力。

- 原子规则：`atom`
- 复合规则: `anyof, anyof`

`Rule`也是一个「一元函数」，入参为`Int`，返回值为`String`。其中，`def atom(matcher: => Matcher, action: => Action)`的入参使用`by-name`的「惰性求值」特性。

### 完备用例集

针对于`FizzBuzzWhizz`问题，以`3, 5, 7`为例，其完备的用例集可以如下描述。此处使用表格驱动的方式组织用例，消除大量的重复代码，并改善其表达力。

```scala
import org.scalatest._
import prop._

class RuleSpec extends PropSpec with TableDrivenPropertyChecks with Matchers {
  import Rule._
  import Matcher._
  import Action._
  
  val spec = {
    val r1_3 = atom(times(3), to("Fizz"))
    val r1_5 = atom(times(5), to("Buzz"))
    val r1_7 = atom(times(7), to("Whizz"))

    val r1 = anyof(r1_3, r1_5, r1_7)

    val r2 = anyof(
      allof(r1_3, r1_5, r1_7),
      allof(r1_3, r1_5),
      allof(r1_3, r1_7),
      allof(r1_5, r1_7))

    val r3 = atom(contains(3), to("Fizz"))
    val rd = atom(always(true), nop);

    anyof(r3, r2, r1, rd)
  }

  val specs = Table(
    ("n",         "expect"),
    (3,           "Fizz"),
    (5,           "Buzz"),
    (7,           "Whizz"),
    (3 * 5,       "FizzBuzz"),
    (3 * 7,       "FizzWhizz"),
    ((5 * 7) * 2, "BuzzWhizz"),
    (3 * 5 * 7,   "FizzBuzzWhizz"),
    (13,          "Fizz"),
    (35/*5*7*/,   "Fizz"),
    (2,           "2")
  )

  property("fizz buzz whizz") {
    forAll(specs) { spec(_) should be (_) }
  }
}
```

### 语义模型

归纳上述设计，可以得到`FizzBuzzWhizz`的语义模型。

```scala
Rule:    Int => String
Matcher: Int => Boolean
Action:  Int => String
```

其中，`Rule`存在三种基本的类型：

```scala
Rule: atom | allof | anyof
```

三者之间构成了隐式的「树型结构」。

```scala
atom: (Matcher, Action) => String
allof: rule1 && rule2 ... 
anyof: rule1 || rule2 ... 
```

### 总结

本文通过对`FizzBuzzWhizz`的小游戏的设计和实现，首先尝试使用`Scala`的面向对象实现，然后采用函数式的设计；采用`TDD`的方式，演进式地完成功能的实现。

中间也曾遇到了「样本类」，「类型别名」，「伴生对象」等常用的技术。相信经过本文的实践，你应该对`Scala`有了一个大体的影响和感觉，接下来让我们开启`Scala`的愉快之旅吧。
