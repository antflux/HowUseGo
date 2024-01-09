# 理解 Go 中的 defer

## 介绍

Go 拥有许多其他编程语言中常见的控制流关键字，如`if`、`switch`、`for` 等。有一个关键字在大多数其他编程语言中都找不到，那就是`defer`，虽然它不太常见，但你很快就会发现它在你的程序中是多么有用。

`defer`语句的主要用途之一是清理资源，如打开的文件、网络连接和[数据库句柄](https://en.wikipedia.org/wiki/Handle_(computing))。当程序使用完这些资源后，必须关闭它们，以避免耗尽程序的限制，并允许其他程序访问这些资源。通过将关闭文件/资源的调用保持在打开调用的附近，`defer`语句使我们的代码更简洁，更不容易出错。

在本文中，我们将学习如何正确使用`defer`语句来清理资源，以及在使用`defer` 时容易犯的几个错误。

## 什么是`defer`声明

`defer`语句将`defer`关键字后面的[函数调用](https://www.digitalocean.com/community/tutorials/how-to-define-and-call-functions-in-go)添加到堆栈中。堆栈中的所有调用都会在添加调用的函数return时被调用。由于调用是放在堆栈中的，因此它们是按照后进先出的顺序被调用的。

让我们通过打印一些文本来看看`defer`是如何工作的：

主程序

```go
package main

import "fmt"

func main() {
 defer fmt.Println("Bye")
 fmt.Println("Hi")
}
```

在`main`函数中，我们有两个语句。第一条语句以`defer`关键字开头，后跟`print`打印输出 的语句`Bye`。打印出下一行`Hi`。

如果我们运行该程序，我们将看到以下输出：

```
Output  Hi
Bye
```

请注意，`Hi`被首先打印出来。这是因为任何前面带有`defer`关键字的语句都要到使用`defer`的函数结束时才会被调用。

让我们再来看看这个程序，这次我们将添加一些注释来帮助说明正在发生的事情


```go title="main.go"
package main

import "fmt"

func main() {
 // defer statement is executed, and places
 // fmt.Println("Bye") on a list to be executed prior to the function returning
 defer fmt.Println("Bye")

 // The next line is executed immediately
 fmt.Println("Hi")

 // fmt.Println*("Bye") is now invoked, as we are at the end of the function scope
}
```

理解`defer`的关键在于，当`defer`语句被执行时，被defer函数的参数会立即被评估。当执行`defer`语句时，它会把后面的语句放在列表中，在函数return之前调用。

虽然这段代码展示了`defer`的运行顺序，但这并不是编写 Go 程序时使用 defer 的典型方式。我们更可能使用`defer`来清理资源，例如文件句柄。接下来让我们看看如何做到这一点。

## 用于`defer`清理资源

使用`defer`清理资源在 Go 中非常常见。我们先来看一个程序，它将字符串写入文件，但不使用`defer`来处理资源清理：

```go title="main.go"
package main

import (
 "io"
 "log"
 "os"
)

func main() {
 if err := write("readme.txt", "This is a readme file"); err != nil {
  log.Fatal("failed to write file:", err)
 }
}

func write(fileName string, text string) error {
 file, err := os.Create(fileName)
 if err != nil {
  return err
 }
 _, err = io.WriteString(file, text)
 if err != nil {
  return err
 }
 file.Close()
 return nil
}
```

在这个程序中，有一个名为`write`的函数会首先尝试创建一个文件。如果出现错误，它将return错误并退出函数。接下来，它会尝试将字符串`This is a readme file（这是一个自述文件`）写入指定文件。如果出现错误，将return错误并退出函数。然后，函数将尝试关闭文件并将资源释放回系统。最后，函数return`nil`，表示函数执行无误。

虽然这段代码可以工作，但存在一个微妙的错误。如果调用`io.WriteString`失败，函数将return，但不会关闭文件并将资源释放回系统。

我们可以通过添加另一条`file.Close()`语句来解决这个问题，这也是在没有`defer`的语言中解决这个问题的方法：

```go title="main.go"
package main

import (
 "io"
 "log"
 "os"
)

func main() {
 if err := write("readme.txt", "This is a readme file"); err != nil {
  log.Fatal("failed to write file:", err)
 }
}

func write(fileName string, text string) error {
 file, err := os.Create(fileName)
 if err != nil {
  return err
 }
 _, err = io.WriteString(file, text)
 if err != nil {
  file.Close()
  return err
 }
 file.Close()
 return nil
}
```

现在，即使调用`io.WriteString`失败，我们仍会关闭文件。虽然这是一个比较容易发现和修复的错误，但如果函数比较复杂，这个错误可能会被遗漏。

我们可以使用`defer`语句来确保，无论在执行过程中采取了哪些分支，我们始终调用`Close`()，而不是添加对`file.` `Close`() 的第二次调用。

下面是使用`defer`关键字的版本：

```go title="main.go"
package main

import (
 "io"
 "log"
 "os"
)

func main() {
 if err := write("readme.txt", "This is a readme file"); err != nil {
  log.Fatal("failed to write file:", err)
 }
}

func write(fileName string, text string) error {
 file, err := os.Create(fileName)
 if err != nil {
  return err
 }
 defer file.Close()
 _, err = io.WriteString(file, text)
 if err != nil {
  return err
 }
 return nil
}
```

这次我们添加了一行代码：`defer file.Close()`。这就告诉编译器，它应该在退出函数`写入`之前执行`file.`Close。

现在我们已经确保，即使将来我们添加了更多代码并创建了另一个退出函数的分支，我们也将始终清理并关闭该文件。

然而，我们通过添加defer引入了另一个错误。我们不再检查`close`方法可能return的潜在错误。这是因为当我们使用`defer` 时，无法将任何return值传递回我们的函数。

在 Go 语言中，在不影响程序行为的情况下多次调用`Close()`被认为是一种安全且可接受的做法。如果`Close()`将return错误，它将在第一次调用时return错误。这样，我们就可以在函数的成功执行路径中显式地调用它。

让我们看看如何既能`defer`调用`close`，又能在遇到错误时报告错误。

```go title="main.go"
package main

import (
 "io"
 "log"
 "os"
)

func main() {
 if err := write("readme.txt", "This is a readme file"); err != nil {
  log.Fatal("failed to write file:", err)
 }
}

func write(fileName string, text string) error {
 file, err := os.Create(fileName)
 if err != nil {
  return err
 }
 defer file.Close()
 _, err = io.WriteString(file, text)
 if err != nil {
  return err
 }

 return file.Close()
}
```

该程序中唯一的变化是最后一行，我们return`file.Close()`。如果对`Close`的调用导致错误，该错误将如期return给调用函数。请记住，我们的`defer file.Close()`语句也将在`return`语句之后运行。这意味着`file.Close()`有可能被调用两次。虽然这并不理想，但却是可以接受的做法，因为它不会对程序产生任何副作用。

但是，如果我们在函数的较早部分（如调用`WriteString` 时）收到错误，函数将return该错误，并且还会尝试调用`file.Close`，因为它被defer了。虽然`file.Close`也可能（而且很可能会）return错误，但这不再是我们关心的问题，因为我们收到的错误更有可能告诉我们一开始出了什么问题。

到目前为止，我们已经了解了如何使用单个`defer`语句来确保正确清理资源。接下来，我们将了解如何使用多个`defer`语句来清理多个资源。

## 多重`defer`语句

一个函数中包含多个`defer`语句是很正常的。让我们创建一个只有`defer`语句的程序，看看引入多个defer语句后会发生什么：

```go title="main.go"
package main

import "fmt"

func main() {
 defer fmt.Println("one")
 defer fmt.Println("two")
 defer fmt.Println("three")
}
```

如果我们运行该程序，我们将收到以下输出：

```
Output  three
two
one
```

请注意，我们调用`defer`语句的顺序正好相反。这是因为每个被调用的defer语句都堆叠在前一个语句之上，然后在函数退出作用域时反向调用*（后进先出*）。

在一个函数中，你可以根据需要调用尽可能多的defer调用，但一定要记住，所有defer调用都将以执行时相反的顺序被调用。

既然我们已经了解了多重defer的执行顺序，那么让我们来看看如何使用多重defer来清理多个资源。我们将创建一个程序，打开一个文件，向其中写入内容，然后再次打开该文件，将内容copy另一个文件中。

```go title="main.go"
package main

import (
 "fmt"
 "io"
 "log"
 "os"
)

func main() {
 if err := write("sample.txt", "This file contains some sample text."); err != nil {
  log.Fatal("failed to create file")
 }

 if err := fileCopy("sample.txt", "sample-copy.txt"); err != nil {
  log.Fatal("failed to copy file: %s")
 }
}

func write(fileName string, text string) error {
 file, err := os.Create(fileName)
 if err != nil {
  return err
 }
 defer file.Close()
 _, err = io.WriteString(file, text)
 if err != nil {
  return err
 }

 return file.Close()
}

func fileCopy(source string, destination string) error {
 src, err := os.Open(source)
 if err != nil {
  return err
 }
 defer src.Close()

 dst, err := os.Create(destination)
 if err != nil {
  return err
 }
 defer dst.Close()

 n, err := io.Copy(dst, src)
 if err != nil {
  return err
 }
 fmt.Printf("Copied %d bytes from %s to %s\n", n, source, destination)

 if err := src.Close(); err != nil {
  return err
 }

 return dst.Close()
}
```

我们添加了一个名为`fileCopy` 的新函数。在这个函数中，我们首先打开要复制的源文件。我们会检查打开文件时是否出错。如果是，我们将`return`错误并退出函数。否则，我们将`defer`关闭刚刚打开的源文件。

接下来，我们创建目标文件。我们会再次检查创建文件时是否出错。如果是，我们将`return`错误信息并退出函数。否则，我们也会`defer`目标文件的`Close()`函数。现在我们有了两个`defer`函数，当函数退出其作用域时，它们将被调用。

现在两个文件都已打开，我们将把源文件中的数据`copy`目标文件中。如果成功，我们将尝试关闭两个文件。如果在尝试关闭任一文件时出现错误，我们将`return`错误并退出函数作用域。

请注意，尽管`defer`也会调用 Close`()`，但我们还是为每个文件明确调用了`Close()`。这是为了确保在关闭文件时出现错误，我们会报告该错误。这还能确保在函数因任何原因提前退出时，例如在两个文件之间复制失败时，每个文件仍会尝试通过defer调用正确关闭。

## 结论

在这篇文章中，我们学习了`defer`语句，以及如何使用它来确保在程序中正确清理系统资源。正确清理系统资源将使你的程序使用更少的内存，运行得更好。要了解有关`defer`的更多信息，请阅读 "处理恐慌 "一文，或浏览我们的 "[如何编写 Go 代码 "系列]