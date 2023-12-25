# 如何在 Go 中使用变量和常量

*变量*是需要掌握的重要编程概念。它们是代表您在程序中使用的值的符号。

本教程将介绍一些变量基础知识以及在您创建的 Go 程序中使用它们的最佳实践。

## 理解变量

用技术术语来说，变量将存储位置分配给与符号名称或标识符相关的值。我们使用变量名称来引用计算机程序中存储的值。

我们可以将变量视为带有名称的标签，并将其与值联系起来。

![Go 中的变量](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/variable1-20231202120320767.png)

假设我们有一个整数 ，`1032049348`并且希望将其存储在变量中，而不是一遍又一遍地连续重新输入长数字。为了实现这一点，我们可以使用一个容易记住的名称，例如变量`i`。要将值存储在变量中，我们使用以下语法：

```go
i := 1032049348
```

我们可以将此变量视为与值绑定的标签。

![Go 变量示例](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/variable2-20231202120321127.png)

标签`i`上写有变量名称，并与整数值相关联`1032049348`。

该短语`i := 1032049348`是一个声明和赋值语句，由以下几个部分组成：

- 变量名 ( `i`)
- 短变量声明赋值 ( `:=`)
- 与变量名称关联的值 ( `1032049348`)
- `int`Go ( )推断的数据类型

稍后我们将在下一节中看到如何显式设置类型。

这些部分共同组成了将变量设置`i`为等于整数值的语句`1032049348`。

一旦我们将变量设置为等于某个值，我们就*初始化*或创建该变量。完成此操作后，我们就可以使用变量而不是值。

一旦我们设置`i`等于 的值`1032049348`，我们就可以用它`i`来代替整数，所以让我们把它打印出来：

```go
package main

import "fmt"

func main() {
 i := 1032049348
 fmt.Println(i)
}
```

```
Output1032049348
```

我们还可以使用变量快速轻松地进行数学计算。使用，我们可以使用以下语法`i := 1032049348`减去整数值：`813`

```go
fmt.Println(i - 813)
```

```
Output 1032048535
```

在此示例中，Go 为我们进行数学计算，从变量中减去 813`i`以返回 sum `1032048535`。

说到数学，变量可以设置为等于数学方程的结果。您还可以将两个数字相加并将总和的值存储到变量中`x`：

```go
x := 76 + 145
```

您可能已经注意到，这个示例看起来与代数类似。就像我们使用字母和其他符号来表示公式和方程中的数字和数量一样，变量也是表示数据类型值的符号名称。为了获得正确的 Go 语法，您需要确保变量位于任何方程的左侧。

让我们继续打印`x`：

```go
package main

import "fmt"

func main() {
 x := 76 + 145
 fmt.Println(x)
}
```

```
Output 221
```

Go 返回了该值，`221`因为变量被设置为等于和`x`的总和。`76``145`

变量可以表示任何数据类型，而不仅仅是整数：

```go
s := "Hello, World!"
f := 45.06
b := 5 > 9 // A Boolean value will return either true or false
array := [4]string{"item_1", "item_2", "item_3", "item_4"}
slice := []string{"one", "two", "three"}
m := map[string]string{"letter": "g", "number": "seven", "symbol": "&"}
```

如果打印这些变量中的任何一个，Go 将返回该变量等价的内容。让我们使用字符串`slice`数据类型的赋值语句：

```go
package main

import "fmt"

func main() {
 slice := []string{"one", "two", "three"}
 fmt.Println(slice)
}
```

```
Output [one two three]
```

我们将 的切片值分配`[]string{"one", "two", "three"}`给变量`slice`，然后使用该`fmt.Println`函数通过调用 来打印该值`slice`。

变量的工作原理是在计算机中划分出一小块内存区域，该区域接受指定的值，然后将这些值与该空间关联起来。

## 声明变量

在 Go 中，有多种方法来声明变量，并且在某些情况下，声明完全相同的变量和值的方法不止一种。

我们可以声明一个名为`i`数据类型的变量`int`而无需初始化。这意味着我们将声明一个空间来放置值，但不给它一个初始值：

```go
var i int
```

这将创建一个声明为`i`数据类型 的变量`int`。

我们可以使用 equal ( `=`) 运算符来初始化该值，如下例所示：

```go
var i int = 1
```

在 Go 中，这两种声明形式都称为*长变量声明*。

我们还可以使用*短变量声明*：

```go
i := 1
```

在本例中，我们有一个名为 的变量`i`，以及 的数据类型`int`。当我们不指定数据类型时，Go 将推断数据类型。

对于声明变量的三种方式，Go 社区采用了以下习惯用法：

- `var i int`当您不初始化变量时，使用长格式。
- `i := 1`声明和初始化时使用缩写形式 , 。
- 如果您不希望 Go 推断您的数据类型，但仍想使用短变量声明，则可以使用以下语法将值包装在您想要的类型中：

```go
i := int64(1)
```

当我们初始化值时，使用长变量声明形式在 Go 中并不被认为是惯用的：

```go
var i int = 1
```

遵循 Go 社区通常如何声明变量是一种很好的做法，以便其他人可以无缝地读取您的程序。

## 零值

所有内置类型的值都为零。任何分配的变量都是可用的，即使它从未分配过值。我们可以看到以下类型的零值：

```go
package main

import "fmt"

func main() {
 var a int
 var b string
 var c float64
 var d bool

 fmt.Printf("var a %T = %+v\n", a, a)
 fmt.Printf("var b %T = %q\n", b, b)
 fmt.Printf("var c %T = %+v\n", c, c)
 fmt.Printf("var d %T = %+v\n\n", d, d)
}
```

```
Outputvar a int =  0
var b string = ""
var c float64 = 0
var d bool = false
```

我们`%T`在`fmt.Printf`陈述中使用了动词。这告诉函数打印`data type`变量的 。

在 Go 中，因为所有值都有一个`zero`值，所以我们不能`undefined`像其他一些语言那样有值。例如，[`boolean`](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#booleans)某些语言中的 a 可以是`undefined`、`true`、 或`false`，这允许对`three`变量进行状态。在 Go 中，我们不能有超过`two`布尔值的状态。

## 命名变量：规则和风格

变量的命名非常灵活，但需要记住一些规则：

- 变量名只能是一个单词（如没有空格）。
- 变量名称只能由字母、数字和下划线 ( ) 组成`_`。
- 变量名不能以数字开头。

遵循这些规则，让我们看看有效和无效的变量名称：

| 有效的     | 无效的      | 为什么无效       |
| ---------- | ----------- | ---------------- |
| `userName` | `user-name` | 不允许使用连字符 |
| `name4`    | `4name`     | 不能以数字开头   |
| `user`     | `$user`     | 不能使用符号     |
| `userName` | `user name` | 不能超过一个字   |

此外，在命名变量时请记住它们区分大小写。这些名称`userName`、`USERNAME`、`UserName`、 和`uSERnAME`都是完全不同的变量。最佳实践是避免在程序中使用类似的变量名称，以确保您和您的协作者（当前和未来）都能保持变量的正确性。

虽然变量区分大小写，但变量首字母的大小写在 Go 中具有特殊含义。如果变量以大写字母开头，则可以在声明它的包外部访问该变量 (或`exported`)。如果变量以小写字母开头，则它仅在声明它的包中可用。

```go
var Email string
var password string
```

`Email`以大写字母开头，可以被其他包访问。`password`以小写字母开头，只能在声明它的包内访问。

在 Go 中使用非常简洁（或短）的变量名是很常见的。`userName`如果要在使用和for 变量之间进行选择`user`，通常选择`user`。

范围对于变量名的简洁性也起着重要作用。规则是变量存在的范围越小，变量名越小：

```go
names := []string{"Mary", "John", "Bob", "Anna"}
for i, n := range names {
 fmt.Printf("index: %d = %q\n", i, n)
}
```

我们在更大的范围内使用变量`names`，因此通常会给它一个更有意义的名称，以帮助记住它在程序中的含义。但是，我们会在下一行代码中立即使用`i`和`n`变量，然后不再使用它们……因此，阅读代码的人不会对变量的使用位置或含义感到困惑。

接下来，让我们介绍一下有关可变样式的一些注意事项。对于多单词名称，其风格是使用`MixedCaps`or`mixedCaps`，而不是下划线

| 传统风格    | 不拘一格的风格 | 为什么非传统                   |
| ----------- | -------------- | ------------------------------ |
| `userName`  | `user_name`    | 下划线不是常规的               |
| `i`         | `index`        | `i`比`index`更好，因为它更短 |
| `serveHTTP` | `serveHttp`    | 首字母应大写                  |

关于风格最重要的是保持一致，并且你工作的团队也同意这种风格。

## 重新分配变量

正如“变量”一词所暗示的那样，我们可以轻松地更改 Go 变量。这意味着我们可以通过重新分配将不同的值与先前分配的变量连接起来。能够重新分配是很有用的，因为在整个程序过程中，我们可能需要接受用户生成的值到已经初始化的变量中。我们可能还需要将分配更改为先前定义的内容。

当处理其他人编写的大型程序并且不清楚已经定义了哪些变量时，知道我们可以轻松地重新分配变量会很有用。

让我们将 的值分配给类型`76`为 的变量，然后为其分配一个新值：`i``int``42`

```go
package main

import "fmt"

func main() {
 i := 76
 fmt.Println(i)

 i = 42
 fmt.Println(i)
}
```

```
Output 76
42
```

这个例子表明，我们可以先给变量赋值`i`一个整数，然后再重新给这个变量`i`赋值，这次赋值为`42`。

**注意：**当声明**和**初始化变量时，可以使用`:=`，但是，当您想简单地更改已声明的变量的值时，只需使用等于运算符（`=`）。

因为 Go 是一种`typed`语言，所以我们不能将一种类型分配给另一种类型。例如，我们不能将值分配`"Sammy"`给类型的变量`int`：

```go
i := 72
i = "Sammy"
```

尝试互相分配不同的类型将导致编译时错误：

```
Outputcannot use "Sammy" (type string) as type int in assignment
```

Go 不允许我们多次使用一个变量名：

```go
var s string
var s string
```

```
Outputs redeclared in this block
```

如果我们尝试对同一变量名多次使用短变量声明，我们也会收到编译错误。这可能是错误发生的，因此了解错误消息的含义会很有帮助：

```go
i := 5
i := 10
```

```
Outputno new variables on left side of :=
```

与变量声明类似，考虑变量的命名将提高您和其他人将来重新访问程序时的可读性。

## 多重赋值

Go 还允许我们为同一行中的多个变量分配多个值。这些值中的每一个都可以具有不同的数据类型：

```go
j, k, l := "shark", 2.05, 15
fmt.Println(j)
fmt.Println(k)
fmt.Println(l)
```

```
Output shark
2.05
15
```

在此示例中，变量`j`被分配给字符串`"shark"`，变量`k`被分配给浮点`2.05`，变量`l`被分配给整数`15`。

这种将多个变量分配给一行中的多个值的方法可以减少代码中的行数。然而，重要的是不要因为更少的代码行而牺牲可读性。

## 全局变量和局部变量

在程序中使用变量时，记住*变量范围*很重要。变量的作用域是指可以从给定程序的代码中访问该变量的特定位置。这就是说，并非所有变量都可以从给定程序的所有部分访问 - 有些变量是全局变量，有些变量是局部变量。

全局变量存在于函数之外。局部变量存在于函数内。

让我们看一下全局变量和局部变量的作用：

```go
package main

import "fmt"


var g = "global"

func printLocal() {
 l := "local"
 fmt.Println(l)
}

func main() {
 printLocal()
 fmt.Println(g)
}
```

```
Output local
global
```

这里我们用来`var g = "global"`在函数外部创建一个全局变量。然后我们定义函数`printLocal()`。在函数内部，`l`分配了一个称为 local 的变量，然后将其打印出来。程序通过调用`printLocal()`然后打印全局变量来结束`g`。

因为`g`是全局变量，所以我们可以在`printLocal()`. 让我们修改之前的程序来做到这一点：

```go
package main

import "fmt"


var g = "global"

func printLocal() {
 l := "local"
 fmt.Println(l)
 fmt.Println(g)
}

func main() {
 printLocal()
 fmt.Println(g)
}
```

```
Outputlocal
global
global
```

我们首先声明一个全局变量`g`, `var g = "global"`。在`main`函数中，我们调用函数`printLocal`，它声明了一个局部变量`l`并将其打印出来，`fmt.Println(l)`。然后，`printLocal`打印出全局变量`g`, `fmt.Println(g)`。尽管`g`未在 中定义`printLocal`，但仍然可以访问它，因为它是在全局范围内声明的。最后，该函数也`main`打印出来。`g`

现在让我们尝试在函数外部调用局部变量：

```go
package main

import "fmt"

var g = "global"

func printLocal() {
 l := "local"
 fmt.Println(l)
}

func main() {
 fmt.Println(l)
}
```

```
Output undefined: l
```

我们不能在分配局部变量的函数之外使用局部变量。如果您尝试这样做，您将`undefined`在编译时收到错误。

让我们看另一个例子，其中全局变量和局部变量使用相同的变量名：

```go
package main

import "fmt"

var num1 = 5

func printNumbers() {
 num1 := 10
 num2 := 7  

 fmt.Println(num1)
 fmt.Println(num2)
}

func main() {
 printNumbers()
 fmt.Println(num1)
}
```

```
Output 10
7
5
```

在这个程序中，我们声明了该`num1`变量两次。首先，我们`num1`在全局范围内声明 ，并再次在函数`var num1 = 5`的局部范围内声明。当我们从程序中打印时，我们会看到打印出来的值。这是因为只能看到全局变量声明。但是，当我们从函数中打印出来时，它会看到本地声明，并将打印出 的值。即使创建了一个名为 的新变量并为其分配了值，它也不会影响值为的全局实例。`printNumbers``num1 := 10``num1``main``5``main``num1``printNumbers``10``printNumbers``num1``10``num1``5`

使用变量时，您还需要考虑程序的哪些部分需要访问每个变量；相应地采用全局或局部变量。在 Go 程序中，您会发现局部变量通常更为常见。

## 常量

常量就像变量一样，只不过一旦声明就不能修改。常量对于定义将在程序中多次使用但不能更改的值很有用。

例如，如果我们想声明购物车系统的税率，我们可以使用一个常量，然后计算程序的不同区域的税收。在未来的某个时刻，如果税率发生变化，我们只需在程序中的一个位置更改该值即可。如果我们使用变量，我们可能会意外地更改程序中某处的值，从而导致计算不正确。

要声明常量，我们可以使用以下语法：

```go
const shark = "Sammy"
fmt.Println(shark)
```

```
Output Sammy
```

如果我们尝试在声明后修改常量，我们将收到编译时错误：

```
Outputcannot assign to shark
```

常量可以是`untyped`. 这在处理整数类型数据等数字时非常有用。如果常量为`untyped`，则显式转换它，而`typed`常量则不会。让我们看看如何使用常量：

```go
package main

import "fmt"

const (
 year     = 365
 leapYear = int32(366)
)

func main() {
 hours := 24
 minutes := int32(60)
 fmt.Println(hours * year)    
 fmt.Println(minutes * year)   
 fmt.Println(minutes * leapYear)
}
```

```
Output 8760
21900
21960
```

如果您用类型声明常量，则该常量将是该类型。这里，当我们声明常量时`leapYear`，我们将其定义为数据类型`int32`。因此它是一个`typed`常量，这意味着它只能对`int32`数据类型进行操作。我们声明的常量`year`没有类型，所以它被认为是`untyped`。因此，您可以将它与任何整数数据类型一起使用。

当`hours`被定义时，它*推断*它是类型，`int`因为我们没有显式地给它一个类型，`hours := 24`。当我们声明时`minutes`，我们明确地将其声明为`int32`, `minutes := int32(60)`。

现在让我们详细了解一下每个计算及其原理：

```go
hours * year
```

在本例中，`hours`是一个`int`，并且`years`是*无类型的*。当程序编译时，它会显式转换`years`为`int`，从而允许乘法运算成功。

```go
minutes * year
```

在本例中，`minutes`是一个`int32`，并且`year`是*无类型的*。当程序编译时，它会显式转换`years`为`int32`，从而允许乘法运算成功。

```go
minutes * leapYear
```

在本例中，`minutes`是`int32`，`leapYear`是 的*类型*常量`int32`。这次编译器无需执行任何操作，因为两个变量已经是同一类型。

如果我们尝试将两种兼容`typed`和不兼容的类型相乘，程序将无法编译：

```go
fmt.Println(hours * leapYear)
```

```go
Output invalid operation: hours * leapYear (mismatched types int and int32)
```

在本例中，`hours`被推断为`int`，并被`leapYear`显式声明为`int32`。因为 Go 是一种类型语言，所以 an`int`和 an`int32`不兼容数学运算。要将它们相乘，您需要[将 1 转换为 a`int32`或 an`int`](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-number-types)。

## 结论

在本教程中，我们回顾了 Go 中变量的一些常见用例。变量是编程的重要构建块，充当代表我们在程序中使用的数据类型值的符号。
