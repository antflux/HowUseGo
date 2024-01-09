# 捕获错误

在Go中，错误处理是一项重要的任务，用于识别程序的异常状态并采取适当的步骤记录诊断信息以供后续调试。与其他语言不同的是，Go中的错误是一种具有`error`类型的值，而不是需要使用专门的语法来处理的特殊语法结构。在Go中，我们必须检查函数可能返回的错误，并决定是否发生了错误，然后采取适当的措施来保护数据，并通知用户或操作员发生了错误。

## 创建错误

在处理错误之前，我们首先需要创建一些错误。标准库提供了两个内置函数来创建错误：`errors.New`和`fmt.Errorf`。这两个函数允许您指定一个自定义错误消息，以便稍后向用户呈现。

`errors.New`接受一个参数，即作为字符串的错误消息，您可以自定义以向用户提醒发生了什么问题。例如：

```go
package main

import (
 "errors"
 "fmt"
)

func main() {
 err := errors.New("barnacles")
 fmt.Println("Sammy says:", err)
}
```

输出：

```
Sammy says: barnacles
```

我们使用标准库的`errors.New`函数创建了一个带有字符串"barnacles"的新错误消息。遵循约定，我们在这里使用小写字母作为错误消息，这是Go编程语言风格指南建议的。

另一方面，`fmt.Errorf`函数允许您动态构建错误消息。它的第一个参数是包含错误消息的字符串，其中包含占位符值，如`%s`表示字符串，`%d`表示整数。`fmt.Errorf`将紧随格式化字符串之后的参数插值到这些占位符中：

```go
package main

import (
 "fmt"
 "time"
)

func main() {
 err := fmt.Errorf("error occurred at: %v", time.Now())
 fmt.Println("An error happened:", err)
}
```

输出：

```
An error happened: Error occurred at: 2019-07-11 16:52:42.532621 -0400 EDT m=+0.000137103
```

在这里，我们使用`fmt.Errorf`函数构建了一个错误消息，其中包含当前时间。我们提供给`fmt.Errorf`的格式化字符串包含`%v`格式指令，告诉`fmt.Errorf`使用提供给格式化字符串之后的第一个参数的默认格式。该参数将是由标准库的`time.Now`函数提供的当前时间。与之前的例子一样，我们将错误消息与短前缀组合起来，并使用`fmt.Println`函数将结果打印到标准输出。

## 处理错误

通常情况下，您不太可能看到像上面的例子一样创建的错误被立即用于没有其他目的。实际上，更常见的情况是在出现错误时创建一个错误并从函数中返回它。然后，调用该函数的调用者将使用`if`语句检查错误是否存在或为`nil`（未初始化的值）。

下面的例子包含一个始终返回错误的函数。请注意，当运行程序时，它产生与前面的例子相同的输出，即使这次是由函数返回错误。在不同的位置声明错误不会更改错误的消息。

```go
package main

import (
 "errors"
 "fmt"
)

func boom() error {
 return errors.New("barnacles")
}

func main() {
 err := boom()

 if err != nil {
  fmt.Println("An error occurred:", err)
  return
 }
 fmt.Println("Anchors away!")
}
```

输出：

```
An error occurred: barnacles
```

在这里，我们定义了一个名为`boom`的函数，该函数返回一个由`errors.New`构造的错误。然后，我们调用此函数并使用`err := boom()`行捕获错误。一旦我们分配了这个错误，我们使用`if err != nil`条件检查它是否存在。在这里，条件将始终评估为`true`，因为我们总是从`boom`函数返回一个错误。

这并不总是情况，所以最好的做法是在错误不存在（`nil`）的情况下处理“正常路径”逻辑，并在错误存在时处理“错误路径”逻辑。当错误存在时，我们使用`fmt.Println`打印错误，如前面的例子一样。最后，我们使用`return`语句跳过执行`fmt.Println("Anchors away!")`，因为这只有在没有错误发生时才应该执行。

{==

在上一个例子中显示的`if err != nil`结构是Go语言错误处理中的工作机制。无论何时一个函数可能产生错误，都很重要使用`if`语句来检查是否发生了错误。通过这种方式，符合Go语言的编码风格，正常路径的逻辑自然地位于第一级缩进级别，而所有错误路径的逻辑则位于第二级缩进级别。

==}

`if`语句还有一个可选的赋值子句，可用于帮助简化调用函数并处理其错误。

运行下一个程序，看到与我们之前的例子相同的输出，但这次使用复合`if`语句来减少一些样板代码：

```go
package main

import (
 "errors"
 "fmt"
)

func boom() error {
 return errors.New("barnacles")
}

func main() {
 if err := boom(); err != nil {
  fmt.Println("An error occurred:", err)
  return
 }
 fmt.Println("Anchors away!")
}
```

输出：

```
An error occurred: barnacles
```

与以前一样，我们有一个名为`boom`的函数，该函数总是返回一个错误。我们将从`boom()`返回的错误分配给`err`，作为`if`语句的第一部分。在`if`语句的第二部分中，分号

后面的部分，那个`err`变量就可用了。我们检查错误是否存在，如以前的例子一样，使用`fmt.Println`打印我们的错误和一个前缀字符串。

在这一部分，我们学会了处理只返回错误的函数。这样的函数很常见，但同样重要的是能够处理那些可能返回多个值的函数的错误。

## 返回值中同时返回错误和值

通常，只返回单个错误值的函数通常是那些影响某种状态变化的函数，例如向数据库插入行。写函数，如果成功完成，还可以返回一个值，如果该函数失败，还可以返回一个潜在的错误。Go允许函数返回多个结果，这可以同时返回一个值和一个错误类型。

要创建一个返回多个值的函数，我们在函数签名的括号中列出每个返回值的类型。例如，一个返回字符串和错误的`capitalize`函数将使用`func capitalize(name string) (string, error)`声明。括号中的`(string, error)`告诉Go编译器，这个函数将按照这个顺序返回一个字符串和一个错误。

运行以下程序，看到一个返回字符串和错误的函数的输出：

```go
package main

import (
 "errors"
 "fmt"
 "strings"
)

func capitalize(name string) (string, error) {
 if name == "" {
  return "", errors.New("no name provided")
 }
 return strings.ToTitle(name), nil
}

func main() {
 name, err := capitalize("sammy")
 if err != nil {
  fmt.Println("Could not capitalize:", err)
  return
 }

 fmt.Println("Capitalized name:", name)
}
```

输出：

```
Capitalized name: SAMMY
```

我们定义了`capitalize`作为一个函数，该函数接受一个字符串（要大写的名称）并返回一个字符串和一个错误值。在`main`函数中，我们调用`capitalize`并使用`:=`运算符在函数返回的两个值中分别为`name`和`err`分配变量。之后，我们执行`if err != nil`检查，如之前的例子一样，如果错误存在，则使用`fmt.Println`打印错误。如果没有错误，我们打印出大写的名称。

可以尝试将`name, err := capitalize("sammy")`中的字符串"sammy"更改为空字符串("")，您将收到错误`Could not capitalize: no name provided`。

当调用函数的返回值有多个时，通常情况下，我们只对错误值感兴趣。您可以使用特殊的`_`变量名放弃函数返回的任何不需要的值。在下面的程序中，我们修改了我们涉及`capitalize`函数的第一个例子，通过传入空字符串("")来产生一个错误。尝试运行此程序，看看我们如何能够只检查错误而舍弃`capitalize`返回的第一个值：

```go
package main

import (
 "errors"
 "fmt"
 "strings"
)

func capitalize(name string) (string, error) {
 if name == "" {
  return "", errors.New("no name provided")
 }
 return strings.ToTitle(name), nil
}

func main() {
 _, err := capitalize("")
 if err != nil {
  fmt.Println("Could not capitalize:", err)
  return
 }
 fmt.Println("Success!")
}
```

输出：

```
Could not capitalize: no name provided
```

在`main`函数中，这次我们将大写的名称（首先返回的字符串）分配给下划线变量(_)。同时，我们将由`capitalize`返回的错误分配给`err`变量。然后，我们检查错误是否存在于`if err != nil`条件中。由于我们在`capitalize("")`的参数中硬编码了一个空字符串，因此这个条件将始终评估为`true`。这导致在`if`语句的正文中通过`fmt.Println`调用打印出"Could not capitalize: no name provided"。在这之后的`return`将跳过`fmt.Println("Success!")`的执行。

## 结论

我们看到了使用标准库创建错误以及如何以符合Go语言的方式构建返回错误的函数的多种方法。在本教程中，我们成功地使用标准库的`errors.New`和`fmt.Errorf`函数创建了各种错误。在以后的教程中，我们将看看如何创建自己的自定义错误类型，以向用户传递更丰富的信息。
