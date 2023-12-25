# 如何在 Go 中转换数据类型

### [介绍](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#introduction)

在Go语言中，数据类型用于对特定类型的数据进行分类，确定可以分配给该类型的值以及可以对其执行的操作。在编程时，有时需要在类型之间转换值，以便以不同的方式操作值。例如，您可能需要将数值与字符串连接起来，或者在初始化为整数值的数字中表示小数位。用户生成的数据通常被自动分配为字符串数据类型，即使它由数字组成;为了在此输入中执行数学运算，您必须将字符串转换为数字数据类型。

由于Go是一种静态类型语言，[数据类型被绑定到变量](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#declaring-data-types-for-variables)而不是值。这意味着，如果你将一个变量定义为`int`，它只能是`int`;你不能在不转换变量的数据类型的情况下将`string`赋值给它。Go语言中数据类型的静态特性使得学习转换它们的方法变得更加重要。

本教程将指导您转换数字和字符串，并提供示例以帮助您熟悉不同的用例。

## [转换号码类型](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-number-types)

Go有几种数值类型可供选择。它们主要分为两种类型：[整数](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#integers)和[浮点数](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#floating-point-numbers)。

在许多情况下，您可能希望在数值类型之间进行转换。在[不同大小的数值类型](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#sizes-of-numeric-types)之间进行转换有助于优化特定类型的系统架构的性能。如果你有一个来自代码另一部分的整数，并想对它进行除法运算，你可能想将整数转换为浮点数，以保持运算的精度。此外，处理持续时间通常涉及整数转换。为了解决这些情况，Go语言为大多数数值类型提供了内置的类型转换。

### [在数据类型之间转换](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-between-integer-types)

Go语言有很多整型数据类型可供选择。什么时候使用一种类型而不是另一种类型通常更多的是关于[性能](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#picking-numeric-data-types);然而，有时候你需要从一种整数类型转换到另一种类型。例如，Go有时会自动生成数值`int`，这可能与您的输入值不匹配。如果您的输入值是`int64`，您将无法在相同的数学表达式中使用`int`和`int64`数字，直到您将其数据类型转换为匹配。

假设你有一个`int8`，你需要将它转换为`int32`。你可以通过将它包装在`int32()`类型转换中来实现：

```go
var index int8 = 15

var bigIndex int32

bigIndex = int32(index)

fmt.Println(bigIndex)
```

```
Output  15
```

此代码块将`index`定义为`int8`数据类型，将`bigIndex`定义为`int32`数据类型。为了将`index`的值存储在`bigIndex`中，它将数据类型转换为`int32`。这是通过将`int32()`转换包装在`index`变量周围来完成的。

要验证您的数据类型，您可以使用以下语法的`fmt.Printf`语句和`%T`动词：

```go
fmt.Printf("index data type:    %T\n", index)
fmt.Printf("bigIndex data type: %T\n", bigIndex)
```

```
Output  index data type:    int8
bigIndex data type: int32
```

由于这使用了`%T`动词，print语句输出变量的类型，而不是变量的实际值。这样，您就可以确认转换后的数据类型。

您也可以将较大的位大小整数转换为较小的位大小整数：

```go
var big int64 = 64

var little int8

little = int8(big)

fmt.Println(little)
```

```
Output  64
```

请记住，在转换整数时，您可能会超过数据类型的最大值和[回绕](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#overflow-vs-wraparound)：

```go
var big int64 = 129
var little = int8(big)
fmt.Println(little)
```

```
Output  -127
```

当值被转换为太小而无法容纳它的数据类型时，就会发生回绕。在前面的示例中，8位数据类型`int8`没有足够的空间容纳64位变量`big`。从较大的数字数据类型转换为较小的数字数据类型时应始终小心，以免意外截断数据。

### [将整数转换为浮点数](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-integers-to-floats)

在Go中将整数转换为浮点数类似于将一种整数类型转换为另一种类型。你可以使用内置的类型转换，方法是在你要转换的整数周围包装`float64()`或`float32()`：

```
var x int64 = 57

var y float64 = float64(x)

fmt.Printf("%.2f\n", y)
Output  57.00
```

这段代码声明了一个类型为`x`的变量`int64`，并将其值转换为`57`。

```go
var x int64 = 57
```

将`float64()`转换包装在`x`周围会将`57`的值转换为浮点值`57.00`。

```go
var y float64 = float64(x)
```

`%.2f` print动词告诉`fmt.Printf`用两个小数格式化浮点数。

你也可以在变量上使用这个过程。下面的代码声明`f`等于`57`，然后打印出新的float：

```go
var f float64 = 57
fmt.Printf("%.2f\n", f)
```

```
Output  57.00
```

通过使用`float32()`或`float64()`，您可以将整数转换为浮点数。接下来，您将学习如何将浮点数转换为整数。

### [将浮点数转换为整数](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-floats-to-integers)

Go可以将浮点数转换为整数，但程序将失去浮点数的精度。

在`int()`中包装浮点数，或它的一种与体系结构无关的数据类型，其工作原理与您使用它从一种整数类型转换为另一种整数类型时类似。您可以在括号内添加一个浮点数，将其转换为整数：

```go
var f float64 = 390.8
var i int = int(f)

fmt.Printf("f = %.2f\n", f)
fmt.Printf("i = %d\n", i)
```

```
Output  f = 390.80
i = 390
```

这种语法会将浮点数`390.8`转换为整数`390`，去掉小数位。

你也可以使用变量。下面的代码声明`b`等于`125.0`，`c`等于`390.8`，然后将它们打印为整数。短变量声明（`:=`）缩短了语法：

```
b := 125.0
c := 390.8

fmt.Println(int(b))
fmt.Println(int(c))
Output  125
390
```

当使用`int()`类型将浮点数转换为整数时，Go会将浮点数的小数和剩余数字切掉以创建整数。请注意，即使你可能想将390.8舍入到391，Go也不会通过`int()`类型来实现这一点。相反，它会删除小数。

### [通过除法转换的数字](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#numbers-converted-through-division)

当在Go语言中对整数类型进行除法时，结果也将是一个整数类型，模或余数被删除：

```go
a := 5 / 2
fmt.Println(a)
```

```
Output  2
```

如果在除法时，任何数字类型都是浮点型，那么所有类型都会自动声明为浮点型：

```go
 a := 5.0 / 2
 fmt.Println(a)
```

```
Output  2.5
```

这将浮点数`5.0`除以整数`2`，并且答案`2.5`是保持小数精度的浮点数。

在本节中，您已经在不同的数字数据类型之间进行了转换，包括不同大小的整数和浮点数。接下来，您将学习如何在数字和字符串之间进行转换。

## [使用字符串转换](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-with-strings)

字符串是一个或多个字符（字母、数字或符号）的序列。字符串是计算机程序中常见的数据形式，您可能需要经常将字符串转换为数字或将数字转换为字符串，特别是当您接收用户生成的数据时。

### [将数字转换为字符串](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-numbers-to-strings)

您可以使用Go标准库中`strconv.Itoa`包中的`strconv`方法将数字转换为字符串。如果您将数字或变量传递到方法的括号中，则该数字值将转换为字符串值。

首先，让我们看看转换整数。要将整数`12`转换为字符串值，您可以将`12`传递到`strconv.Itoa`方法中：

```
package main

import (
 "fmt"
 "strconv"
)

func main() {
 a := strconv.Itoa(12)
 fmt.Printf("%q\n", a)
}
```

运行此程序时，您将收到以下输出：

```
Output  "12"
```

数字12周围的引号表示该数字不再是整数，而是字符串值。

您使用了`:=`赋值操作符来声明一个名为`a`的新变量，并将从`strconv.Itoa()`函数返回的值赋值。在本例中，您将值`12`赋给了变量。您还在`%q`函数中使用了`fmt.Printf`动词，它告诉函数将提供的字符串引用起来。

通过变量，您可以开始了解将整数转换为字符串的实用性。假设您想跟踪用户的日常编程进度，并输入他们一次编写了多少行代码。您希望将此反馈显示给用户，并同时打印出字符串和整数值：

```go
package main

import (
 "fmt"
)

func main() {
 user := "Sammy"
 lines := 50

 fmt.Println("Congratulations, " + user + "! You just wrote " + lines + " lines of code.")
}
```

当你运行这段代码时，你会收到以下错误：

```
Output  invalid operation: ("Congratulations, " + user + "! You just wrote ") + lines (mismatched types string and int)
```

在Go语言中，你不能连接字符串和整数，所以你必须将变量`lines`转换为字符串值：

```
package main

import (
 "fmt"
 "strconv"
)

func main() {
 user := "Sammy"
 lines := 50

 fmt.Println("Congratulations, " + user + "! You just wrote " + strconv.Itoa(lines) + " lines of code.")
}
```

现在，当您运行代码时，您将收到以下输出，祝贺您的用户取得了进展：

```
Output  Congratulations, Sammy! You just wrote 50 lines of code.
```

如果你想把浮点数转换成字符串，而不是把整数转换成字符串，你可以遵循类似的步骤和格式。当你从Go标准库中的`fmt.Sprint`包向`fmt`方法传递一个float时，将返回一个float的字符串值。你可以使用float值本身或变量：

```go
package main

import (
 "fmt"
)

func main() {
 fmt.Println(fmt.Sprint(421.034))

 f := 5524.53
 fmt.Println(fmt.Sprint(f))
}
```

```
Output  421.034
5524.53
```

你可以通过连接一个字符串来测试它是否正确：

```go
package main

import (
 "fmt"
)

func main() {
 f := 5524.53
 fmt.Println("Sammy has " + fmt.Sprint(f) + " points.")
}
```

```
Output  Sammy has 5524.53 points.
```

你可以确信你的浮点数被正确地转换成了字符串，因为连接是正确的。

### [将字符串转换为数字](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-strings-to-numbers)

可以使用Go标准库中的`strconv`包将字符串转换为数字。`strconv`包有转换整数和浮点数类型的函数。这是接受用户输入时非常常见的操作。例如，如果你有一个程序，要求一个人的年龄，当他们输入的反应，它被捕获为一个`string`。然后，您需要将其转换为`int`以进行任何数学运算。

如果你的字符串没有小数位，你很可能想使用`strconv.Atoi`函数将其转换为整数。如果你知道你将使用数字作为浮点数，你会使用`strconv.ParseFloat`。

让我们使用用户Sammy跟踪每天编写的代码行的示例。您可能希望使用数学操作这些值，以便为用户提供更有趣的反馈，但这些值当前存储在字符串中：

```go
package main

import (
 "fmt"
)

func main() {
 lines_yesterday := "50"
 lines_today := "108"

 lines_more := lines_today - lines_yesterday

 fmt.Println(lines_more)
}
```

```
Output  invalid operation: lines_today - lines_yesterday (operator - not defined on string)
```

因为这两个数值存储在字符串中，所以您收到了一个错误。减法的操作数`-`不是两个字符串值的有效操作数。

修改代码以包含将字符串转换为整数的`strconv.Atoi()`方法，这将允许您对最初为字符串的值进行数学运算。由于在将字符串转换为整数时可能会失败，因此必须检查是否有任何错误。 您可以使用`if`语句来检查您的转换是否成功。

```
package main

import (
 "fmt"
 "log"
 "strconv"
)

func main() {
 lines_yesterday := "50"
 lines_today := "108"

 yesterday, err := strconv.Atoi(lines_yesterday)
 if err != nil {
  log.Fatal(err)
 }

 today, err := strconv.Atoi(lines_today)
 if err != nil {
  log.Fatal(err)
 }
 lines_more := today - yesterday

 fmt.Println(lines_more)
}
```

因为字符串可能不是数字，所以`strconv.Atoi()`方法将返回转换后的类型以及潜在的错误。当使用`lines_yesterday`函数从`strconv.Atoi`转换时，您必须检查`err`返回值以确保该值已转换。如果`err`不是`nil`，则意味着`strconv.Atoi`无法成功地将字符串值转换为整数。在本例中，您使用了`if`语句来检查错误，如果返回错误，则使用`log.Fatal`记录错误并退出程序。

当你运行上面的代码时，你会得到：

```
Output  58
```

现在尝试转换一个不是数字的字符串：

```go
package main

import (
 "fmt"
 "strconv"
)

func main() {
 a := "not a number"
 b, err := strconv.Atoi(a)
 fmt.Println(b)
 fmt.Println(err)
}
```

您将得到以下错误：

```
Output  0
strconv.Atoi: parsing "not a number": invalid syntax
```

由于声明了`b`，但`strconv.Atoi`未能进行转换，因此从未将值分配给`b`。注意`b`的值为`0`。这是因为Go有默认值，在Go中称为零值。`strconv.Atoi`提供了一个错误，描述了为什么它无法转换字符串。

### [转换字符串和字符串](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#converting-strings-and-bytes)

Go语言中的字符串被存储为字节片。在Go语言中，你可以通过将一个字节片包装在相应的转换`[]byte()`和`string()`中来在字节片和字符串之间进行转换：

```go
package main

import (
 "fmt"
)

func main() {
 a := "my string"

 b := []byte(a)

 c := string(b)

 fmt.Println(a)

 fmt.Println(b)

 fmt.Println(c)
}
```

在这里，您已经在`a`中存储了一个字符串值，然后将其转换为一个字节片`b`，然后将字节片转换回字符串`c`。然后将`a`、`b`和`c`打印到屏幕上：

```
Output  my string
[109 121 32 115 116 114 105 110 103]
my string
```

输出的第一行是原始字符串`my string`。输出的第二行是组成原始字符串的字节片。第三行显示字节片可以安全地转换回字符串并打印出来。

## [结论](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go#conclusion)

本Go教程演示了如何将几种重要的原生数据类型转换为其他数据类型，主要是通过内置方法。能够在Go中转换数据类型将允许你做一些事情，比如接受用户输入，并在不同的数字类型之间进行数学运算。稍后，当你使用Go语言编写程序，接受来自许多不同来源的数据，如数据库和API，你将使用这些转换方法，以确保你可以对你的数据进行操作。您还可以通过将数据转换为更小的数据类型来优化存储。
