# 如何在 Go 中使用可变参数函数

### [介绍]

可变*参数函数*是一种接受零个、一个或多个值作为单个参数的函数。虽然可变参数函数并不常见，但它们可以用来使您的代码更清晰、更具可用性。

可变参数函数比看起来更常见。最常见的是包`Println`中的函数。[`fmt`](https://golang.org/pkg/fmt)

```go
func Println(a ...interface{}) (n int, err error)
```

参数前面标注了一组省略号 ( ) 的[函数]`...`被认为是可变参数函数。简号表示提供的参数可以是零、一个或多个值。对于`fmt.Println`包来说，它声明的参数`a`是可变的。

让我们创建一个使用该`fmt.Println`函数并创建零个、一个或多个值的程序：

打印.go

```go
package main

import "fmt"

func main() {
 fmt.Println()
 fmt.Println("one")
 fmt.Println("one", "two")
 fmt.Println("one", "two", "three")
}
```

第一次调用时`fmt.Println`，我们不提交任何参数。第二次调用时，`fmt.Println`仅确定一个参数，其值`one`。然后我们通过`one`和`two`，最后是`one`，`two`和`three`。

让我们使用以下命令运行该程序：

```bash
go run print.go
```

我们将看到以下输出：

```
Output  
one
one two
one two three
```

输出的第一行是空白的。`fmt.Println`这是因为我们第一次调用时没有提交任何参数。`one`第二次打印的值。然后`one`是`two`，最后是`one`，`two`，并且`three`。

现在我们已经了解了如何调用可变参数函数，让我们看看如何定义我们自己的可变参数函数。

## [定义可变参数函数]

`...`我们可以通过在前面的参数中使用简化号( )来定义可变参数函数。让我们一个程序，当人们的名字被发送到函数时，它会向人们打招呼：

你好。

```go
package main

import "fmt"

func main() {
 sayHello()
 sayHello("Sammy")
 sayHello("Sammy", "Jessica", "Drew", "Jamie")
}

func sayHello(names ...string) {
 for _, n := range names {
  fmt.Printf("Hello %s\n", n)
 }
}
```

我们创建了`sayHello`一个只接受一个名为 的参数的函数`names`。`...`该参数是可变参数，因为我们在数据类型之前放置了简洁号 ( )：`...string`。这告诉 Go 函数可以接受零个、一个或多个参数。

该`sayHello`函数接收`names`参数作为。由于数据类型是 a ，因此可以将参数视为函数内部的字符串切片 ( ) 。我们可以使用运算符创建一个循环并迭代字符串切片。[`slice`][`string`]`names``[]string`[`range`]

如果我们运行该程序，我们将得到以下输出：

```
Output  Hello Sammy
Hello Sammy
Hello Jessica
Hello Drew
Hello Jamie
```

请注意，我们第一次调用时没有打印任何内容`sayHello`。这是因为可变参数是空`slice`的`string`。由于我们正在循环切片，因此没有任何可迭代的内容，并且`fmt.Printf`永远不会被调用。

让我们修改程序以检测没有发送任何值：

你好。

```go
package main

import "fmt"

func main() {
 sayHello()
 sayHello("Sammy")
 sayHello("Sammy", "Jessica", "Drew", "Jamie")
}

func sayHello(names ...string) {
 if len(names) == 0 {
  fmt.Println("nobody to greet")
  return
 }
 for _, n := range names {
  fmt.Printf("Hello %s\n", n)
 }
}
```

现在，通过使用[`if`语句]，如果没有传递任何值，则 的长度`names`将为`0`，我们将打印出`nobody to greet`：

```
Output  nobody to greet
Hello Sammy
Hello Sammy
Hello Jessica
Hello Drew
Hello Jamie
```

使用可变参数可以使代码更具可读性。让我们创建一个函数，将单词与指定的分隔符连接在一起。我们将首先创建这个不带可变参数函数的程序，以展示它的读取方式：

加入.go

```go
package main

import "fmt"

func main() {
 var line string

 line = join(",", []string{"Sammy", "Jessica", "Drew", "Jamie"})
 fmt.Println(line)

 line = join(",", []string{"Sammy", "Jessica"})
 fmt.Println(line)

 line = join(",", []string{"Sammy"})
 fmt.Println(line)
}

func join(del string, values []string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

在此程序中，我们将逗号 ( `,`) 作为分隔符传递给`join`函数。然后我们传递一个值的切片来连接。这是输出：

```
Output  Sammy,Jessica,Drew,Jamie
Sammy,Jessica
Sammy
```

由于该函数采用字符串切片作为`values`参数，因此在调用该函数时，我们必须将所有单词包装在一个切片中`join`。这可能会使代码难以阅读。

现在，让我们编写相同的函数，但我们将使用可变参数函数：

加入.go

```go
package main

import "fmt"

func main() {
 var line string

 line = join(",", "Sammy", "Jessica", "Drew", "Jamie")
 fmt.Println(line)

 line = join(",", "Sammy", "Jessica")
 fmt.Println(line)

 line = join(",", "Sammy")
 fmt.Println(line)
}

func join(del string, values ...string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

如果我们运行该程序，我们可以看到得到与前一个程序相同的输出：

```
Output  Sammy,Jessica,Drew,Jamie
Sammy,Jessica
Sammy
```

虽然该`join`函数的两个版本以编程方式执行完全相同的操作，但该函数的可变参数版本在调用时更容易阅读。

## [可变参数顺序]

函数中只能有一个可变参数，并且它必须是函数中定义的最后一个参数。在可变参数函数中以除最后一个参数之外的任何顺序定义参数都会导致编译错误：

加入.go

```go
package main

import "fmt"

func main() {
 var line string

 line = join(",", "Sammy", "Jessica", "Drew", "Jamie")
 fmt.Println(line)

 line = join(",", "Sammy", "Jessica")
 fmt.Println(line)

 line = join(",", "Sammy")
 fmt.Println(line)
}

func join(values ...string, del string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

这次我们将`values`参数放在函数的前面`join`。这将导致以下编译错误：

```
Output  ./join_error.go:18:11: syntax error: cannot use ... with non-final parameter values
```

定义任何可变参数函数时，只有最后一个参数可以是可变参数。

## [争论不断]

到目前为止，我们已经看到可以将零个、一个或多个值传递给可变参数函数。但是，有时我们有一个值切片，并且希望将它们发送到可变参数函数。

让我们看看`join`上一节中的函数，看看会发生什么：

加入.go

```
package main

import "fmt"

func main() {
 var line string

 names := []string{"Sammy", "Jessica", "Drew", "Jamie"}

 line = join(",", names)
 fmt.Println(line)
}

func join(del string, values ...string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

如果我们运行这个程序，我们会收到一个编译错误：

```
Output  ./join-error.go:10:14: cannot use names (type []string) as type string in argument to join
```

尽管可变参数函数会将 的参数转换`values ...string`为字符串切片`[]string`，但我们无法将字符串切片作为参数传递。这是因为编译器需要字符串的离散参数。

为了解决这个问题，我们可以通过在切片后添加一组省略号 ( ) 来*分解*切片`...`，并将其转换为离散参数，并将其传递给可变参数函数：

加入.go

```
package main

import "fmt"

func main() {
 var line string

 names := []string{"Sammy", "Jessica", "Drew", "Jamie"}

 line = join(",", names...)
 fmt.Println(line)
}

func join(del string, values ...string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

这次，当我们调用该`join`函数时，我们`names`通过附加省略号 ( `...`) 来分解切片。

这使得程序现在可以按预期运行：

```
Output  Sammy,Jessica,Drew,Jamie
```

值得注意的是，我们仍然可以传递零个、一个或多个参数，以及我们分解的切片。这是传递我们迄今为止看到的所有变体的代码：

加入.go

```go
package main

import "fmt"

func main() {
 var line string

 line = join(",", []string{"Sammy", "Jessica", "Drew", "Jamie"}...)
 fmt.Println(line)

 line = join(",", "Sammy", "Jessica", "Drew", "Jamie")
 fmt.Println(line)

 line = join(",", "Sammy", "Jessica")
 fmt.Println(line)

 line = join(",", "Sammy")
 fmt.Println(line)

}

func join(del string, values ...string) string {
 var line string
 for i, v := range values {
  line = line + v
  if i != len(values)-1 {
   line = line + del
  }
 }
 return line
}
```

```
Output  Sammy,Jessica,Drew,Jamie
Sammy,Jessica,Drew,Jamie
Sammy,Jessica
Sammy
```

我们现在知道如何将零个、一个或多个参数以及我们分解的切片传递给可变参数函数。

## [结论]

在本教程中，我们了解了可变参数函数如何使您的代码更简洁。虽然您并不总是需要使用它们，但您可能会发现它们很有用：

- 如果您发现您正在创建一个临时切片只是为了传递给函数。
- 当输入参数的数量未知或调用时会发生变化时。
- 使您的代码更具可读性。
