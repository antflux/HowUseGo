# 理解 Go 中的布尔逻辑

布尔数据类型（`bool`）可以是两个值之一，true或false。布尔值在编程中用于进行比较和控制程序的流程。

布尔值表示与数学的逻辑分支相关的真值，它为计算机科学中的算法提供了信息。以数学家乔治布尔命名，布尔这个词总是以大写的`B`开头。

Go for Boolean中的数据类型是`bool`，所有的都是。值`true`和`false`将始终分别带有一个值`t`和`f`，因为它们是Go中的特殊值。

本教程将涵盖您需要了解`bool`数据类型如何工作的基础知识，包括布尔比较，逻辑运算符和真值表。

## [比较运算符](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#comparison-operators)

在编程中，比较运算符用于比较值，并计算为单个布尔值（true或false）。

下表显示了布尔比较运算符。

| 操作者 | 意味着什么 |
| ------ | ---------- |
| ==     | 等于       |
| ！=    | 不等于     |
| <      | 小于       |
| >      | 大于       |
| <=     | 小于或等于 |
| >=     | 大于或等于 |

为了理解这些运算符是如何工作的，让我们在Go程序中将两个整数赋给两个变量：

```go
x := 5
y := 8
```


在该示例中，由于`x`具有`5`的值，所以它小于具有`y`的值的`8`。

使用这两个变量和它们的关联值，让我们来看看上表中的运算符。在这个程序中，您将要求Go打印出每个比较运算符的计算结果是true还是false。为了帮助更好地理解这个输出，你还需要让Go打印一个字符串来显示它正在评估的内容：

```go
package main

import "fmt"

func main() {
	x := 5
	y := 8

	fmt.Println("x == y:", x == y)
	fmt.Println("x != y:", x != y)
	fmt.Println("x < y:", x < y)
	fmt.Println("x > y:", x > y)
	fmt.Println("x <= y:", x <= y)
	fmt.Println("x >= y:", x >= y)
}
```


```
Output  x == y: false
x != y: true
x < y: true
x > y: false
x <= y: true
x >= y: false
```

遵循数学逻辑，Go从表达式中评估了以下内容：

- 5（`x`）是否等于8（`y`）？假
- 5不等于8吗？真
- 5比8小吗？真
- 5是否大于8？假
- 5是否小于或等于8？真
- 5是否不小于或等于8？假

虽然这里使用的是整数，但也可以用浮点值代替。

字符串也可以与布尔运算符一起使用。它们是区分大小写的，除非您使用额外的字符串方法。

你可以看看字符串在实践中是如何比较的：

```go
Sammy := "Sammy"
sammy := "sammy"

fmt.Println("Sammy == sammy: ", Sammy == sammy)
```


```
Output  Sammy == sammy:  false
```

字符串`Sammy`不等于字符串`sammy`，因为它们并不完全相同;一个以字符串`S`开始，另一个以字符串`s`开始。但是，如果您添加另一个变量，并将其赋值为`Sammy`，则它们的计算结果将等于：

```go
Sammy := "Sammy"
sammy := "sammy"
alsoSammy := "Sammy"

fmt.Println("Sammy == sammy: ", Sammy == sammy)
fmt.Println("Sammy == alsoSammy", Sammy == alsoSammy)
```


```
Output  Sammy == sammy:  false
Sammy == alsoSammy true
```

您还可以使用其他比较运算符（包括`>`和`<`）来比较两个字符串。Go将使用字符的ASCII值按字典顺序比较这些字符串。

您还可以使用比较运算符计算布尔值：

```go
t := true
f := false

fmt.Println("t != f: ", t != f)
```


```
Output  t != f:  true
```

前面的代码块评估了`true`不等于`false`。

注意两个操作符`=`和`==`之间的区别。

```go
x = y   // Sets x equal to y
x == y  // Evaluates whether x is equal to y
```


第一个`=`是赋值运算符，它将一个值设置为另一个值。第二个，`==`，是一个比较运算符，将评估两个值是否相等。

## [逻辑运算符](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#logical-operators)

有两个逻辑运算符用于比较值。它们将表达式求值为布尔值，返回`true`或`false`。这些运算符是`&&`、`||`和`!`，定义如下：

- &&（`x && y`）是`and`操作符。如果两个陈述都为真，则为真。
- || （`x || y`）是`or`操作符。如果至少有一个陈述为真，则为真。
- ！（`!x`）是`not`操作符。只有当陈述为假时，它才为真。

逻辑运算符通常用于评估两个或多个表达式是否为真。例如，它们可以用来确定成绩是否合格以及学生是否注册在课程中，如果两种情况都是真的，那么学生将在系统中被分配一个成绩。另一个例子是根据用户是否有商店信用或在过去6个月内是否进行了购买来确定用户是否是在线商店的有效活跃客户。

为了理解逻辑运算符是如何工作的，让我们评估三个表达式：

```go
fmt.Println((9 > 7) && (2 < 4))   // Both original expressions are true
fmt.Println((8 == 8) || (6 != 6)) // One original expression is true
fmt.Println(!(3 <= 1))            // The original expression is false
```


```
Output  true
true
true
```

在第一种情况下，`fmt.Println((9 > 7) && (2 < 4))`，由于使用了`9 > 7`运算符，因此`2 < 4`和`and`都需要求值为true。

在第二种情况下，`fmt.Println((8 == 8) || (6 != 6))`，由于`8 == 8`的计算结果为true，因此`6 != 6`的计算结果为false并没有什么不同，因为使用了`or`运算符。如果你使用了`and`操作符，这将评估为false。

在第三种情况下，`fmt.Println(!(3 <= 1))`，`not`运算符否定`3 <=1`返回的假值。

让我们用浮点数代替整数，并瞄准错误的评估：

```go
fmt.Println((-0.2 > 1.4) && (0.8 < 3.1))  // One original expression is false
fmt.Println((7.5 == 8.9) || (9.2 != 9.2)) // Both original expressions are false
fmt.Println(!(-5.7 <= 0.3))               // The original expression is true
```


在这个例子中：

- `and`必须至少有一个false表达式计算为false。
- `or`必须让两个表达式的值都为false。
- `!`的内部表达式必须为true，新表达式才能计算为false。

如果这些结果对你来说似乎不清楚，请查看一些[真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#truth-tables)以进一步澄清。

您也可以使用`&&`、`||`和`!`编写复合语句：

```go
!((-0.2 > 1.4) && ((0.8 < 3.1) || (0.1 == 0.1)))
```


先来看看最内在的表达式：`(0.8 < 3.1) || (0.1 == 0.1)`。这个表达式的计算结果是`true`，因为两个数学语句都是`true`。

接下来，Go获取返回值`true`并将其与下一个内部表达式`(-0.2 > 1.4) && (true)`组合。此示例返回`false`，因为数学语句`-0.2 > 1.4`为false，并且（`false`）和（`true`）返回`false`。

最后，我们有一个外部表达式：`!(false)`，它的计算结果是`true`，所以如果我们把这个语句打印出来，最终返回的值是：

```
Output  true
```

逻辑运算符`&&`、`||`和`!`计算表达式并返回布尔值。

## [真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#truth-tables)

关于数学的逻辑分支有很多东西要学，但你可以有选择地学习其中的一些，以提高你在编程时的算法思维。

以下是比较运算符`==`以及逻辑运算符`&&`、`||`和`!`中的每一个的真值表。虽然你可以推理出它们，但记住它们也是有帮助的，因为这可以使你的编程决策过程更快。

### [`==`（equal）真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#equal-truth-table)

| X    | ==   | 和   | 返回 |
| ---- | ---- | ---- | ---- |
| 真   | ==   | 真   | 真   |
| 真   | ==   | 假   | 假   |
| 假   | ==   | 真   | 假   |
| 假   | ==   | 假   | 真   |

### [`&&`（和）真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#and-truth-table)

| X    | 和   | 和   | 返回 |
| ---- | ---- | ---- | ---- |
| 真   | 和   | 真   | 真   |
| 真   | 和   | 假   | 假   |
| 假   | 和   | 真   | 假   |
| 假   | 和   | 假   | 假   |

### [`||`（或）真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#or-truth-table)

| X    | 或   | 和   | 返回 |
| ---- | ---- | ---- | ---- |
| 真   | 或   | 真   | 真   |
| 真   | 或   | 假   | 真   |
| 假   | 或   | 真   | 真   |
| 假   | 或   | 假   | 假   |

### [`!`（非）真值表](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#not-truth-table)

| 不|X | 返回| | - | - | - | - | | 不|真正|虚假 | | 不|虚假|真正 |

真值表是逻辑中常用的数学表，在计算机编程中构造算法（指令）时需要记住。

## [使用布尔运算符进行流控制](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#using-boolean-operators-for-flow-control)

要以流控制语句的形式控制程序的流和结果，可以使用条件后跟子句。

一个条件的计算结果是一个布尔值true或false，表示在程序中做出决定的点。也就是说，一个条件会告诉你某个东西的计算结果是真还是假。

子句是跟在条件后面的代码块，它指示程序的结果。也就是说，它是构造“如果`x`是`true`，那么就这样做”的“这样做”部分。

下面的代码块显示了一个比较运算符与条件语句协同工作以控制Go程序流的示例：

```go
if grade >= 65 {                 // Condition
	fmt.Println("Passing grade") // Clause
} else {
	fmt.Println("Failing grade")
}
```


这个程序将评估每个学生的成绩是及格还是不及格。如果学生的成绩为`83`，则第一条语句的计算结果将为`true`，并将触发`Passing grade`的print语句。如果一个学生的成绩是`59`，那么第一个语句的计算结果将是`false`，所以程序将继续执行与else表达式绑定的print语句：`Failing grade`。

布尔运算符通过流控制语句提供可用于决定程序最终结果的条件。

## [结论](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go#conclusion)

本教程介绍了属于布尔类型的比较和逻辑运算符，以及真值表和使用布尔值进行程序流控制。