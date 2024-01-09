# 在 Go 中导入软件包

### 导言

在不同项目中借用和共享代码的能力是任何广泛使用的编程语言和整个开源社区的基础。借用代码使程序员能够将大部分时间用于编写满足自身需求的代码，但他们的一些新代码最终往往对他人有用。然后，他们可能会决定将这些可重复使用的部分组织成一个单元，并在自己的团队或更广泛的编程社区中共享。

在 Go 中，可重用代码的基本单位被称为*包*。即使是最简单的 Go 程序也是它自己的软件包，而且可能至少使用了一个其他软件包。在本教程中，您将编写两个小程序：一个使用标准库软件包生成随机数，另一个使用流行的第三方软件包生成 UUID。然后，你还将编写一个更长的程序，比较两个类似的标准库软件包，即使它们的基本名称相同，也要导入和使用这两个软件包。最后，你将使用`goimports`工具查看如何格式化你的导入。

!!! Notice
  在 Go 中还有一种更高级别的可重用代码单元：*模块*。模块是包的版本化集合。您将在后面的文章 "如何使用 Go 模块"中探讨模块。

## [先决条件](https://www.digitalocean.com/community/tutorials/importing-packages-in-go#prerequisites)

在开始本教程之前，您只需安装 Go。请阅读适合您操作系统的教程：

- [如何在 Ubuntu 20.04 上安装围棋]
- [如何在 macOS 上安装 Go 并设置本地编程环境]
- [如何在 Windows 10 上安装 Go 并设置本地编程环境]

## [步骤 1 - 使用标准库软件包]

与大多数语言一样，Go 自带一个可重复使用的内置代码库，您可以用它来完成常见任务。例如，您无需编写自己的代码来格式化和打印字符串，或发送 HTTP 请求。[Go 标准库中](https://pkg.go.dev/std)有用于这两项任务和许多其他任务的包。

[如何用 Go 编写第一个程序]中的程序使用了标准库中的`fmt`和`strings`包。让我们再编写一个程序，使用`math/rand`软件包生成一些随机数。

在`nano`或你喜欢的文本编辑器中打开一个名为`random.go`的新文件：

```bash
nano random.go
```

让我们创建一个程序，打印从 0 到 9 的五个随机整数。将以下内容粘贴到编辑器中：

```go
package main

import "math/rand"

func main() {
 for i := 0; i < 5; i++ {
  println(rand.Intn(10))
 }
}
```

本程序导入[`math/rand`](https://pkg.go.dev/math/rand)软件包，并通过引用其*基本名称* `rand` 来使用它。该名称出现在包中每个 Go 源文件顶部的`包 <pkgname>`声明中。

for 循环的每次迭代都会调用`rand.Intn(10)`，生成一个 0 到 9 之间的随机整数（10 不包含在内），然后将整数打印到控制台。
{==

请注意，调用`println() 时`并未引用软件包名称。这是一个无需导入的[内置函数](https://pkg.go.dev/builtin#println)。通常情况下，你会在这里使用`fmt`软件包中的`fmt` `.Println()`函数，但这个程序使用`println()`来引入内置函数。

==}

保存程序。如果使用的是`nano`，按`CTRL+X`然后按`Y`和`ENTER 键`确认更改。然后运行程序：

```
go run random.go
```

你会看到从 0 到 9 的五个整数：

```
Output
1
7
7
9
1
```

看起来随机数生成器正在工作，但请注意，如果你反复运行程序，它每次都会打印出相同的数字，而不是你所期望的新随机数。这是因为我们没有调用`rand.Seed()`函数用唯一值初始化数字生成器。如果不这样做，软件包的行为就好像调用了`rand.Seed(1)`，因此每次都会生成相同的 "随机 "数字。

因此，每次运行程序时，都需要为数字生成器添加一个唯一的值。程序员通常使用以纳秒为单位的当前时间戳。要获得这个值，你需要使用`时间`包。再次在编辑器中打开`random.go`，然后粘贴以下内容：

```go
package main

import (
 "math/rand"
 "time"
)

func main() {
 now := time.Now()
 rand.Seed(now.UnixNano())
println("Numbers seeded using current date/time:", now.UnixNano())
 for i := 0; i < 5; i++ {
  println(rand.Intn(10))
 }
}
```

导入多个软件包时，可以使用括号创建一个导入块。通过使用 "块"，可以避免在每一行中重复`import`关键字，从而使代码更加简洁。

首先，通过 time`.Now()`函数获取当前系统时间，该函数返回一个`Time`结构。然后将时间传递给`rand.Seed()`函数。该函数使用 64 位整数`(int64`)，因此需要在`now`结构上使用`Time.UnixNano()`方法，以纳秒为单位输入时间。最后，打印随机数生成器的种子时间。

现在保存并再次运行程序：

```bash
go run random.go
```

您应该会看到类似下面的输出：

```
Output
Numbers seeded using current date/time: 1674489465616954000
2
6
3
1
0
```

如果多次运行该程序，每次都会看到不同的整数，以及用于随机数生成器种子的唯一整数。

让我们再编辑一次程序，以更方便用户的格式打印种子时间。编辑包含第一个`println()`调用的行，使其看起来像这样：

```go
 println("Numbers seeded using current date/time:", now.Format(time.StampNano))
```

现在你正在调用`Time.Format()`方法，并传入`时间`包中定义的多种格式之一。`time.StampNano`常量`（const`）是一个字符串，将它传入`Time.Format()`方法，可以打印出月、日和时间，精确到纳秒。保存并再次运行程序：

```go
gorun random.go
```

```
Output
Numbers seeded using current date/time: Jan 23 10:01:50.721413000
7
6
3
7
3
```

这比看到一个代表自 1970 年 1 月 1 日以来已经过去的纳秒数的巨大整数要好。

如果您的程序需要的不是随机整数，而是[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)（许多程序员将UUID 用作部署中数据块的全局唯一标识符），该怎么办？Go 标准库中没有生成 UUID 的软件包，但社区中却有。现在让我们看看如何下载和使用第三方软件包。

## [步骤 2 - 使用第三方软件包](https://www.digitalocean.com/community/tutorials/importing-packages-in-go#step-2-using-third-party-packages)

生成 UUID 最常用的软件包之一是[`github.com/google/uuid`](https://pkg.go.dev/github.com/google/uuid)。第三方软件包的全称包括代码托管网站（如[github.com](http://github.com/)）、开发者或组织（如 google）和基础名称（如 uuid）。在导入软件包、阅读[pkg.go.dev](https://pkg.go.dev/) 上的文档和其他地方时，您将使用软件包的全称。但在代码语句中引用时，只能使用基本名称。

在下载软件包之前，你需要初始化模块，这是 Go 管理程序依赖关系及其版本的方式。要初始化模块，请使用`go mod init`并为自己的软件包输入一个全称。如果你想把你的模块放在 GitHub 上，用户名是 "sammy"，你可以这样初始化你的模块：

```bash
go mod init github.com/sammy/random
```

这会创建一个名为`go.mod` 的文件。让我们来看看这个文件：

```bash
cat go.mod
```

```
Output
module github.com/sammy/random

go 1.19
```

该文件必须出现在任何将作为 Go 模块发布的 Go 代码库的根目录下。它必须至少定义模块名称及其所需的 Go 版本。你自己的 Go 版本可能与上面显示的不同。

在本教程中，您不会分发您的模块，但这一步对于下载和使用第三方软件包是必要的。

现在使用`go get`命令下载第三方 UUID 模块：

```bash
go get github.com/google/uuid
```

这将下载最新版本：

```
Output
go: downloading github.com/google/uuid v1.3.0
go: added github.com/google/uuid v1.3.0
```

软件包会放在本地目录`$GOPATH/pkg/mod/。`如果在 shell 中没有明确设置`$GOPATH`，默认值为`$HOME/go`。例如，如果你的本地用户是`sammy`，运行的是 macOS，那么这个软件包就会下载到`/Users/sammy/go/pkg/mod` 目录中。你可以运行`go env GOMODCACHE`查看 Go 将下载的模块放在何处。

让我们来查看新依赖程序自己的`go.mod`文件：

```bash
cat /Users/sammy/go/pkg/mod/github.com/google/uuid@v1.3.0/go.mod
```

```
Output
module github.com/google/uuid
```

看起来这个模块没有第三方依赖；它只使用 Go 标准库中的软件包。

请注意，模块的版本包含在其目录名中。这样，您就可以在一个程序或正在编写的不同程序中，针对同一软件包的多个版本进行开发和测试。

现在再看看你自己的`go.mod`文件：

```bash
cat go.mod
```

```
Output
module github.com/sammy/random

go 1.19

require github.com/google/uuid v1.3.0 // indirect
```

`go get`命令注意到了当前目录下的`go.mod`文件，并对其进行了更新，以反映程序的新依赖关系，包括其版本。现在，你可以在软件包中使用这个软件包了。在文本编辑器中打开一个名为`uuid.go`的新文件，然后粘贴以下内容：

```go
package main

import "github.com/google/uuid"

func main() {
 for i := 0; i < 5; i++ {
  println(uuid.New().String())
 }
}
```

该程序与`random.go` 类似，但它使用`github.com/google/uuid`打印五个 UUID，而不是使用`math/rand`打印五个整数。

保存新文件并运行：

```bash
go run uuid.go
```

输出结果应与此类似：

```
Output
243811a3-ddc6-4e26-9649-060622bba2b0
b8129aa1-3803-4dae-bd9f-6ba8817f44b2
3ae27c71-caa8-4eaa-b8e6-e629b7c1cb49
37e06706-004d-4504-ad37-03c68252bb0f
a2da6904-a6ab-4cc2-849b-d9d25a86e373
```

`github.com/google/uuid`软件包通过使用标准库软件包`crypto/rand` 生成这些 UUID，它与`random.go` 中使用的`math/rand`软件包相似但又不同。如果需要同时使用这两个软件包呢？它们的基本名称都是`rand`，那么如何在代码中引用每个不同的软件包呢？接下来让我们看看这个问题。

## [步骤 3 - 导入同名软件包]

`math/rand`的文档指出，它实际上生成的是伪随机数，"不适合安全敏感工作"。对于此类工作，你可以使用`crypto/rand`代替。但如果整数随机性的质量对你的程序并不重要呢？也许你真的只需要任意数。

你可以编写一个程序来比较这两个`rand`软件包的性能，但在整个程序中你不能用`rand`名称来引用这两个软件包。为了解决这个问题，Go 允许你在导入软件包时为它选择另一个本地名称。

下面介绍如何导入两个具有相同基本名称的软件包：

```go
import (
    “math/rand”
    crand “crypto/rand”
)
```

您可以选择任何您喜欢的别名（只要它不与其他软件包名称相匹配），并将其放在完全限定的软件包名称的左边。在本例中，别名是`crand`。请注意，别名周围没有引号。在包含该导入块的源文件的其余部分中，你可以使用你选择的名称`crand` 访问`crypto/rand`软件包。

您也可以将包导入自己的命名空间（使用`.`作为别名）或空白标识符（使用`_`作为别名）。有关这方面的更多信息，请阅读[Go 文档](https://go.dev/ref/spec#Import_declarations)。

为了说明如何使用同名软件包，让我们创建并运行一个较长的程序，使用这两个软件包生成随机整数，并测量每种情况下所需的时间。这部分是可选的，如果不感兴趣，请跳到[步骤 4]。

### 比较`数学/Rand`和`密码/Rand`（可选）

#### 获取命令行参数

首先，在工作目录下打开第三个名为`compare.go`的新文件，然后粘贴以下程序：

```go
package main

import "os"
import "strconv"

func main() {
 // User must pass in number of integers to generate
 if len(os.Args) < 2 {
  println("Usage:\n")
  println("  go run compare.go <numberOfInts>")
  return
 }
 n, err := strconv.Atoi(os.Args[1])
 if err != nil { // Maybe they didn't pass an integer
  panic(err) 
 }

 println("User asked for " + strconv.Itoa(n) + " integers.")
}
```

这段代码为稍后使用`rand`软件包生成用户指定数量的伪随机整数做好准备。它使用`os`和`strconv`标准库软件包将单个命令行参数转换为整数。如果没有传递参数，程序将打印使用说明并退出。

以`10`为单一参数运行程序，以确保它能正常运行：

```bash
go run compare.go10
```

```
[seconary_label Output]
User asked for 10 integers.。
```

到目前为止，一切顺利。现在，让我们像以前一样使用`math/rand`软件包生成随机整数，但这次要计算生成随机整数所需的时间。

#### 第 1 阶段--衡量`数学/品牌`绩效

删除最后的`println()`语句，代之以下面的内容：

```go
// Phase 1 — Using math/rand
 // Initialize the byte slice
 b1 := make([]byte, n)
 // Get the time
 start := time.Now()
 // Initialize the random number generator
 rand.Seed(start.UnixNano())
 // Generate the pseudo-random numbers
 for i := 0; i < n; i++ {
  b1[i] = byte(rand.Intn(256)) // Where the magic happens!
 }
 // Compute the elapsed time
 elapsed := time.Now().Sub(start)
 // In case n is very large, only print a few numbers
 for i := 0; i < 5; i++ {
  println(b1[i])
 }
 fmt.Printf("Time to generate %v pseudo-random numbers with math/rand: %v\n", n, elapsed)
```

首先，使用内置函数`make()`创建一个空的字节片（`[[]字节`）来保存生成的整数（作为字节）。它的长度是用户要求的整数个数。

然后，你将获取当前时间，并将其作为随机数生成器的种子，就像步骤 1 中的 random`.go`所做的那样。

然后，你将生成`n 个`0 到 255 之间的伪随机整数，将每个整数转换成一个字节，并将其放入你的字节片中。为什么是 0 到 255 之间的整数？因为您将要编写的`加密/字符使用`代码生成的是字节，而不是任何大小的整数，我们应该对数据包进行等量比较。一个字节为 8 位，可以用 0 到 255 之间的无符号整数表示。(事实上，Go 中的[`字节`类型](https://pkg.go.dev/builtin#byte)是`uint8`类型的别名）。

最后，你只打印前五个字节，以防用户要求打印大量整数。看到几个整数也不错，可以让自己相信数字生成器在工作。

运行程序前，别忘了将新软件包添加到`导入`块中：

```go
import (
 "fmt"
 "math/rand"
 "os"
 "strconv"
 "time"
)
```

添加高亮显示的软件包后，运行程序：

```bash
go run compare.go10
```

```
Output
189
203
209
238
235
Time to generate 10 pseudo-random numbers with math/rand: 33.417µs
```

生成 10 个 0 到 255 之间的整数并将其存储在一个字节片中，耗时 33.417 微秒。让我们看看这与`crypto/rand` 的性能相比如何。

#### 第 2 阶段--衡量`加密/品牌`性能

在添加使用`crypto/rand 的`代码之前，请将软件包添加到`导入`块中，如前所述：

```go
import (
 "fmt"
 "math/rand"
 crand "crypto/rand"
 "os"
 "strconv"
 "time"
)
```

然后，将以下代码添加到`main()`函数的末尾：

```go
 // Phase 2 — Using crypto/rand
 // Initialize the byte slice
 b2 := make([]byte, n)
 // Get the time (Note: no need to seed the random number generator)
 start = time.Now()
 // Generate the pseudo-random numbers
 _, err = crand.Read(b2) // Where the magic happens!
 // Compute the elapsed time
 elapsed = time.Now().Sub(start)
 // exit if error
 if err != nil {
  panic(err)
 }
 // In case n is very large, only print a few numbers
 for i := 0; i < 5; i++ {
  println(b2[i])
 }
 fmt.Printf("Time to generate %v pseudo-random numbers with crypto/rand: %v\n", n, elapsed)
```

这段代码尽量与第 1 阶段代码保持一致。您正在生成大小为`n` 的字节片，获取当前时间，生成`n`个字节，计算经过的时间，最后打印五个整数和经过的时间。使用`crypto/rand`软件包时，无需明确为随机数生成器播种。

**注意：** `crypto/rand`还包括一个`Int()`函数，但我们在这里的示例使用的是`Read(` `)`，因为在软件包的文档中，只有一段示例代码使用了该函数。请自行探索`crypto/rand`软件包。

整个`compare.go`程序应该是这样的：

```go
package main

import (
 "fmt"
 "math/rand"
 crand "crypto/rand"
 "os"
 "strconv"
 "time"
)

func main() {
 // User must pass in number of integers to generate
 if len(os.Args) < 2 {
  println("Usage:\n")
  println("  go run compare.go <numberOfInts>")
  return
 }
 n, err := strconv.Atoi(os.Args[1])
 if err != nil { // Maybe they didn't pass an integer
  panic(err)
 }

 // Phase 1 — Using math/rand
 // Initialize the byte slice
 b1 := make([]byte, n)
 // Get the time
 start := time.Now()
 // Initialize the random number generator
 rand.Seed(start.UnixNano())
 // Generate the pseudo-random numbers
 for i := 0; i < n; i++ {
  b1[i] = byte(rand.Intn(256)) // Where the magic happens!
 }
 // Compute the elapsed time
 elapsed := time.Now().Sub(start)
 // In case n is very large, only print a few numbers
 for i := 0; i < 5; i++ {
  println(b1[i])
 }
 fmt.Printf("Time to generate %v pseudo-random numbers with math/rand: %v\n", n, elapsed)

 // Phase 2 — Using crypto/rand
 // Initialize the byte slice
 b2 := make([]byte, n)
 // Get the time (Note: no need to seed the random number generator)
 start = time.Now()
 // Generate the pseudo-random numbers
 _, err = crand.Read(b2) // Where the magic happens!
 // Compute the elapsed time
 elapsed = time.Now().Sub(start)
 // exit if error
 if err != nil {
  panic(err)
 }
 // In case n is very large, only print a few numbers
 for i := 0; i < 5; i++ {
  println(b2[i])
 }
 fmt.Printf("Time to generate %v pseudo-random numbers with crypto/rand: %v\n", n, elapsed)
}
```

运行参数为 10 的程序，使用每个软件包生成 10 个 8 位整数：

```bash
go run compare.go10
```

```
Output
145
65
231
211
250
Time to generate 10 pseudo-random numbers with math/rand: 32.5µs
101
188
250
45
208
Time to generate 10 pseudo-random numbers with crypto/rand: 42.667µs
```

在本例执行中，`math/rand`软件包比`crypto/rand` 稍快。试着运行`compare.go`多次，参数设置为 10。然后尝试生成一千个整数或一百万个整数。哪个软件包的速度更快？

本示例程序旨在展示如何在同一程序中使用两个名称相同、用途相似的软件包。这并不意味着推荐使用其中一个软件包，而不是另一个。如果你想扩展`compare.go`，你可以使用`math/stats`软件包来比较每个软件包生成的字节的随机性。无论你正在编写什么程序，你都可以对不同的软件包进行评估，并选择最适合自己需要的软件包。

最后，让我们看看如何使用`goimports`工具来格式化`导入`声明。

## [步骤 4 - 使用 Goimports]

有时，在编程过程中，你会忘记导入正在使用的软件包，或者删除没有导入的软件包。[`goimports`](https://pkg.go.dev/golang.org/x/tools/cmd/goimports)命令行工具不仅能格式化你的`导入`声明和代码的其他部分，使其成为功能更强大的`gofmt`替代工具，还能为代码引用的软件包添加缺失的`导入`，并删除未使用软件包的`导入`。

该工具默认情况下不与 Go 一起提供，因此请使用`go install` 立即安装：

```bash
go install golang.org/x/tools/cmd/goimports@latest
```

这将把`goimports`二进制文件放到`$GOPATH/bin`目录中。如果你按照教程《[如何在 macOS 上安装 Go 并设置本地编程环境](https://www.digitalocean.com/community/tutorials/how-to-install-go-and-set-up-a-local-programming-environment-on-macos)》（或与你的操作系统相匹配的教程）操作，这个目录应该已经在你的`$PATH` 中了。尝试运行该工具：

```bash
goimports --help
```

如果看不到工具的用法说明，说明`$GOPATH/bin`不在你的`$PATH` 中。请阅读操作系统的[Go 环境设置指南]进行设置。

将`goimports`放入`$PATH` 后，删除`random.go` 中的整个`导入`块。然后，使用`-d`选项运行`goimports`，以显示它要添加的内容的差异：

```bash
goimports -d random.go
```

```
Outputs
diff -u random.go.orig random.go
--- random.go.orig 2023-01-25 16:29:38
+++ random.go 2023-01-25 16:29:38
@@ -1,5 +1,10 @@
 package main
 
+import (
+ "math/rand"
+ "time"
+)
+
 func main() {
  now := time.Now()
  rand.Seed(now.UnixNano())
```

令人印象深刻，但如果第三方软件包通过`go get` 安装在本地，`goimports`也能识别并添加它们。删除`uuid.go`中的`导入`，然后运行`goimports`：

```bash
goimports -d uuid.go
```

```
Output
diff -u uuid.go.orig uuid.go
--- uuid.go.orig 2023-01-25 16:32:56
+++ uuid.go 2023-01-25 16:32:56
@@ -1,8 +1,9 @@
 package main
 
+import "github.com/google/uuid"
+
 func main() {
  for i := 0; i < 5; i++ {
   println(uuid.New().String())
  }
 }
```

现在编辑`uuid.go`并

1. 为`math/rand` 添加`导入`，代码中没有使用。
2. 将内置`println()`函数改为`fmt.Println()`，但不要添加`导入 "fmt"。`

```go title="uuid.go"
package main

import "math/rand"

func main() {
 for i := 0; i < 5; i++ {
  fmt.Println(uuid.New().String())
 }
}
```

保存文件并再次运行`goimports`：

```bash
goimports -d uuid.go
```

```
Output
diff -u uuid.go.orig uuid.go
--- uuid.go.orig 2023-01-25 16:36:28
+++ uuid.go 2023-01-25 16:36:28
@@ -1,10 +1,13 @@
 package main
 
-import "math/rand"
+import (
+ "fmt"
 
+ "github.com/google/uuid"
+)
+
 func main() {
  for i := 0; i < 5; i++ {
   fmt.Println(uuid.New().String())
  }
 }
```

该工具不仅能添加缺失的导入，还能删除不必要的导入。还要注意的是，它将导入内容放在括号内的代码块中，而不是在每一行都使用`导入`关键字。

要将更改写入`uuid.go`（或任何文件），而不是输出到终端，请使用带有`-w`选项的`goimports`：

```bash
goimports -w uuid.go
```

在保存`*.go`文件时，您应该配置文本编辑器或集成开发环境调用`goimports`，这样您的源文件就能始终保持格式正确。如前所述，`goimports`取代了`gofmt`，因此如果你的文本编辑器已经在使用`gofmt`，请将其配置为使用`goimports`。

`goimports`的另一项功能是为您的导入执行特定的排序顺序。手动调整导入顺序既繁琐又容易出错，因此请让`goimports`帮您处理这个问题。

如果 Go 团队更改了 Go 源文件的官方格式，他们会更新`goimports`以反映这些更改，因此您应该定期更新`goimports`以确保您的代码始终符合最新标准。

## [结论](https://www.digitalocean.com/community/tutorials/importing-packages-in-go#conclusion)

在本教程中，您使用流行的软件包创建并运行了两个不到十行的程序，以帮助您生成随机数和 UUID。Go 生态系统中有大量编写精良的软件包，因此用 Go 编写新程序应该是一件愉快的事，而且你会发现自己编写满足特定需求的实用程序所花费的精力比你想象的要少得多。
