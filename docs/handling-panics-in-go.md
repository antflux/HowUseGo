# 在 Go 中创建自定义错误

### [导言](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#introduction)

程序遇到的错误可分为两大类：程序员已预料到的错误和程序员未预料到的错误。我们在前两篇关于[错误处理](https://www.digitalocean.com/community/tutorials/handling-errors-in-go)的文章中介绍的`错误`接口主要处理的是我们在编写 Go 程序时预期会出现的错误。`错误`接口甚至允许我们确认函数调用中出现错误的罕见可能性，以便在这种情况下做出适当的响应。

恐慌属于第二类错误，是程序员未曾预料到的。这些意外错误会导致程序自发终止并退出正在运行的 Go 程序。常见错误往往是造成恐慌的原因。在本教程中，我们将研究常见操作在 Go 中产生恐慌的几种方式，并了解避免这些恐慌的方法。我们还将使用[`defer`](https://www.digitalocean.com/community/tutorials/understanding-defer-in-go)语句和`recover`函数来捕获 panic，以免它们意外终止我们正在运行的 Go 程序。

## [了解恐慌](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#understanding-panics)

Go 中的某些操作会自动返回恐慌并停止程序。常见的操作包括[数组](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#arrays)索引超出其容量、执行类型断言、调用 nil 指针上的方法、错误使用互斥，以及尝试使用关闭的通道。这些情况大多源于编程时所犯的错误，而编译器在编译程序时无法检测到这些错误。

由于恐慌包含了对解决问题有用的细节，因此开发人员通常将恐慌作为程序开发过程中出现错误的标志。

### [越界恐慌](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#out-of-bounds-panics)

当您尝试访问的索引超出片的长度或数组的容量时，Go 运行时将产生恐慌。

下面的示例犯了一个常见错误，即尝试使用`len`内置函数返回的片段长度访问片段的最后一个元素。试着运行这段代码，看看为什么会产生恐慌：

```go
package main

import (
	"fmt"
)

func main() {
	names := []string{
		"lobster",
		"sea urchin",
		"sea cucumber",
	}
	fmt.Println("My favorite sea creature is:", names[len(names)])
}
```

输出结果如下

```
Output
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
main.main()
	/tmp/sandbox879828148/prog.go:13 +0x20
```

恐慌输出的名称提供了一个提示：`恐慌：运行时错误：索引超出范围`。我们创建了一个包含三个海洋生物的片段。然后，我们尝试使用内置函数`len`用片段的长度为片段建立索引，从而获取片段的最后一个元素。请记住，片段和数组都是基于零的，因此第一个元素为零，而片段中的最后一个元素位于索引`2`。由于我们试图访问位于第三个索引`3` 处的片段，片段中没有元素可以返回，因为它超出了片段的边界。运行时别无选择，只能终止并退出，因为我们要求它做一些不可能的事情。在编译过程中，Go 也无法证明这段代码会尝试这样做，因此编译器无法捕捉到这一点。

请注意，随后的代码并没有运行。这是因为恐慌是完全停止 Go 程序执行的事件。产生的消息包含多种信息，有助于诊断恐慌的原因。

## [恐慌剖析](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#anatomy-of-a-panic)

恐慌由一条显示恐慌原因的消息和[堆栈跟踪](https://en.wikipedia.org/wiki/Stack_trace)组成，堆栈跟踪可帮助您找到产生恐慌的代码位置。

任何恐慌的第一部分都是信息。它总是以 panic`:` 字符串开头，后面的字符串根据恐慌的原因而有所不同。上一个练习中的恐慌信息是

```
panic: runtime error: index out of range [3] with length 3
```

panic`:`前缀后面的字符串`runtime error` `:`告诉我们，这个 panic 是由语言运行时产生的。这个恐慌告诉我们，我们试图使用的索引`[3]`超出了片段长度`3` 的范围。

该信息之后是堆栈跟踪。堆栈跟踪形成了一个映射，我们可以根据这个映射准确地找到产生恐慌时正在执行的代码行，以及这些代码是如何被前面的代码调用的。

```
goroutine 1 [running]:
main.main()
	/tmp/sandbox879828148/prog.go:13 +0x20
```

上例中的堆栈跟踪显示，我们的程序在`/tmp/sandbox879828148/prog.go`文件的第 13 行产生了恐慌。它还告诉我们，这个恐慌是在`main`软件包的 main(`)`函数中产生的。

堆栈跟踪被分成不同的块--程序中的每个[goroutine](https://tour.golang.org/concurrency/1)都有一个块。每个 Go 程序的执行都由一个或多个 goroutine 来完成，每个 goroutine 都可以独立地同时执行 Go 代码的一部分。每个程序块都以`goroutine X [state]:` 开头。头文件给出了 goroutine 的 ID 号以及发生恐慌时的状态。在头部之后，堆栈跟踪会显示恐慌发生时程序正在执行的函数，以及执行该函数的文件名和行号。

上例中的恐慌是由于对片段的越界访问引起的。当方法调用未设置的指针时，也会产生恐慌。

## [无接收器](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#nil-receivers)

Go 编程语言使用指针来指代运行时存在于计算机内存中的某种类型的特定实例。指针可以取值为`nil`，表示没有指向任何东西。当我们试图调用一个为`nil` 的指针上的方法时，Go 运行时会产生恐慌。同样，当调用接口类型变量的方法时，也会产生恐慌。要查看这些情况下产生的恐慌，请尝试下面的示例：

```go
package main

import (
	"fmt"
)

type Shark struct {
	Name string
}

func (s *Shark) SayHello() {
	fmt.Println("Hi! My name is", s.Name)
}

func main() {
	s := &Shark{"Sammy"}
	s = nil
	s.SayHello()
}
```

产生的恐慌将是这样的

```
Output
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xdfeba]

goroutine 1 [running]:
main.(*Shark).SayHello(...)
	/tmp/sandbox160713813/prog.go:12
main.main()
	/tmp/sandbox160713813/prog.go:18 +0x1a
```

在本例中，我们定义了一个名为`Shark` 的结构体。`Shark`的指针接收器上定义了一个名为`SayHello`的方法，该方法将在调用时向标准输出端打印问候语。在`主`函数的主体中，我们创建了一个`Shark`结构的新实例，并使用`&`运算符请求一个指向它的指针。这个指针被赋值给`s`变量。然后，我们使用`s` `=` `nil`语句将`s`变量重新赋值为`nil`。最后，我们尝试调用变量`s` 的`SayHello`方法。我们没有收到来自 Sammy 的友好信息，而是收到了一个恐慌信息，提示我们试图访问一个无效的内存地址。由于`s`变量为`nil`，当`SayHello`函数被调用时，它试图访问`*Shark`类型的`Name`字段。 由于这是一个指针接收器，而在这种情况下接收器为`nil`，因此会出现恐慌，因为它不能对`nil`指针进行反引用。

虽然我们在本例中明确地将`s`设置为`nil`，但实际上这种情况发生得并不那么明显。当你看到涉及`取消引用 nil 指针`的恐慌时，请确保你已经正确地分配了你可能创建的任何指针变量。

由 nil 指针和越界访问产生的恐慌是运行时经常出现的两种恐慌。也可以使用内置函数手动生成恐慌。

## [使用`panic`内置功能](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#using-the-panic-builtin-function)

我们还可以使用`panic`内置函数生成自己的 panic。该函数以单个字符串为参数，即 panic 将产生的信息。通常情况下，该信息比重写代码返回错误信息要简洁得多。此外，我们还可以在自己的软件包中使用该函数，向开发人员说明他们在使用我们软件包的代码时可能犯了错误。只要有可能，最佳做法是尽量向软件包的用户返回`错误`值。

运行此代码，查看从另一个函数调用的函数产生的恐慌：

```go
package main

func main() {
	foo()
}

func foo() {
	panic("oh no!")
}
```

恐慌输出结果如下

```
Output
panic: oh no!

goroutine 1 [running]:
main.foo(...)
	/tmp/sandbox494710869/prog.go:8
main.main()
	/tmp/sandbox494710869/prog.go:4 +0x40
```

在这里，我们定义了一个函数`foo`，用字符串`"哦，不！"`调用`panic`内置函数。该函数由我们的`主`函数调用。请注意输出中的信息 panic`: oh no!`和堆栈跟踪显示的单个 goroutine，堆栈跟踪中有两行：一行是`main()`函数，一行是我们的`foo(`) 函数。

我们发现，恐慌似乎会在程序生成时终止程序。当有打开的资源需要适当关闭时，这就会产生问题。Go 提供了一种机制，即使在出现恐慌时，也能始终执行某些代码。

## [延迟功能](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#deferred-functions)

即使在运行时正在处理 panic 时，您的程序也可能有必须妥善清理的资源。Go 允许你推迟函数调用的执行，直到其调用函数执行完毕。推迟执行的函数即使在出现恐慌时也能运行，它是一种安全机制，可防止恐慌的混乱性质。函数延迟的方法是像往常一样调用函数，然后在整个语句前加上`defer`关键字，如`defer sayHello()`。运行此示例，看看如何在产生恐慌的情况下仍能打印消息：

```go
package main

import "fmt"

func main() {
	defer func() {
		fmt.Println("hello from the deferred function!")
	}()

	panic("oh no!")
}
```

本例的输出结果如下：

```
Output
hello from the deferred function!
panic: oh no!

goroutine 1 [running]:
main.main()
	/Users/gopherguides/learn/src/github.com/gopherguides/learn//handle-panics/src/main.go:10 +0x55
```

在本例的`主`函数中，我们首先调用了一个匿名函数，该匿名函数会打印`"hello from the deferred function!"（来自延迟函数的你好！`）消息。然后，`主`函数立即使用`panic`函数产生恐慌。在这个程序的输出中，我们首先看到延迟函数被执行并打印了信息。随后是我们在`main` 中产生的恐慌。

递延函数提供了防止惊恐的保护。在延迟函数中，Go 还为我们提供了一个机会，让我们可以使用另一个内置函数阻止恐慌终止我们的 Go 程序。

## [处理恐慌](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#handling-panics)

恐慌只有一种恢复机制--内置`恢复`函数。该函数允许你在恐慌通过调用堆栈时拦截它，防止它意外终止你的程序。它有严格的使用规则，但在生产应用程序中却非常宝贵。

由于它是`内置`软件包的一部分，因此无需导入任何其他软件包即可调用`恢复`功能：

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	divideByZero()
	fmt.Println("we survived dividing by zero!")

}

func divideByZero() {
	defer func() {
		if err := recover(); err != nil {
			log.Println("panic occurred:", err)
		}
	}()
	fmt.Println(divide(1, 0))
}

func divide(a, b int) int {
	return a / b
}
```

此示例将输出

```
Output
2009/11/10 23:00:00 panic occurred: runtime error: integer divide by zero
we survived dividing by zero!
```

本例中的`主`函数调用了一个我们定义的函数`divideByZero`。在这个函数中，我们`延迟`调用一个匿名函数，该函数负责处理执行`divideByZero` 时可能出现的任何恐慌。在这个延迟的匿名函数中，我们调用`恢复`内置函数，并将其返回的错误赋值给一个变量。如果`divideByZero`陷入恐慌，这个`错误`值将被设置，否则将为`零`。通过比较`err`变量和`nil`，我们可以检测到是否发生了恐慌，在这种情况下，我们使用`log.Println`函数记录恐慌，就像记录其他`错误`一样。

在这个递延匿名函数之后，我们调用另一个我们定义的函数`divide`，并尝试使用`fmt.Println 打印`其结果。所提供的参数会导致`divide`执行除以零的除法运算，从而产生恐慌。

在这个示例的输出中，我们首先看到的是匿名函数恢复恐慌的日志信息，然后是`除以零后我们幸存下来`的信息`！`我们确实做到了。我们确实做到了这一点，这要归功于`恢复`内置函数阻止了可能会终止 Go 程序的灾难性恐慌。

从`recover()`返回的`err`值与调用`panic()` 时提供的值完全相同。因此，确保在未发生 panic 时`err`值为零至关重要。

## [通过`恢复`检测恐慌](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#detecting-panics-with-recover)

`恢复`函数依靠错误值来判断是否发生了 panic。由于`panic`函数的参数是空接口，因此它可以是任何类型。任何接口类型（包括空接口）的零值都是`nil`。正如本例所示，必须注意避免将`nil`作为`panic`的参数：

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	divideByZero()
	fmt.Println("we survived dividing by zero!")

}

func divideByZero() {
	defer func() {
		if err := recover(); err != nil {
			log.Println("panic occurred:", err)
		}
	}()
	fmt.Println(divide(1, 0))
}

func divide(a, b int) int {
	if b == 0 {
		panic(nil)
	}
	return a / b
}
```

这将输出

```
Output
we survived dividing by zero!
```

这个示例与上一个涉及`恢复`的示例相同，只是稍作修改。如果`除数` `b` 等于`0`，它将使用参数为`nil` 的`panic`内置函数产生恐慌。尽管`除法`函数产生了恐慌，但这次的输出并不包括显示恐慌发生的日志信息。因此，确保`panic`内置函数的参数不为`nil` 非常重要。

## [结论](https://www.digitalocean.com/community/tutorials/handling-panics-in-go#conclusion)

我们已经了解了在 Go 中创建恐慌`的`多种方法，以及如何使用内置的`recover 恢复`恐慌。虽然您不一定会亲自使用`panic`，但正确地恢复 panic 是使 Go 应用程序为生产就绪的重要一步。
