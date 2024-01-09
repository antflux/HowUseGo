# 如何定义和调用函数

### [介绍]

函数是一段代码，一旦定义，就可以重用。 函数是用来让你的代码更容易理解的，它将代码分解成小的、可理解的任务，这些任务可以在整个程序中多次使用。

Go附带了一个强大的标准库，其中包含许多预定义的函数。您可能已经熟悉[fmt](https://golang.org/pkg/fmt/)包中的一些功能：

- `fmt.Println()`将打印对象到标准输出（最可能是您的终端）。
- `fmt.Printf()`这将允许您格式化打印输出。

函数名包含圆括号，并且可能包含参数。

在本教程中，我们将介绍如何定义您自己的函数以在编码项目中使用。

## [定义一个函数]

让我们从把经典的“你好，世界！[”编程]成函数。

我们将在我们选择的文本编辑器中创建一个新的文本文件，并调用程序`hello.go`。然后，我们将定义函数。

函数是使用`func`关键字定义的。然后，后面跟着您选择的名称和一组括号，其中包含函数将采用的任何参数（它们可以为空）。函数代码行被括在大括号`{}`中。

在本例中，我们将定义一个名为`hello()`的函数：

hello.go

```go
func hello() {}
```


这将设置用于创建函数的初始语句。

从这里开始，我们将添加第二行来提供函数的操作说明。在本例中，我们将把`Hello, World!`打印到控制台：

hello.go

```go
func hello() {
	fmt.Println("Hello, World!")
}
```


我们的函数现在已经完全定义了，但是如果我们在这个时候运行程序，什么也不会发生，因为我们没有调用函数。

所以，在我们的`main()`函数块中，让我们用`hello()`调用函数：

hello.go

```go
package main

import "fmt"

func main() {
	hello()
}

func hello() {
	fmt.Println("Hello, World!")
}
```


现在，让我们运行程序：

```bash
go run hello.go
```


您将收到以下输出：

```
Output  Hello, World!
```

请注意，我们还引入了一个名为`main()`的函数。`main()`函数是一个特殊的函数，它告诉编译器这是程序应该开始的地方。对于任何你想执行的程序（一个可以从命令行运行的程序），你都需要一个`main()`函数。`main()`函数必须只出现一次，在`main()`包中，并且不接收和返回任何参数。这允许在任何Go程序中[执行程序](https://golang.org/ref/spec#Program_execution)。按照以下示例：

main.go

```go
package main

import "fmt"

func main() {
	fmt.Println("this is the main section of the program")
}
```


函数可能比我们定义的`hello()`函数更复杂。我们可以在函数块中使用[`for`循环]、[条件语句]等。

例如，下面的函数使用条件语句来检查`name`变量的输入是否包含元音，然后使用`for`循环来遍历`name`字符串中的字母。

names.go

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	names()
}

func names() {
	fmt.Println("Enter your name:")

	var name string
	fmt.Scanln(&name)
	// Check whether name has a vowel
	for _, v := range strings.ToLower(name) {
		if v == 'a' || v == 'e' || v == 'i' || v == 'o' || v == 'u' {
			fmt.Println("Your name contains a vowel.")
			return
		}
	}
	fmt.Println("Your name does not contain a vowel.")
}
```


我们在这里定义的`names()`函数设置了一个带有输入的`name`变量，然后在`for`循环中设置了一个条件语句。这显示了如何在函数定义中组织代码。然而，根据我们对程序的意图以及我们希望如何设置代码，我们可能希望将条件语句和`for`循环定义为两个单独的函数。

在程序中定义函数使我们的代码模块化和可重用，这样我们就可以调用相同的函数而无需重写它们。

## [使用参数]

到目前为止，我们已经研究了不带参数的空括号函数，但是我们可以在函数定义中在括号内定义参数。

参数是函数定义中的命名实体，指定函数可以接受的参数。在Go语言中，必须为每个参数指定[数据类型]。

让我们创建一个程序，重复一个单词指定的次数。它将使用一个名为`string`的`word`参数和一个名为`int`的`reps`参数来计算单词重复的次数。

repeat.go

```go
package main

import "fmt"

func main() {
	repeat("Sammy", 5)
}

func repeat(word string, reps int) {
	for i := 0; i < reps; i++ {
		fmt.Print(word)
	}
}
```


我们为`Sammy`参数传入值`word`，为`5`参数传入值`reps`。 这些值按照给定的顺序与每个参数相对应。`repeat`函数有一个`for`循环，它将执行由`reps`参数指定的循环次数。 对于每次迭代，打印`word`参数的值。

下面是程序的输出：

```
Output  SammySammySammySammySammy
```

如果有一组参数的值都相同，则可以省略每次指定类型。让我们创建一个小程序，它接受参数`x`、`y`和`z`，这些参数都是`int`值。我们将创建一个函数，将不同配置中的参数添加到一起。这些的总和将由函数打印出来。然后我们将调用该函数并将数字传递给该函数。

add_numbers.go

```go
package main

import "fmt"

func main() {
	addNumbers(1, 2, 3)
}

func addNumbers(x, y, z int) {
	a := x + y
	b := x + z
	c := y + z
	fmt.Println(a, b, c)
}
```


当我们为`addNumbers`创建函数签名时，我们不需要每次都指定类型，而只需要在最后指定。

我们为`1`参数传入了数字`x`，为`2`参数传入了数字`y`，为`3`参数传入了数字`z`。这些值按照给定的顺序与每个参数对应。

程序根据我们传递给参数的值进行以下数学运算：

```
a = 1 + 2
b = 1 + 3
c = 2 + 3
```

该函数还打印`a`、`b`和`c`，根据这个数学公式，我们预计`a`等于`3`，`b`等于`4`，`c`等于`5`。让我们运行程序：

```bash
go run add_numbers.go
```


```
Output  3 4 5
```

当我们将`1`、`2`和`3`作为参数传递给`addNumbers()`函数时，我们会收到预期的输出。

参数是通常定义为函数定义中的变量的参数。当你运行这个方法时，可以给它们赋值，把参数传递给函数。

## [返回一个值]

你可以将一个参数值传递给一个函数，函数也可以产生一个值。

一个函数可以用`return`语句产生一个值，这将退出一个函数，并可选地将一个表达式传递回调用者。还必须指定返回数据类型。

到目前为止，我们在函数中使用了`fmt.Println()`语句而不是`return`语句。让我们创建一个程序，它将返回一个变量，而不是打印。

在一个名为`double.go`的新文本文件中，我们将创建一个程序，将参数`x`加倍并返回变量`y`。我们发出一个调用来打印`result`变量，它是通过运行传递了`double()`的`3`函数形成的：

double.go

```go
package main

import "fmt"

func main() {
	result := double(3)
	fmt.Println(result)
}

func double(x int) int {
	y := x * 2
	return y
}
```


我们可以运行程序并查看输出：

```bash
go run double.go
```


```
Output  6
```

整数`6`作为输出返回，这是我们通过将`3`乘以`2`所期望的。

如果一个函数指定了一个返回值，你必须在代码中提供一个返回值。 如果不这样做，您将收到一个编译错误。

我们可以用return语句注释掉这一行来证明这一点：

double.go

```go
package main

import "fmt"

func main() {
	result := double(3)
	fmt.Println(result)
}

func double(x int) int {
	y := x * 2
	// return y
}
```


现在，让我们再次运行该程序：

```bash
go run double.go
```


```
Output  ./double.go:13:1: missing return at end of function
```

如果不使用`return`语句，程序将无法编译。

函数在遇到`return`语句时立即退出，即使它们不在函数的末尾：

return_loop.go

```go
package main

import "fmt"

func main() {
	loopFive()
}

func loopFive() {
	for i := 0; i < 25; i++ {
		fmt.Print(i)
		if i == 5 {
			// Stop function at i == 5
			return
		}
	}
	fmt.Println("This line will not execute.")
}
```


在这里，我们遍历`for`循环，并告诉循环运行`25`迭代。 然而，在`for`循环中，我们有一个条件`if`语句，它检查`i`的值是否等于`5`。 如果是的话，我们会发布一个`return`声明。因为我们在`loopFive`函数中，所以在函数中的任何一点的任何`return`都将退出函数。因此，我们永远不会到达这个函数的最后一行来打印语句`This line will not execute.`。

在`return`循环中使用`for`语句结束函数，因此循环外部的行将不会运行。相反，如果我们使用了[`break`语句]，那么只有循环会在那个时候退出，最后一行`fmt.Println()`会运行。

`return`语句退出一个函数，并且如果在函数签名中指定，则可以返回一个值。

## [返回多个值]

可以为一个函数指定多个返回值。让我们检查一下`repeat.go`程序，让它返回两个值。第一个将是重复的值，如果`reps`参数不是大于`0`的值，则第二个将是错误：

repeat.go

```go
package main

import "fmt"

func main() {
	val, err := repeat("Sammy", -1)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(val)
}

func repeat(word string, reps int) (string, error) {
	if reps <= 0 {
		return "", fmt.Errorf("invalid value of %d provided for reps. value must be greater than 0.", reps)
	}
	var value string
	for i := 0; i < reps; i++ {
		value = value + word
	}
	return value, nil
}
```


`repeat`函数做的第一件事是检查`reps`参数是否是有效值。任何不大于`0`的值都将导致错误。因为我们传入了`-1`的值，所以代码的这个分支将执行。请注意，当我们从函数返回时，我们必须提供`string`和`error`返回值。由于提供的参数导致错误，我们将为第一个返回值传回一个空字符串，为第二个返回值传回错误。

在`main()`函数中，我们可以通过声明两个新变量`value`和`err`来接收两个返回值。因为返回的结果中可能有错误，所以我们想在继续执行程序之前检查一下是否收到了错误。在这个例子中，我们收到了一个错误。 我们打印出错误和`return`函数的`main()`以退出程序。

如果没有错误，我们将打印出函数的返回值。

注意：最佳做法是只返回两个或三个值。此外，您应该始终将所有错误作为函数的最后一个返回值返回。

运行该程序将导致以下输出：

```
Output  invalid value of -1 provided for reps. value must be greater than 0.
```

在本节中，我们回顾了如何使用`return`语句从函数中返回多个值。

## [结论]

函数是在程序中执行操作的指令代码块，有助于使代码可重用和模块化。

要了解更多关于如何使代码更加模块化的信息，您可以阅读我们的指南[如何在Go中编写包]。