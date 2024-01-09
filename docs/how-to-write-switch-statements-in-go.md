# 如何在 Go 中编写 Switch 语句

### [介绍]

[条件语句]给予程序员指示程序在条件为真时执行某个操作，在条件为假时执行另一个操作的能力。通常，我们希望将某个[变量]与多个可能的值进行比较，并在每种情况下采取不同的操作。仅使用`if`语句就可以实现这一然而，编写软件不仅仅是让事情工作，而且还要将您的意图传达给未来的自己和其他开发人员。`switch`是一个可选的条件语句，用于传达Go程序在出现不同选项时所采取的操作。

我们可以用switch语句写的所有东西也可以用`if`语句写。在本教程中，我们将看几个例子，看看switch语句可以做什么，它取代的`if`语句，以及它最适合应用的地方。

## [Switch语句的结构]

开关通常用于描述当一个变量被赋予特定值时程序所采取的操作。下面的例子演示了我们如何使用`if`语句来实现这一点：

```go
package main

import "fmt"

func main() {
 flavors := []string{"chocolate", "vanilla", "strawberry", "banana"}

 for _, flav := range flavors {
  if flav == "strawberry" {
   fmt.Println(flav, "is my favorite!")
   continue
  }

  if flav == "vanilla" {
   fmt.Println(flav, "is great!")
   continue
  }

  if flav == "chocolate" {
   fmt.Println(flav, "is great!")
   continue
  }

  fmt.Println("I've never tried", flav, "before")
 }
}
```

这将生成以下输出：

```
Output
chocolate is great!
vanilla is great!
strawberry is my favorite!
I've never tried banana before
```

在`main`中，我们定义了一[片]冰淇淋口味。然后我们使用[`for loop`]来遍历它们。我们使用三个`if`语句打印出不同的消息，表明对不同冰淇淋口味的偏好。每个`if`语句都必须使用`continue`语句来停止`for`循环的执行，这样就不会为首选的冰淇淋口味打印结尾的默认消息。

当我们添加新的冰淇淋首选项时，我们必须不断添加`if`语句来处理新的情况。重复的消息，如在`"vanilla"`和`"chocolate"`的情况下，必须具有重复的`if`语句。对于我们代码的未来读者（包括我们自己）来说，`if`语句的重复性掩盖了它们所做的重要部分-将变量与多个值进行比较并采取不同的操作。此外，我们的回退消息与条件句分开设置，使其看起来不相关。`switch`语句可以帮助我们更好地组织这个逻辑。

`switch`语句以`switch`关键字开始，后面是一些变量，以其最基本的形式执行比较。后面是一对花括号（`{}`），其中可以出现多个case子句。Case子句描述了当提供给switch语句的变量等于case子句引用的值时，Go程序应该采取的操作。下面的示例将上一个示例转换为使用`switch`语句而不是多个`if`语句：

```go
package main

import "fmt"

func main() {
 flavors := []string{"chocolate", "vanilla", "strawberry", "banana"}

 for _, flav := range flavors {
  switch flav {
  case "strawberry":
   fmt.Println(flav, "is my favorite!")
  case "vanilla", "chocolate":
   fmt.Println(flav, "is great!")
  default:
   fmt.Println("I've never tried", flav, "before")
  }
 }
}
```

输出与之前相同：

```
Output
chocolate is great!
vanilla is great!
strawberry is my favorite!
I've never tried banana before
```

我们再次在`main`中定义了一片冰淇淋口味，并使用`range`语句来覆盖每种口味。然而，这一次，我们使用了一个`switch`语句来检查`flav`变量。我们使用两个`case`子句来表示偏好。我们不再需要`continue`语句，因为`case`语句只执行一个`switch`子句。我们还可以通过在`"chocolate"`子句的声明中用逗号分隔`"vanilla"`和`case`条件语句来联合收割机重复的逻辑。`default`条款是我们的全面条款。它将运行任何我们在`switch`语句的主体中没有考虑到的风格。在这种情况下，`"banana"`将导致`default`执行，打印消息`I've never tried banana before`。

这种简化形式的`switch`语句解决了它们最常见的用途：将一个变量与多个备选项进行比较。它还为我们提供了方便，当我们想要对多个不同的值执行相同的操作时，以及当使用提供的`default`关键字不满足列出的任何条件时执行其他一些操作。

当这种简化形式的`switch`被证明过于局限时，我们可以使用更一般的形式的`switch`语句。

## [通用开关语句]

`switch`语句用于对更复杂的条件集合进行分组，以表明它们之间存在某种联系。这通常用于比较某个变量与一系列值，而不是像前面的示例中那样比较特定值。下面的示例使用可以从`if`语句中受益的`switch`语句实现了一个猜测游戏：

```go
package main

import (
 "fmt"
 "math/rand"
 "time"
)

func main() {
 rand.Seed(time.Now().UnixNano())
 target := rand.Intn(100)

 for {
  var guess int
  fmt.Print("Enter a guess: ")
  _, err := fmt.Scanf("%d", &guess)
  if err != nil {
   fmt.Println("Invalid guess: err:", err)
   continue
  }

  if guess > target {
   fmt.Println("Too high!")
   continue
  }

  if guess < target {
   fmt.Println("Too low!")
   continue
  }

  fmt.Println("You win!")
  break
 }
}
```

输出将根据所选的随机数和您玩游戏的程度而有所不同。下面是一个示例会话的输出：

```
Output
Enter a guess: 10
Too low!
Enter a guess: 15
Too low!
Enter a guess: 18
Too high!
Enter a guess: 17
You win!
```

我们的猜谜游戏需要一个随机数来比较猜测结果，所以我们使用了`rand.Intn`包中的`math/rand`函数。为了确保我们每次玩游戏时得到不同的`target`值，我们使用`rand.Seed`来根据当前时间随机化随机数生成器。参数`100`到`rand.Intn`将给予一个0-100范围内的数字。然后我们使用一个`for`循环来开始收集玩家的猜测。

`fmt.Scanf`函数为我们提供了一种将用户输入读取到我们选择的变量中的方法。它接受一个格式字符串动词，将用户的输入转换为我们期望的类型。`%d`在这里意味着我们期待一个`int`，我们传递`guess`变量的地址，以便`fmt.Scanf`能够设置该变量。在[处理了所有解析错误]之后，我们使用两个`if`语句将用户的猜测值与`target`值进行比较。他们返回的`string`，沿着与`bool`一起，控制显示给玩家的消息以及游戏是否会退出。

这些`if`语句掩盖了一个事实，即变量所比较的值的范围都以某种方式相关。也很难一眼看出我们是否错过了范围的某一部分。下一个示例重构了上一个示例，以使用`switch`语句：

```go
package main

import (
 "fmt"
 "math/rand"
)

func main() {
 target := rand.Intn(100)

 for {
  var guess int
  fmt.Print("Enter a guess: ")
  _, err := fmt.Scanf("%d", &guess)
  if err != nil {
   fmt.Println("Invalid guess: err:", err)
   continue
  }

  switch {
  case guess > target:
   fmt.Println("Too high!")
  case guess < target:
   fmt.Println("Too low!")
  default:
   fmt.Println("You win!")
   return
  }
 }
}
```

这将生成类似于以下内容的输出：

```
Output
Enter a guess: 25
Too low!
Enter a guess: 28
Too high!
Enter a guess: 27
You win!
```

在这个版本的猜谜游戏中，我们用一个`if`语句替换了`switch`语句块。我们省略了`switch`的表达式参数，因为我们只对使用`switch`收集条件感兴趣。每个`case`子句都包含一个不同的表达式，用于比较`guess`和`target`。与第一次用`if`替换`switch`语句类似，我们不再需要`continue`语句，因为只有一个`case`子句将被执行。最后，`default`子句处理`guess == target`的情况，因为我们已经用其他两个`case`子句覆盖了所有其他可能的值。

在到目前为止我们看到的例子中，只有一个case语句会被执行。有时，您可能希望将多个`case`子句的行为联合收割机组合起来。`switch`语句提供了实现此行为的另一个关键字。

## [Fallthrough]

有时候，你会想重用另一个`case`子句包含的代码。在这些情况下，可以使用`case`关键字要求Go运行下一个`fallthrough`子句的主体。下一个例子修改了我们前面的冰淇淋口味的例子，以更准确地反映我们对草莓冰淇淋的热情：

```go
package main

import "fmt"

func main() {
 flavors := []string{"chocolate", "vanilla", "strawberry", "banana"}

 for _, flav := range flavors {
  switch flav {
  case "strawberry":
   fmt.Println(flav, "is my favorite!")
   fallthrough
  case "vanilla", "chocolate":
   fmt.Println(flav, "is great!")
  default:
   fmt.Println("I've never tried", flav, "before")
  }
 }
}
```

我们将看到这样的输出：

```
Output
chocolate is great!
vanilla is great!
strawberry is my favorite!
strawberry is great!
I've never tried banana before
```

正如我们之前看到的，我们定义了一个`string`的切片来表示口味，并使用`for`循环通过这个切片来表示口味。这里的`switch`语句与我们之前看到的语句相同，但在`fallthrough`的`case`子句末尾添加了`"strawberry"`关键字。这将导致Go运行`case "strawberry":`的主体，首先打印出字符串`strawberry is my favorite!`。当它遇到`fallthrough`时，它将运行下一个`case`子句的主体。这将导致`case "vanilla", "chocolate":`的主体运行，打印`strawberry is great!`。

`fallthrough`关键字不经常被Go开发人员使用。通常，通过使用`fallthrough`实现的代码重用可以通过使用公共代码定义函数来更好地获得。由于这些原因，通常不鼓励使用`fallthrough`。

## [结论]

`switch`语句帮助我们向其他阅读我们代码的开发人员传达一组比较是以某种方式相互关联的。当将来添加新的case时，它们可以更容易地添加不同的行为，并且可以确保我们忘记的任何内容都可以通过`default`子句正确处理。下次你发现自己写了多个涉及同一个变量的`if`语句时，试着用一个`switch`语句重写它--你会发现当需要考虑其他替代值时，返工会更容易。
