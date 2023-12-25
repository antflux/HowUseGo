# 如何在 Go 中使用运算符进行数学运算

### [介绍](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#introduction)

数字在编程中很常见。它们用于表示诸如屏幕尺寸、地理位置、金钱和点数、视频中经过的时间、游戏化身的位置、通过分配数字代码的颜色等。

在编程中有效地执行数学运算是一项重要的技能，因为您将经常使用数字。虽然对数学的高水平理解当然可以帮助你成为一个更好的程序员，但这不是先决条件。如果你没有数学背景，试着把数学看作是一种工具，可以帮助你完成你想实现的目标，也可以提高你的逻辑思维能力。

我们

- [整数](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#integers)是整数，可以是正数、负数或0（...、`-1`、`0`、`1`、...）。
- [浮点数](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go#floating-point-numbers)是包含小数点的真实的数字，如`9.0`或`-2.25`.

本教程将回顾我们可以在Go中使用数字数据类型的运算符。

## [运营商](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#operators)

运算符是一个符号或函数，表示一个操作。例如，在数学中，加号或`+`是表示加法的运算符。

在Go语言中，我们会看到一些从数学中引入的熟悉的运算符。然而，我们将使用的其他运算符是特定于计算机编程的。

这里是Go中与数学相关的运算符的快速参考表。我们将在本教程中涵盖以下所有操作。

| 操作    | 它返回什么        |
| ------- | ----------------- |
| `x + y` | `x`和`y`之和      |
| `x + y` | `x`和`y`的区别    |
| `x + y` | `x`的更改符号     |
| `x + y` | 身份`x`           |
| `x + y` | `x`和`y`的产品    |
| `x + y` | `x`和`y`的商      |
| `x + y` | `x` / `y`剩余部分 |

我们还将介绍复合赋值运算符，包括`+=`和`*=`，它们将算术运算符与`=`运算符组合在一起。

## [加减法](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#addition-and-subtraction)

在Go中，加法和减法运算符的执行就像它们在数学中一样。事实上，你可以使用Go编程语言作为计算器。

让我们看一些例子，从整数开始：

```go
fmt.Println(1 + 5)
```

```
Output  6
```

我们可以使用如下语法初始化变量来代表整数值，而不是直接将整数传递到`fmt.Println`语句中：

```go
a := 88
b := 103

fmt.Println(a + b)
```

```
Output  191
```

因为整数可以是正数和负数（也可以是0），我们可以用正数加上负数：

```go
c := -36
d := 25

fmt.Println(c + d)
```

```
Output  -11
```

加法的行为与浮点数类似：

```go
e := 5.5
f := 2.5

fmt.Println(e + f)
```

```
Output  8
```

因为我们把两个浮点数加在一起，Go返回一个带小数位的浮点数。然而，由于在这种情况下小数位为零，`fmt.Println`删除了小数格式。为了正确地格式化输出，我们可以使用`fmt.Printf`和动词`%.2f`，它将格式设置为两位小数，就像这个例子：

```go
fmt.Printf("%.2f", e + f)
```

```
Output  8.00
```

减法的语法与加法的语法相同，只是我们将运算符从加号（`+`）改为减号（`-`）：

```go
g := 75.67
h := 32.0

fmt.Println(g - h)
```

```
Output  43.67
```

在Go语言中，我们只能在相同的数据类型上使用运算符。我们

```go
i := 7
j := 7.0
fmt.Println(i + j)
```

```
Output  i + j (mismatched types int and float64)
```

尝试对不同的数据类型使用运算符将导致编译器错误。

## [一元算术运算](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#unary-arithmetic-operations)

一元数学表达式仅由一个分量或元素组成。在Go语言中，我们可以使用加号和减号作为一个单独的元素与一个值配对：返回值的标识（`+`），或者改变值的符号（`-`）。

虽然不常用，但加号表示值的标识。我们可以用加号表示正值：

```go
i := 3.3
fmt.Println(+i)
```

```
Output  3.3
```

当我们将加号与负值一起使用时，它也将返回该值的标识，在这种情况下，它将是负值：

```go
j := -19
fmt.Println(+j)
```

```
Output  -19
```

对于负值，加号返回相同的负值。

然而，负号改变了值的符号。因此，当我们传递一个正值时，我们会发现该值之前的负号将返回一个负值：

```go
k := 3.3
fmt.Println(-k)
```

```
Output  -3.3
```

或者，当我们使用带负值的负号一元运算符时，将返回正值：

```go
j := -19
fmt.Println(-j)
```

```
Output  19
```

由加号和减号表示的一元算术运算将返回值的标识（在`+i`的情况下）或值的相反符号（如`-i`中）。

## [乘法和除法](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#multiplication-and-division)

像加法和减法一样，乘法和除法看起来与数学中的方法非常相似。在Go语言中，乘法的符号是`*`，除法的符号是`/`。

下面是一个在Go语言中用两个浮点值做乘法的例子：

```go
k := 100.2
l := 10.2

fmt.Println(k * l)
```

```
Output  1022.04
```

在Go语言中，除法有不同的特性，这取决于我们要除法的数值类型。

如果我们要对整数进行除法，Go语言的`/`运算符会执行地板除法，其中对于商x，返回的数字是小于或等于x的最大整数。

如果你运行下面的例子来划分`80 / 6`，你会收到`13`作为输出，数据类型将是`int`：

```go
package main

import (
 "fmt"
)

func main() {
 m := 80
 n := 6

 fmt.Println(m / n)
}
```

```
Output  13
```

如果所需的输出是浮点数，则必须在除法之前显式转换值。

您可以通过将所需的float类型`float32()`或`float64()`包装在您的值周围来做到这一点：

```go
package main

import (
 "fmt"
)

func main() {
 s := 80
 t := 6
 r := float64(s) / float64(t)
 fmt.Println(r)
}
```

```
Output  13.333333333333334
```

## [Module模式](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#modulo)

`%`运算符是模运算符，它返回余数而不是除法后的商。这对于查找同一数字的倍数非常有用。

让我们看一个modulo的例子：

```go
o := 85
p := 15

fmt.Println(o % p)
```

```
Output  10
```

为了分解这个，`85`除以`15`返回`5`的商和`10`的余数。我们的程序在这里返回值`10`，因为模运算符返回除法表达式的余数。

要使用`float64`数据类型进行模数计算，您将使用`Mod`包中的`math`函数：

```
package main

import (
 "fmt"
 "math"
)

func main() {
 q := 36.0
 r := 8.0

 s := math.Mod(q, r)

 fmt.Println(s)
}
Output  4
```

## [运算符优先级](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#operator-precedence)

在围棋中，就像在数学中一样，我们需要记住，运算符将按照优先级顺序进行计算，而不是从左到右或从右到左。

如果我们看看下面的数学表达式：

```
u = 10 + 10 * 5
```

我们可以从左到右读它，但乘法将首先完成，所以如果我们要打印`u`，我们将得到以下值：

```
Output  60
```

这是因为`10 * 5`计算为`50`，然后我们添加`10`以返回`60`作为最终结果。

相反，如果我们想将值`10`与`10`相加，然后将该和乘以`5`，我们在Go中使用括号，就像在数学中一样：

```go
u := (10 + 10) * 5
fmt.Println(u)
```

```
Output  100
```

记住操作顺序的一种方法是通过首字母缩略词PEMDAS：

| 秩序 | 信   | 代表 |
| ---- | ---- | ---- |
| 1    | P    | 括号 |
| 2    | E    | 指数 |
| 3    | M    | 乘法 |
| 4    | D    | 司   |
| 5    | 一   | 此外 |
| 6    | S    | 减法 |

您可能熟悉操作顺序的另一个首字母缩写，如BEDMAS或BODMAS。无论哪个缩写词最适合你，在Go中执行数学运算时都要记住它，以便返回你期望的结果。

## [赋值运算符](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#assignment-operators)

最常见的赋值运算符是您已经使用过的：等号`=`。`=`赋值操作符将右边的值赋给左边的变量。例如，`v = 23`将整数`23`的值赋给变量`v`。

在编程时，通常使用复合赋值运算符对变量的值执行运算，然后将结果新值赋给该变量。这些复合运算符联合收割机将算术运算符与`=`运算符组合在一起。因此，为了加法，我们将联合收割机`+`与`=`组合以得到复合运算符`+=`。让我们看看它是什么样子的：

```go
w := 5
w += 1
fmt.Println(w)
```

```
Output  6
```

首先，我们将变量`w`设置为等于`5`的值，然后使用`+=`复合赋值运算符将右边的数字加到左边变量的值上，然后将结果赋给`w`。

复合赋值运算符经常用于`for`循环的情况，当你想重复一个过程多次时，你会使用它：

```go
package main

import "fmt"

func main() {

 values := []int{0, 1, 2, 3, 4, 5, 6}

 for _, x := range values {

  w := x

  w *= 2

  fmt.Println(w)
 }

}
```

```
Output  0
2
4
6
8
10
12
```

通过使用`for`循环遍历名为`values`的切片，您可以自动执行`*=`运算符的过程，该运算符将变量`w`乘以数字`2`，然后将结果赋回变量`w`。

对于本教程中讨论的每一个算术运算符，Go语言都有一个复合赋值运算符。

添加然后赋值：

```go
y += 1
```

要减去然后赋值，请执行以下操作：

```go
y -= 1
```

先乘后赋值：

```go
y *= 2
```

要除，然后赋值：

```go
y /= 3
```

返回余数然后赋值：

```go
y %= 3
```

当需要递增或递减时，或者当需要自动化程序中的某些过程时，复合赋值运算符非常有用。

## [结论](https://www.digitalocean.com/community/tutorials/how-to-do-math-in-go-with-operators#conclusion)

本教程介绍了将用于整数和浮点数字数据类型的许多运算符。您可以在[了解Go中的数据类型](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go)和[如何转换数据类型](https://www.digitalocean.com/community/tutorials/how-to-convert-data-types-in-go)中了解有关不同数据类型的更多信息。
