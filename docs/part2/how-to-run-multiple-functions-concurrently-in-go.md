# 如何在 Go 中同时运行多个函数

## 导言

Go 语言最受欢迎的功能之一是它对[*并发性*](https://en.wikipedia.org/wiki/Concurrency_(computer_science))的一流支持，即程序可以同时做多件事。随着计算机从更快地运行单一代码流转向同时运行更多代码流，并发运行代码的能力正成为编程的一个重要部分。为了更快地运行程序，程序员需要设计并发运行的程序，以便程序的每个并发部分都能独立运行。Go 的两个特性--[goroutines](https://golangdocs.com/goroutines-in-golang)和[channels](https://golangdocs.com/channels-in-golang)--一起使用时，并发运行会变得更容易。goroutines 解决了在程序中设置和运行并发代码的困难，而通道则解决了并发运行的代码之间安全通信的困难。

在本教程中，你将探索 goroutines 和通道。首先，您将创建一个使用 goroutines 同时运行多个函数的程序。然后，为程序添加通道，以便在运行的 goroutines 之间进行通信。最后，你将在程序中添加更多的 goroutines，以模拟一个由多个工作 goroutines 运行的程序。

## 先决条件

要学习本教程，您需要

- 已安装 Go 1.16 或更高版本。要进行此设置，请按照操作系统的 "[如何安装 Go]"教程进行操作。
- 熟悉 Go 函数，可参阅[如何在 Go 中定义和调用函数]教程。

## 使用 Gooutines 同时运行函数

在现代计算机中，处理器（或[CPU](https://en.wikipedia.org/wiki/Central_processing_unit)）的设计是为了同时运行尽可能多的代码流。这些处理器有一个或多个 "内核"，每个内核能同时运行一个代码流。因此，程序能同时使用的内核越多，程序运行的速度就越快。然而，为了让程序充分利用[多核](https://en.wikipedia.org/wiki/Multi-core_processor)带来的速度提升，程序需要能够被分割成多个代码流。将程序拆分成多个部分可能是编程中更具挑战性的事情之一，但 Go 的设计就是为了让这一切变得更容易。

Go 实现这一功能的方法之一是使用一种名为*goroutines* 的功能。goroutine 是一种特殊类型的函数，它可以在其他 goroutine 同时运行的情况下运行。当一个程序被设计为同时运行多个代码流时，该程序就被设计为[*并发*](https://en.wikipedia.org/wiki/Concurrency_(computer_science))运行。通常情况下，当一个函数被调用时，它会在其后的代码继续运行之前完全结束运行。这被称为在 "前台 "运行，因为在它结束之前，程序无法执行其他任何操作。使用 goroutine 时，函数调用将立即继续运行下一段代码，而 goroutine 则在 "后台 "运行。当代码在完成之前不会阻止其他代码运行时，它就被认为是在后台运行。

goroutine 的强大之处在于，每个 goroutine 可以同时在一个处理器内核上运行。如果你的电脑有四个处理器内核，而你的程序有四个 goroutines，那么所有四个 goroutines 都可以同时运行。当多个代码流同时在不同的内核上运行时，就叫做[*并行*](https://en.wikipedia.org/wiki/Parallel_computing)运行。

要直观了解并发与并行的区别，请看下图。当处理器运行一个函数时，它并不总是一次性从开始运行到结束。有时，当一个函数在等待其他事情发生（如读取文件）时，操作系统会在 CPU 内核上交错运行其他函数、例行程序或其他程序。该图显示了为并发设计的程序如何在单核和多核上运行。它还显示了并行运行时，与在单核心上运行时相比，在相同的时间范围内可以容纳更多的程序段（如图所示，9 个垂直程序段）。

![Diagram split into two columns, labeled Concurrency and Parallelism. The Concurrency column has a single tall rectangle, labeled CPU core, divided into stacked sections of varying colors signifying different functions. The Parallelism column has two similar tall rectangles, both labeled CPU core, with each stacked section signifying different functions, except it only shows goroutine1 running on the left core and goroutine2 running on the right core.](https://assets.digitalocean.com/articles/68067/diagram2.png)

图中左列标有 "并发"，显示了围绕并发设计的程序如何在单个 CPU 内核上运行：先运行`goroutine1` 的一部分，然后运行另一个函数、goroutine 或程序，接着运行`goroutine2`，然后再运行`goroutine1`，以此类推。对用户来说，这就好像程序在同时运行所有函数或 goroutine，尽管它们实际上是一个接一个地分成小部分运行的。

图表右侧标有 "并行性 "的一栏显示了同一程序如何在具有两个 CPU 内核的处理器上并行运行。第一个 CPU 内核显示`goroutine1`与其他函数、goroutine 或程序穿插运行，而第二个 CPU 内核显示`goroutine2`与该内核上的其他函数或 goroutine 一起运行。有时，`goroutine1`和`goroutine2`同时运行，只是运行在不同的 CPU 内核上。

这张图还展示了 Go 的另一个强大特性--[*可扩展性*](https://en.wikipedia.org/wiki/Scalability)。当一个程序可以在从只有几个处理器内核的小型计算机到拥有几十个内核的大型服务器上运行，并充分利用这些额外资源时，它就是可扩展的。从图中可以看出，通过使用 goroutines，你的并发程序可以在单个 CPU 内核上运行，但随着 CPU 内核的增加，更多的 goroutines 可以并行运行，从而加快程序的运行速度。

要开始使用新的并发程序，请在您选择的位置创建一个`multifunc`目录。你可能已经有了一个项目目录，但在本教程中，你将创建一个名为`projects` 的目录。您可以通过集成开发环境或命令行创建`项目`目录。

如果使用命令行，首先创建`项目`目录并导航到该目录：

```bash
mkdir projects
cd projects
```

在`projects`目录下，使用`mkdir`命令创建程序目录`（multifunc`），然后导航进入：

```bash
mkdir multifunc
cd multifunc
```

进入`multifunc`目录后，用`nano` 或你喜欢的编辑器打开名为`main.go`的文件：

```bash
nano main.go
```

Paste or type the following code in the `main.go` file to get started.

```go title="projects/multifunc/main.go"
package main

import (
 "fmt"
)

func generateNumbers(total int) {
 for idx := 1; idx <= total; idx++ {
  fmt.Printf("Generating number %d\n", idx)
 }
}

func printNumbers() {
 for idx := 1; idx <= 3; idx++ {
  fmt.Printf("Printing number %d\n", idx)
 }
}

func main() {
 printNumbers()
 generateNumbers(3)
}
```

这个初始程序定义了两个函数：`generateNumbers`和`printNumbers`，然后在`主`函数中运行这两个函数。`generateNumbers`函数将要 "生成 "的数字数量作为参数，在本例中为 1 到 3，然后将每个数字打印到屏幕上。`printNumbers`函数还不需要任何参数，但它也会打印出 1 到 3 的数字。

保存`main.go`文件后，使用`go`run 运行它，查看输出结果：

```bash
go run main.go
```

输出结果将与此相似：

```
Output
Printing number 1
Printing number 2
Printing number 3
Generating number 1
Generating number 2
Generating number 3
```

你会看到函数一个接一个地运行，`printNumbers`先运行，`generateNumbers`后运行。

现在，假设`printNumbers`和`generateNumbers`的运行时间各为 3 秒。如果*同步*运行，或者像上一个例子那样一个接一个地运行，程序的运行时间将是 6 秒。首先，`printNumbers`运行三秒，然后`generateNumbers`运行三秒。但在你的程序中，这两个函数是相互独立的，因为它们的运行并不依赖于另一个函数的数据。你可以利用这一点，使用 goroutines 并发运行这两个函数，从而加快这个假设程序的运行速度。当两个函数同时运行时，理论上程序的运行时间可以缩短一半。如果`printNumbers`和`generateNumbers`这两个函数的运行时间都是 3 秒，并且同时启动，那么程序可以在 3 秒内完成。(但实际速度可能会因外部因素而变化，例如计算机有多少个内核或计算机上同时运行着多少个其他程序）。

以 goroutine 方式并发运行函数与同步运行函数类似。要以 goroutine 方式运行函数（与标准同步函数不同），只需在函数调用前添加`go`关键字即可。

不过，为了让程序同时运行这些程序，你还需要做一个额外的改动。你需要添加一种方法，让程序等待两个 goroutines 都运行完毕。如果不等待 goroutines 运行完毕，而`主程序`又已完成，那么 goroutines 就有可能永远无法运行，或者只能运行一部分，而无法完成运行。

要等待函数完成，您需要使用 Go 的[`sync`](https://pkg.go.dev/sync)包中的[`WaitGroup`](https://pkg.go.dev/sync#WaitGroup)。`sync`包包含 "同步原语"，例如`WaitGroup`，用于同步程序的各个部分。在本例中，同步将跟踪两个函数的运行时间，以便退出程序。

`WaitGroup`原始码的工作原理是使用`Add`、`Done` 和`Wait`函数计算需要等待的事件数量。`Add`函数将计数增加一个提供给该函数的数字，而`Done`则将计数减少一个。然后可以使用`Wait`函数等待，直到计数为零，即`Done`被调用的次数足以抵消`Add` 的调用次数。一旦计数为零，`Wait`函数将返回，程序将继续运行。

接下来，更新`main.go`文件中的代码，使用`go`关键字将两个函数作为 goroutines 运行，并在程序中添加`sync.WaitGroup`：

```go title="projects/multifunc/main.go"
package main

import (
 "fmt"
 "sync"
)

func generateNumbers(total int, wg *sync.WaitGroup) {
 defer wg.Done()

 for idx := 1; idx <= total; idx++ {
  fmt.Printf("Generating number %d\n", idx)
 }
}

func printNumbers(wg *sync.WaitGroup) {
 defer wg.Done()

 for idx := 1; idx <= 3; idx++ {
  fmt.Printf("Printing number %d\n", idx)
 }
}

func main() {
 var wg sync.WaitGroup

 wg.Add(2)
 go printNumbers(&wg)
 go generateNumbers(3, &wg)

 fmt.Println("Waiting for goroutines to finish...")
 wg.Wait()
 fmt.Println("Done!")
}
```

声明`WaitGroup` 后，它需要知道要等待多少次。在`主`函数中加入 wg`.Add(2)`，然后再启动 goroutines，这样就可以告诉`wg`等待两次`Done`调用，然后再考虑是否结束该组。如果在开始执行程序之前不这样做，可能会导致事情发生的顺序被打乱，或者由于`wg`不知道应该等待任何`Done`调用而导致代码慌乱。

然后，每个函数都将使用`defer`调用`Done`，以便在函数运行结束后将计数减一。此外，还更新了`主`函数，在`WaitGroup` 中加入了对`Wait` 的调用，因此`主`函数将一直等到两个函数都调用`Done`才继续运行并退出程序。

保存`main.go`文件后，像之前一样使用`go run`运行它：

```bash
go run main.go
```

输出结果将与此相似：

```
Output
Printing number 1
Waiting for goroutines to finish...
Generating number 1
Generating number 2
Generating number 3
Printing number 2
Printing number 3
Done!
```

您的输出结果可能与此处打印的结果不同，甚至可能在每次运行程序时都会发生变化。当两个函数同时运行时，输出结果取决于 Go 和操作系统为每个函数提供的运行时间。有时，每个函数都有足够的时间完全运行，你会看到两个函数不间断地打印它们的整个序列。其他时候，你会看到文本穿插在一起，就像上面的输出一样。

您可以尝试删除`主`函数中的`wg.Wait()`调用，然后用`go run`再次运行程序几次。根据你的电脑情况，你可能会看到`generateNumbers`和`printNumbers`函数的一些输出，但也有可能根本看不到它们的任何输出。移除对`Wait` 的调用后，程序将不再等待这两个函数运行完毕后才继续运行。由于`主`函数在`Wait`函数之后很快就会结束，因此程序很有可能会在 goroutines 运行结束之前到达`主`函数的末尾并退出。当这种情况发生时，你会看到一些数字被打印出来，但不会看到每个函数的所有三个数字。

在本节中，你创建了一个程序，使用`go`关键字同时运行两个 goroutines 并打印一串数字。你还使用了`sync.WaitGroup`，使程序在退出程序前等待这些运行程序结束。

您可能已经注意到`generateNumbers`和`printNumbers`函数没有返回值。在 Go 中，goroutines 不能像标准函数那样返回值。你仍然可以使用`go`关键字调用一个返回值的函数，但这些返回值会被抛出，你将无法访问它们。那么，如果不能返回值，需要在另一个 goroutine 中使用一个 goroutine 的数据时该怎么办呢？解决办法是使用 Go 的一个名为 "通道 "的功能，它允许你从一个 goroutine 向另一个 goroutine 发送数据。

## 使用通道在各程序之间安全通信

并发编程中比较困难的部分之一是在同时运行的程序的不同部分之间进行安全通信。稍有不慎，你可能会遇到只有并发程序才可能出现的问题。例如，当程序的两个部分同时运行时，其中一部分试图更新一个变量，而另一部分同时试图读取该变量，这就可能发生[*数据竞赛*](https://en.wikipedia.org/wiki/Race_condition)。发生这种情况时，读取或写入的顺序可能会被打乱，导致程序的一个或两个部分都使用了错误的值。数据竞赛 "这一名称的由来是程序的两个部分在访问数据时相互 "竞赛"。

虽然在 Go 语言中仍有可能遇到数据竞赛等并发问题，但该语言的设计使避免这些问题变得更加容易。除了 goroutines 之外，通道是另一个让并发更安全、更易用的特性。通道可以被认为是两个或多个不同 goroutine 之间的管道，数据可以通过它发送。一个程序将数据放入管道的一端，另一个程序将同样的数据取出。确保数据安全地从一端传送到另一端的困难部分由我们代为处理。

在 Go 中创建通道与使用内置`make()`函数创建[片]类似。通道的类型声明使用`chan`关键字，后面跟要在通道上发送的[type of data]。例如，要创建一个用于发送`int`值的通道，可以使用`chan int` 类型。如果要创建一个发送`[]byte`值的通道，则应使用`chan []byte` 类型，如图所示：

```go
bytesChan := make(chan []byte)
```

一旦创建了通道，就可以使用箭头所示的`<-`操作符在通道上发送或接收数据。`<-`操作符相对于通道变量的位置决定了你是从通道读取数据还是向通道写入数据。

要写入通道，首先要写入通道变量，然后是`<-`操作符，最后是要写入通道的值：

```go
intChan := make(chan int)
intChan <- 10
```

要从通道中读取数值，首先要将数值输入变量，用`=`或`:=`为变量赋值，然后是`<-`操作符，最后是要读取的通道：

```go
intChan := make(chan int)
intVar := <- intChan
```

为了使这两个操作保持一致，记住`<-`箭头总是指向左边（与`-> 相反）`，箭头指向值的去向可能会有所帮助。在向通道写入时，箭头指向通道。从通道读取时，箭头指向变量所在的通道。

与切片一样，也可以在[`for`loop]中使用`range`关键字读取通道。使用`range`关键字读取通道时，循环的每次迭代都会从通道中读取下一个值，并将其放入循环变量中。然后将继续读取通道，直到通道关闭或`for`循环以其他方式（如`break`）退出：

```go
intChan := make(chan int)
for num := range intChan {
 // Use the value of num received from the channel
 if num < 1 {
  break
 }
}
```

在某些情况下，您可能只想让函数从一个通道读取数据或向一个通道写入数据，而不是同时读取和写入两个通道。为此，可以在`chan`类型声明中添加`<-`操作符。与从通道读写类似，通道类型使用`<-`箭头允许变量将通道限制为只读、只写或同时读写。例如，要定义一个包含`int`值的只读通道，类型声明应为`<-chan int`：

```go
func readChannel(ch <-chan int) {
 // ch is read-only
}
```

如果希望通道只允许写入，可以将其声明为`chan<- int`：

```go
func writeChannel(ch chan<- int) {// ch 只允许写入}
```

请注意，箭头指向通道外用于读取，指向通道内用于写入。如果声明中没有箭头，如`chan int`，则通道既可用于读，也可用于写。

最后，一旦通道不再使用，就可以使用内置的`close()`函数关闭它。这一步非常重要，因为如果在程序中创建了通道，但又多次闲置不用，就会导致所谓的[*内存泄漏*](https://en.wikipedia.org/wiki/Memory_leak)。内存泄漏是指程序创建的内容占用了电脑内存，但在使用完毕后却没有将内存释放回电脑。这会导致程序随着时间的推移慢慢（有时也不那么慢）占用更多内存，就像漏水一样。当使用`make()` 创建一个通道时，计算机的部分内存会被通道占用，然后当调用通道上的`close()`时，这些内存又会还给计算机，用于其他用途。

现在，更新程序中的`main.go`文件，使用`chan int`通道在各程序之间进行通信。`generateNumbers`函数将生成数字并写入通道，而`printNumbers`函数将从通道读取数字并打印到屏幕上。在`主`函数中，您将创建一个新通道作为参数传递给其他函数，然后在通道上使用`close()`关闭它，因为它将不再被使用。`generateNumbers`函数也不应该再是一个 goroutine，因为一旦该函数运行完毕，程序就已经完成了所有需要生成的数字。这样，只有在两个函数都运行完毕之前，才会在通道上调用`close()`函数。

```go title="projects/multifunc/main.go"
package main

import (
 "fmt"
 "sync"
)

func generateNumbers(total int, ch chan<- int, wg *sync.WaitGroup) {
 defer wg.Done()

 for idx := 1; idx <= total; idx++ {
  fmt.Printf("sending %d to channel\n", idx)
  ch <- idx
 }
}

func printNumbers(ch <-chan int, wg *sync.WaitGroup) {
 defer wg.Done()

 for num := range ch {
  fmt.Printf("read %d from channel\n", num)
 }
}

func main() {
 var wg sync.WaitGroup
 numberChan := make(chan int)

 wg.Add(2)
 go printNumbers(numberChan, &wg)

 generateNumbers(3, numberChan, &wg)

 close(numberChan)

 fmt.Println("Waiting for goroutines to finish...")
 wg.Wait()
 fmt.Println("Done!")
}
```

在`generateNumbers`和`printNumbers` 的参数中，你会看到`chan`类型使用了只读和只写类型。由于`generateNumbers`只需将数字写入通道，因此它是只写类型，`<-`箭头指向通道。`printNumbers`只需从通道读取数字，因此它是只读类型，`<-`箭头指向通道之外。

尽管这些类型可以是允许读写的`chan int`，但将它们限制在函数所需的范围内还是很有帮助的，这样可以避免意外导致程序因[*死锁*](https://en.wikipedia.org/wiki/Deadlock)而停止运行。当程序的一部分在等待程序的另一部分做某事，而程序的另一部分也在等待程序的第一部分完成时，就会出现死锁。由于程序的两个部分都在互相等待，程序将永远无法继续运行，就像两个齿轮卡住一样。

由于 Go 中的通道通信方式，死锁可能会发生。当程序的一部分正在向一个通道写入数据时，它会一直等到程序的另一部分从该通道读取数据后才继续。同样，如果一个程序正在从一个通道读取数据，它也会等到程序的另一部分写入该通道后才会继续。程序中等待其他事情发生的部分被称为[*阻塞*](https://en.wikipedia.org/wiki/Blocking_(computing))，因为在其他事情发生之前，它无法继续运行。通道在被写入或读出时会阻塞。因此，如果你有一个函数希望写入一个通道，但却意外地从通道中读出，那么你的程序可能会进入死锁，因为通道永远不会被写入。确保这种情况不会发生是使用`chan<- int`或`<-chan`int 而不只是`chan int` 的原因之一。

更新代码的另一个重要方面是使用`close()`在`generateNumbers` 完成写入后关闭通道。在这个程序中，`close(`) 会导致`printNumbers`中的`for ... range`循环退出。由于使用`range`从一个通道读取数据会一直持续到所读取的通道关闭为止，因此如果不在`numberChan`上调用`close`，`printNumbers`将永远不会结束。如果`printNumbers`永远不会结束，那么当`printNumbers`退出时，`WaitGroup`的`Done`方法也永远不会被`defer`调用。如果`printNumbers` 从未调用`Done`方法，程序本身也不会退出，因为`主`函数中的`WaitGroup`的`Wait`方法永远不会继续。这是另一个死锁的例子，因为`主`函数正在等待永远不会发生的事情。

现在，再次在`main.`go 上使用`go run`命令运行更新后的代码。

```bash
go run main.go
```

您的输出结果可能与下图略有不同，但总体上应该差不多：

```
Output
sending 1 to channel
sending 2 to channel
read 1 from channel
read 2 from channel
sending 3 to channel
Waiting for functions to finish...
read 3 from channel
Done!
```

程序输出显示，`generateNumbers`函数正在生成 1 到 3 的数字，同时将它们写入与`printNumbers` 共享的通道。`printNumbers`收到数字后，会将其打印到屏幕上。在`generateNumbers`生成所有三个数字后，它将退出，允许`主`函数关闭通道并等待`printNumbers`完成。`printNumbers`打印完最后一个数字后，会在`WaitGroup`上调用`Done`，然后程序退出。与之前的输出类似，你看到的确切输出将取决于各种外部因素，例如操作系统或 Go 运行时选择运行特定的 goroutines，但应该相对接近。

使用 goroutines 和通道设计程序的好处在于，一旦你设计好了程序的分割，就可以将其扩展到更多的 goroutines。由于`generateNumbers`只是向一个通道写入数据，因此有多少其他程序从该通道读取数据并不重要。它只会向读取该通道的任何设备发送数字。你可以运行多个`printNumbers`goroutine 来利用这一点，这样每个 goroutine 都会从同一个通道读取数据，并同时处理数据。

现在您的程序已使用通道进行通信，再次打开`main.go`文件并更新您的程序，以便启动多个`printNumbers`goroutine。您需要调整对`wg.Add`的调用，以便每启动一个 goroutine 就添加一个。您不必再担心为调用`generateNumbers`的`WaitGroup`添加一个，因为程序不会在未完成整个函数的情况下继续运行，这与您将其作为一个 goroutine 运行时的情况不同。为确保程序结束时不会减少`WaitGroup`的计数，应删除函数中的`defer wg.Done()`行。接下来，将 goroutine 的编号添加到`printNumbers`中，可以更容易地查看每个 goroutine 是如何读取通道的。增加生成数字的数量也是一个好主意，这样更容易看到数字的分布情况：

```go title="projects/multifunc/main.go"
...

func generateNumbers(total int, ch chan<- int, wg *sync.WaitGroup) {
 for idx := 1; idx <= total; idx++ {
  fmt.Printf("sending %d to channel\n", idx)
  ch <- idx
 }
}

func printNumbers(idx int, ch <-chan int, wg *sync.WaitGroup) {
 defer wg.Done()

 for num := range ch {
  fmt.Printf("%d: read %d from channel\n", idx, num)
 }
}

func main() {
 var wg sync.WaitGroup
 numberChan := make(chan int)

 for idx := 1; idx <= 3; idx++ {
  wg.Add(1)
  go printNumbers(idx, numberChan, &wg)
 }

 generateNumbers(5, numberChan, &wg)

 close(numberChan)

 fmt.Println("Waiting for goroutines to finish...")
 wg.Wait()
 fmt.Println("Done!")
}
```

更新了`main.go`之后，就可以使用`go`run with`main.go` 再次运行程序了。在继续生成数字之前，程序应该会启动三个`printNumbers`程序。程序现在也应该生成五个数字，而不是三个，这样可以更容易地看到三个`printNumbers`goroutines 中的每个数字：

```bash
go run main.go
```

输出结果可能与此类似（尽管您的输出结果可能会有很大差异）：

```
Output
sending 1 to channel
sending 2 to channel
sending 3 to channel
3: read 2 from channel
1: read 1 from channel
sending 4 to channel
sending 5 to channel
3: read 4 from channel
1: read 5 from channel
Waiting for goroutines to finish...
2: read 3 from channel
Done!
```

当你这次查看程序输出时，很有可能会与上面的输出大相径庭。由于有三个`printNumbers`goroutine 在运行，因此决定哪个 goroutine 接收到一个特定的数字存在一定的偶然性。当一个`printNumbers`程序接收到一个数字时，它会花少量时间将该数字打印到屏幕上，而另一个程序则从通道中读取下一个数字并做同样的事情。当一个 goroutine 完成打印数字的工作并准备读取另一个数字时，它将返回并再次读取通道以打印下一个数字。如果没有更多的数字要从通道中读取，它将开始阻塞，直到可以读取下一个数字。一旦`generateNumbers`完成并调用了通道上的`close()`，所有三个`printNumbers`程序都将完成其`范围`循环并退出。当所有三个程序都退出并调用了`WaitGroup` 上的`Done`后，`WaitGroup`的计数将归零，程序也将退出。您还可以尝试增加或减少 goroutines 或生成数字的数量，看看它们对输出有何影响。

使用 goroutines 时，应避免启动过多。理论上，一个程序可以有数百甚至数千个 goroutines。不过，根据程序运行的计算机情况，使用较多的 goroutines 实际上可能会更慢。如果 goroutines 数量过多，程序就有可能陷入[*资源枯竭*](https://en.wikipedia.org/wiki/Starvation_(computer_science))。每次 Go 运行一个 goroutine 的一部分时，除了运行下一个函数中的代码所需的时间外，还需要一点额外的时间来重新开始运行。由于需要额外的时间，计算机在运行每个 goroutine 之间切换所需的时间有可能比实际运行 goroutine 本身所需的时间还要长。当这种情况发生时，我们称之为资源饥饿，因为程序和它的程序运行时得不到所需的资源，或者得到的资源非常少。在这种情况下，降低程序中并发运行的部分数量可能会更快，因为这样可以减少在这些部分之间切换所需的时间，并将更多时间用于运行程序本身。记住程序运行在多少个内核上，是决定使用多少个梭子线的良好起点。

结合使用 goroutines 和通道，可以创建功能非常强大的程序，从小型台式机到大型服务器都能运行。正如你在本节中所看到的，通道可用于在几个 goroutines 到潜在的成千上万个 goroutines 之间进行通信，而只需做很小的改动。如果你在编写程序时考虑到这一点，你就能充分利用 Go 中的并发性，为用户提供更好的整体体验。

## 结论

在本教程中，你使用`go`关键字创建了一个程序，用于启动并发运行的 goroutine，在运行过程中打印出数字。程序运行后，你使用`make(chan int)` 创建了一个新的`int`值通道，然后使用该通道在一个 goroutine 中生成数字，并将它们发送到另一个 goroutine 中，打印到屏幕上。最后，你同时启动了多个 "打印 "goroutine，以此为例说明如何使用通道和 goroutine 来加快多核计算机上的程序运行速度。

如果您有兴趣了解 Go 中并发的更多信息，Go 团队创建的[Effective Go](https://golang.org/doc/effective_go#concurrency)文档将为您提供更详细的介绍。[并发不是并行](https://go.dev/blog/waza-talk)Go》博文也是一篇关于并发与并行关系的有趣后续文章，并发与并行这两个术语有时会被错误地混为一谈。
