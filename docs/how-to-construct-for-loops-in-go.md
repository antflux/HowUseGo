# 如何在 Go 中构建 For 循环

### [介绍]

在计算机编程中，*循环*是一种循环重复执行一段代码的代码结构，通常直到满足某些条件状态。在计算机编程中使用循环可以让您自动执行类似的任务并多次重复类似的任务。想象一下，如果您有一个需要处理的文件列表，或者您想计算文章中的行数。您可以在代码中使用循环来解决这些类型的问题。

在 Go 中，`for`基于循环变量或循环变量来实现代码的重复执行。与其他具有多个循环结构（例如、等）的编程语言不同`while`，`do`Go 仅循环`for`。这可以使您的代码更清晰、更多可执行性，因为您不必担心使用多种策略来实现相同的循环构造。与其他语言相比，这种增强的可执行性和减少的开发过程中的分配负担也影响您的代码更不重要容易出错。

在本教程中，您将了解 Go`for`循环的工作原理，包括其三个主要变体的使用。我们将首先展示如何创建不同类型的`for`循环，然后展示如何[在 Go 中循环访问顺序数据类型]。最后我们将解释了如何使用洼地循环。

## [声明 ForClause 和条件循环]

为了考虑到各种例子，可以使用不同的方法`for`在 Go 中创建循环，终结方法都有自己的功能。`for`这些是用**Condition**、**ForClause**或**RangeClause**创建循环。在本节中，我们将解释如何声明并使用 ForClause 和 Condition 变体。

`for`首先让我们看看如何将循环与 ForClause 一起使用。

*ForClause循环*被定义为具有一个*初始语句*，后跟一个*条件*，然后是一个*后语句*。它们按以下语法排列：

```
for [ Initial Statement ] ; [ Condition ] ; [ Post Statement ] {
    [Action]
}
```

为了解释前面的组件的作用，让我们看一下`for`使用 ForClause 递增语法指定范围的值的循环：

```go
for i := 0; i < 5; i++ {
 fmt.Println(i)
}
```

让我们分解这个循环并识别每个部分。

循环的第一部分是`i := 0`。这是最初的声明：

```
for i := 0; i < 5; i++ {
 fmt.Println(i)
}
```

它表明我们正在声明一个名为的变量`i`，大会最终值设置为`0`。

接下来是条件：

```
for i := 0; i < 5; i++ {
 fmt.Println(i)
}
```

在这种情况下，我们声明当`i`小于的值时`5`，应该循环继续循环。

最后，我们有 post 语句：

```
for i := 0; i < 5; i++ {
 fmt.Println(i)
}
```

`i`在 post 语句中，每次迭代发生时，我们使用`i++` [增量](https://golang.org/ref/spec#IncDec_statements)运算符将循环变量加一。

当我们运行这个程序时，输出如下所示：

```
Output  0
1
2
3
4
```

循环运行了 5 次。最初，它设置`i`为`0`，然后检查是否`i`小于`5`。由于 的值`i`小于`5`，因此执行循环并`fmt.Println(i)`执行 的操作。循环结束后，`i++`调用 的 post 语句，并且 的值`i`加 1。

**注意：**请记住，在编程中我们倾向于从索引 0 开始，因此这就是为什么虽然打印出 5 个数字，但它们的范围是 0-4。

我们不限于从 0 开始或以指定值结束。我们可以为初始语句分配任何值，也可以在后置语句中停止于任何值。这允许我们创建任何所需的范围来循环：

```go
for i := 20; i < 25; i++ {
 fmt.Println(i)
}
```

在这里，迭代从 20（含）到 25（不含），因此输出如下所示：

```
Output  20
21
22
23
24
```

我们还可以使用 post 语句以不同的值递增。`step`这与其他语言类似：

首先，让我们使用具有正值的 post 语句：

```
for i := 0; i < 15; i += 3 {
 fmt.Println(i)
}
```

在本例中，`for`循环被设置为打印从 0 到 15 的数字，但增量为 3，因此只打印每三个数字，如下所示：

```
Output  0
3
6
9
12
```

我们还可以对 post 语句参数使用负值来向后迭代，但我们必须相应地调整我们的初始语句和条件参数：

```go
for i := 100; i > 0; i -= 10 {
 fmt.Println(i)
}
```

这里，我们设置`i`为 的初始值`100`，使用 的条件`i < 0`停止于`0`，post 语句使用运算符将该值减 10 `-=`。循环开始于`100`并结束于`0`，每次迭代减少 10。我们可以在输出中看到这种情况发生：

```
Output  100
90
80
70
60
50
40
30
20
10
```

您还可以从语法中排除初始语句和后置语句`for`，而仅使用条件。这就是所谓的*条件循环*：

```
i := 0
for i < 5 {
 fmt.Println(i)
 i++
}
```

这次，我们在前面的代码行中`i`独立于循环声明了该变量。`for`该循环只有一个条件子句，用于检查是否`i`小于`5`。只要条件计算结果为`true`，循环就会继续迭代。

有时您可能不知道完成某项任务所需的迭代次数。在这种情况下，您可以省略所有语句，并使用`break`关键字退出执行：

```
for {
 if someCondition {
  break
 }
 // do action here
}
```

一个例子可能是，如果我们正在从缓冲区等不确定大小的结构中读取数据[，](https://golang.org/pkg/bytes/#Buffer)并且我们不知道何时会完成读取：

缓冲区.go

```go
package main

import (
 "bytes"
 "fmt"
 "io"
)

func main() {
 buf := bytes.NewBufferString("one\ntwo\nthree\nfour\n")

 for {
  line, err := buf.ReadString('\n')
  if err != nil {
   if err == io.EOF {

    fmt.Print(line)
    break
   }
   fmt.Println(err)
   break
  }
  fmt.Print(line)
 }
}
```

在前面的代码中，`buf :=bytes.NewBufferString("one\ntwo\nthree\nfour\n")`声明了一个包含一些数据的缓冲区。因为我们不知道缓冲区何时完成读取，所以我们创建了一个`for`不带子句的循环。在循环内部`for`，我们`line, err := buf.ReadString('\n')`从缓冲区读取一行并检查从缓冲区读取是否有错误。如果有，我们解决错误，并[使用`break`关键字退出 for 循环]。有了这些`break`要点，您就不需要包含停止循环的条件。

在本节中，我们学习了如何声明 ForClause 循环并使用它来迭代已知的值范围。我们还学习了如何使用条件循环进行迭代，直到满足特定条件。接下来，我们将了解如何使用 RangeClause 来迭代顺序数据类型。

## [使用 RangeClause 循环顺序数据类型]

在 Go 中，通常使用循环来迭代顺序或集合数据类型（如[切片、数组]和[字符串）]`for`的元素。为了更容易做到这一点，我们可以使用带有*RangeClause*语法的循环。虽然您可以使用 ForClause 语法循环遍历顺序数据类型，但 RangeClause 更清晰且更易于阅读。`for`

在使用 RangeClause 之前，我们先看看如何使用 ForClause 语法来迭代切片：

主程序

```
package main

import "fmt"

func main() {
 sharks := []string{"hammerhead", "great white", "dogfish", "frilled", "bullhead", "requiem"}

 for i := 0; i < len(sharks); i++ {
  fmt.Println(sharks[i])
 }
}
```

运行此命令将给出以下输出，打印出切片的每个元素：

```
Output  hammerhead
great white
dogfish
frilled
bullhead
requiem
```

现在，让我们使用 RangeClause 来执行相同的一组操作：

主程序

```
package main

import "fmt"

func main() {
 sharks := []string{"hammerhead", "great white", "dogfish", "frilled", "bullhead", "requiem"}

 for i, shark := range sharks {
  fmt.Println(i, shark)
 }
}
```

在本例中，我们打印列表中的每个项目。尽管我们使用了变量`i`和`shark`，但我们可以将该变量称为任何其他[有效的变量名称]，并且我们会得到相同的输出：

```
Output  0 hammerhead
1 great white
2 dogfish
3 frilled
4 bullhead
5 requiem
```

当`range`在切片上使用时，它将始终返回两个值。第一个值将是循环当前迭代所在的索引，第二个值是该索引处的值。在本例中，对于第一次迭代，索引为`0`，值为`hammerhead`。

有时，我们只想要切片元素内的值，而不是索引。但是，如果我们将前面的代码更改为仅打印出值，我们将收到编译时错误：

主程序

```
package main

import "fmt"

func main() {
 sharks := []string{"hammerhead", "great white", "dogfish", "frilled", "bullhead", "requiem"}

 for i, shark := range sharks {
  fmt.Println(shark)
 }
}
Output  src/range-error.go:8:6: i declared and not used
```

因为在循环`i`中声明了`for`，但从未使用过，所以编译器将响应 的错误`i declared and not used`。这与您在 Go 中声明变量但不使用它时收到的错误相同。

因此，Go 有一个[空白标识符](https://golang.org/ref/spec#Blank_identifier)，即下划线 ( `_`)。在`for`循环中，您可以使用空白标识符来忽略从`range`关键字返回的任何值。在本例中，我们要忽略索引，它是返回的第一个参数。

主程序

```
package main

import "fmt"

func main() {
 sharks := []string{"hammerhead", "great white", "dogfish", "frilled", "bullhead", "requiem"}

 for _, shark := range sharks {
  fmt.Println(shark)
 }
}
Output  hammerhead
great white
dogfish
frilled
bullhead
requiem
```

此输出显示`for`循环迭代了字符串切片，并打印了切片中不带索引的每个项目。

您还可以使用`range`将项目添加到列表中：

主程序

```
package main

import "fmt"

func main() {
 sharks := []string{"hammerhead", "great white", "dogfish", "frilled", "bullhead", "requiem"}

 for range sharks {
  sharks = append(sharks, "shark")
 }

 fmt.Printf("%q\n", sharks)
}
Output  ['hammerhead', 'great white', 'dogfish', 'frilled', 'bullhead', 'requiem', 'shark', 'shark', 'shark', 'shark', 'shark', 'shark']
```

`"shark"`在这里，我们为切片长度的每个项目添加了一个占位符字符串`sharks`。

请注意，我们不必使用空白标识符`_`来忽略运算符的任何返回值`range`。`range`如果我们不需要使用任何一个返回值，Go 允许我们省略语句的整个声明部分。

我们还可以使用`range`运算符来填充切片的值：

主程序

```
package main

import "fmt"

func main() {
 integers := make([]int, 10)
 fmt.Println(integers)

 for i := range integers {
  integers[i] = i
 }

 fmt.Println(integers)
}
```

在此示例中，切片`integers`使用十个空值进行初始化，但`for`循环设置列表中的所有值，如下所示：

```
Output  [0 0 0 0 0 0 0 0 0 0]
[0 1 2 3 4 5 6 7 8 9]
```

第一次打印 slice 的值时`integers`，我们看到全为零。然后我们迭代每个索引并将值设置为当前索引。然后，当我们第二次打印值时，显示它们现在都具有through`integers`值。`0``9`

我们还可以使用`range`运算符来迭代字符串中的每个字符：

主程序

```go
package main

import "fmt"

func main() {
 sammy := "Sammy"

 for _, letter := range sammy {
  fmt.Printf("%c\n", letter)
 }
}
```

```
Output  S
a
m
m
y
```

[迭代]映射时，`range`将返回**键**和**值**：

主程序

```go
package main

import "fmt"

func main() {
 sammyShark := map[string]string{"name": "Sammy", "animal": "shark", "color": "blue", "location": "ocean"}

 for key, value := range sammyShark {
  fmt.Println(key + ": " + value)
 }
}
```

```
Output  color: blue
location: ocean
name: Sammy
animal: shark
```

**注意：**需要注意的是，地图返回的顺序是随机的。每次运行该程序时，您可能会得到不同的结果。

现在我们已经学习了如何使用`range` `for`循环迭代顺序数据，让我们看看如何在循环内使用循环。

## [嵌套 For 循环]

循环可以在 Go 中嵌套，就像其他编程语言一样。*嵌套*是指我们将一个结构置于另一个结构之中。在这种情况下，嵌套循环是发生在另一个循环内的循环。当您希望对数据集的每个元素执行循环操作时，这些可能很有用。

[嵌套循环在结构上与嵌套`if`语句]类似。它们的构造如下：

```
for {
    [Action]
    for {
        [Action]  
    }
}
```

程序首先遇到外循环，执行其第一次迭代。第一次迭代会触发内部嵌套循环，然后运行直至完成。然后程序返回到外循环的顶部，完成第二次迭代并再次触发嵌套循环。同样，嵌套循环运行至完成，并且程序返回到外循环的顶部，直到序列完成或中断或其他语句中断该过程。

让我们实现一个嵌套`for`循环，以便我们可以仔细观察。在此示例中，外部循环将迭代称为 的整数切片`numList`，而内部循环将迭代称为 的字符串切片`alphaList`。

主程序

```go
package main

import "fmt"

func main() {
 numList := []int{1, 2, 3}
 alphaList := []string{"a", "b", "c"}

 for _, i := range numList {
  fmt.Println(i)
  for _, letter := range alphaList {
   fmt.Println(letter)
  }
 }
}
```

当我们运行这个程序时，我们将收到以下输出：

```
Output  1
a
b
c
2
a
b
c
3
a
b
c
```

输出表明程序通过打印 完成外循环的第一次迭代，`1`然后触发内循环的完成，连续打印`a`, 。内部循环完成后，程序返回到外部循环的顶部，打印，然后再次打印整个内部循环（、、）等。`b``c``2``a``b``c`

嵌套`for`循环对于迭代由切片组成的切片内的项目非常有用。在由切片组成的切片中，如果我们只使用一个`for`循环，程序会将每个内部列表作为一项输出：

主程序

```go
package main

import "fmt"

func main() {
 ints := [][]int{
  []int{0, 1, 2},
  []int{-1, -2, -3},
  []int{9, 8, 7},
 }

 for _, i := range ints {
  fmt.Println(i)
 }
}
```

```
Output  [0 1 2]
[-1 -2 -3]
[9 8 7]
```

为了访问内部切片的每个单独项目，我们将实现一个嵌套`for`循环：

主程序

```go
package main

import "fmt"

func main() {
 ints := [][]int{
  []int{0, 1, 2},
  []int{-1, -2, -3},
  []int{9, 8, 7},
 }

 for _, i := range ints {
  for _, j := range i {
   fmt.Println(j)
  }
 }
}
```

```
Output  0
1
2
-1
-2
-3
9
8
7
```

当我们在这里使用嵌套`for`循环时，我们能够迭代切片中包含的各个项目。

## [结论]

在本教程中，我们学习了如何声明和使用`for`循环来解决 Go 中的重复任务。我们还学习了循环的三种不同变体`for`以及何时使用它们。要了解有关`for`循环以及如何控制循环流程的更多信息，请阅读[在 Go 中使用循环时使用 Break 和 continue 语句]。
