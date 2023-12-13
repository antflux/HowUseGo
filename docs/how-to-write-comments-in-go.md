# 介绍

几乎所有编程语言都有为代码添加注释的语法，Go 也不例外。注释是程序中的行，用人类语言解释代码如何工作或为什么这样编写。它们会被编译器忽略，但不会被细心的程序员忽略。注释添加了宝贵的上下文，可以帮助您的协作者以及未来的您避免陷阱并编写更易于维护的代码。

任何包中的普通注释都会解释该代码执行其操作的原因。它们是针对软件包开发人员的注释和警告。*文档注释*总结了包的每个组件的功能及其工作原理，还提供了示例代码和命令用法。它们是其用户的官方包文档。

在本文中，我们将查看一些 Go 包中的一些真实注释，不仅说明注释在 Go 中的外观，还说明它们应该传达什么内容。

## 普通评论

Go 中的注释以两个正斜杠 ( `//`) 开头，后跟一个空格（不是必需的，但惯用的），然后是注释。它可能出现在其相关代码的上方或右侧。当上面时，它会缩进以与代码对齐。

这个 Hello World 程序在其自己的行中包含一条注释：

```go title="hello.go"
package main

import "fmt"

func main() {
 // Say hi via the console
 fmt.Println("Hello, World!")
}
```

!!! Note

    如果您添加的注释与代码不一致，该[`gofmt`](https://pkg.go.dev/cmd/gofmt)工具将修复该问题。该工具随 Go 安装一起提供，可将 Go 代码（包括注释）格式化为通用格式，以便各处的 Go 代码看起来都相同，并且程序员不会争论制表符和空格。作为[Gopher](https://go.dev/blog/gopher)（Go 爱好者的称呼），您应该在编写 Go 代码时不断对其进行格式化——当然是在将其提交给版本控制之前。您可以`gofmt`手动运行 ( `gofmt -w hello.go`)，但配置文本编辑器或 IDE 以在每次保存文件时运行它会更方便。

由于此注释很短，因此它可以显示为代码右侧的内联注释：

```go title="hello.go"
...
    fmt.Println("Hello, World!") // Say hi via the console
... 
```

大多数评论都会单独出现，除非它们像这样非常简短。

较长的注释跨越多行。Go 支持使用和标签来打开和关闭很长注释的C 风格`/*`和`/*`块注释，但这些仅在特殊情况下使用。（稍后会详细介绍。）普通的多行注释以 `//` 开始每一行，而不是使用块注释标记。`/*` `*/`

这是一些带有许多注释的代码，每个注释都正确缩进。一条多行注释被突出显示：

```go title="godo.go"
package main

import "fmt"

const favColor string = "blue" // Could have chosen any color

func main() {
 var guess string
 // Create an input loop
 for {
  // Ask the user to guess my favorite color
  fmt.Println("Guess my favorite color:")

  // Try to read a line of input from the user.
  // Print out an error and exit, if there is one.
  if _, err := fmt.Scanln(&guess); err != nil {
   fmt.Printf("%s\n", err)
   return
  }

  // Did they guess the correct color?
  if favColor == guess {
   // They guessed it!
   fmt.Printf("%q is my favorite color!\n", favColor)
   return
  }
  // Wrong! Have them guess again.
  fmt.Printf("Sorry, %q is not my favorite color. Guess again.\n", guess)
 }
}
```

这些注释大多数实际上都是混乱的。如此小而简单的程序不应该包含这么多注释，而且其中大多数内容都是代码本身显而易见的。您可以相信其他 Go 程序员了解 Go 语法、控制流、数据类型等基础知识。您不需要编写注释来宣布代码将迭代一个切片或将两个浮点数相乘。

然而，其中一条注释很有用。

### 什么是好的注释

任何程序中最有用的注释不是解释代码做什么或如何做，而是解释为什么这样做。有时没有 *为什么* ，即使指出这一点也很有用，就像这个内联注释所做的那样：


```go title="godo.go"
const favColor string = "blue" // Could have chosen any color
```

这个注释说了一些代码不能说的事情：该值`“blue”`是由程序员任意选择的。换句话说，`// Feel free to change this`。

不过，大多数代码确实都有一个 *为什么*。这是Go 标准库的[`net/http`包](https://pkg.go.dev/net/http)中的一个函数，其中包含两个非常有用的注释：

```go title="client.go"
. . .
// refererForURL returns a referer without any authentication info or
// an empty string if lastReq scheme is https and newReq scheme is http.
func refererForURL(lastReq, newReq *url.URL) string {
 // https://tools.ietf.org/html/rfc7231#section-5.5.2
 //   "Clients SHOULD NOT include a Referer header field in a
 //    (non-secure) HTTP request if the referring page was
 //    transferred with a secure protocol."
 if lastReq.Scheme == "https" && newReq.Scheme == "http" {
  return ""
 }
 referer := lastReq.String()
 if lastReq.User != nil {
  // This is not very efficient, but is the best we can
  // do without:
  // - introducing a new method on URL
  // - creating a race condition
  // - copying the URL struct manually, which would cause
  //   maintenance problems down the line
  auth := lastReq.User.String() + "@"
  referer = strings.Replace(referer, auth, "", 1)
 }
 return referer
}
. . .
```

第一个突出显示的注释警告维护人员不要更改下面的代码，因为它是故意编写的以符合 HTTP 协议的 RFC（官方规范）。第二条突出显示的评论承认下面的代码并不理想，暗示维护人员可能会如何尝试改进它，并警告他们这样做的危险。

像这样的注释是必不可少的。他们阻止维护人员无意中引入错误和其他问题，但也可能会邀请他们谨慎地实施新想法。

声明上方的注释`func`也很有帮助，但方式不同。接下来让我们探讨一下这种注释。

## 文档注释

出现在顶级（非缩进）声明（如`package`、`func`、`const`、`var`和 ）上方的注释`type`称为*文档注释*。它们之所以如此命名，是因为它们代表了包及其所有[导出名称](https://go.dev/tour/basics/3)的官方文档。

!!! Note

    在 Go 中，*导出的含义与某些语言中 public* 含义相同：导出的组件是其他包在导入您的包时可能使用的组件。要导出包中的任何顶级名称，您所要做的就是将其大写。

### 文档注释解释 *什么以及如何*

与我们刚刚看到的普通注释不同，文档注释通常确实解释了代码的作用或代码的作用。这是因为它们不是为软件包的维护者准备的，而是为通常不想阅读或贡献代码的用户准备的。

用户通常会在以下三个位置之一阅读您的文档注释：

1. 在本地终端中，通过`go doc`在单个源文件或目录上运行。
2. 在[pkg.go.dev](https://pkg.go.dev/about)上，这是任何公共 Go 包的官方文档中心。
3. 在私人运行的 Web 服务器上，由您的团队使用该[`godoc`](https://pkg.go.dev/golang.org/x/tools/cmd/godoc)工具托管。该工具可让您的团队为私有 Go 包创建自己的参考门户。

开发 Go 包时，您应该为每个导出的名称编写文档注释。（[偶尔也适用于未导出的名称](https://github.com/golang/go/wiki/CodeReviewComments#doc-comments)[`godo`](https://pkg.go.dev/github.com/digitalocean/godo)。）以下是DigitalOcean API 的 Go 客户端库中的一行文档注释：

```go title="godo.go"
// Client manages communication with DigitalOcean V2 API.
type Client struct {
```

像这样的简单文档注释可能看起来没有必要，但请记住，它将与所有其他文档注释一起出现，以全面记录包的每个可用组件。

这是该包中较长的文档注释：

```go title="godo.go"
// Do sends an API request and returns the API response. The API response is JSON decoded and stored in the value
// pointed to by v, or returned as an error if an API error has occurred. If v implements the io.Writer interface,
// the raw response will be written to v, without attempting to decode it.
func (c *Client) Do(ctx context.Context, req *http.Request, v interface{}) (*Response, error) {
. . .
}
```

函数的文档注释应明确指定参数的预期格式（当它不明显时）以及函数返回的数据的格式。他们还可能总结该功能的工作原理。

将函数的文档注释`Do`与函数内的此注释进行比较：

```go title="godo.go"
 // Ensure the response body is fully read and closed
 // before we reconnect, so that we reuse the same TCPConnection.
 // Close the previous response's body. But read at least some of
 // the body so if it's small the underlying TCP connection will be
 // re-used. No need to check for errors: if it fails, the Transport
 // won't reuse it anyway.
```

这就像我们在包裹中看到的注释一样`net/http`。阅读此注释下面的代码的维护者可能会想，“为什么不检查错误？”，然后添加错误检查。但评论解释了为什么他们不需要这样做。它不像文档注释那样是高级别的“*什么”*或*“如何”*。

最高级别的文档注释是包注释。每个包应该只包含一个包注释，该注释给出了包是什么以及如何使用它的高级概述，包括代码和/或命令示例。包注释可以出现在任何源文件中（并且只能出现在该文件中）的`package <name>`声明上方。通常，包注释出现在其自己的文件中，称为`doc.go`.

与我们看过的所有其他注释不同，包注释经常使用`/*`和 ，`*/`因为它们可能很长。这是包注释的开头`gofmt`：

```go
/*
Gofmt formats Go programs.
It uses tabs for indentation and blanks for alignment.
Alignment assumes that an editor is using a fixed-width font.

Without an explicit path, it processes the standard input. Given a file,
it operates on that file; given a directory, it operates on all .go files in
that directory, recursively. (Files starting with a period are ignored.)
By default, gofmt prints the reformatted sources to standard output.

Usage:

gofmt [flags] [path ...]

The flags are:

-d
Do not print reformatted sources to standard output.
If a file's formatting is different than gofmt's, print diffs
to standard output.
. . .
*/
package main
```

那么文档注释的格式又如何呢？它们可以（或必须）具有什么样的结构？

### 文档注释有格式

根据Go 创建者的[一篇旧博客文章：](https://go.dev/blog/godoc)

> Godoc 在概念上与 Python 的 Docstring 和 Java 的 Javadoc 相关，但其设计更简单。godoc 读取的注释不是语言结构（如 Docstring），也不一定有自己的机器可读语法（如 Javadoc）。Godoc 注释只是好的注释，即使 godoc 不存在，您也会想阅读这种注释。

[虽然文档注释没有必需的格式，但它们可以选择使用Go 文档](https://go.dev/doc/comment)中完整描述的“Markdown 的简化子集”格式。在文档注释中，您将编写段落和列表，在缩进块中显示示例代码或命令，提供参考链接等等。当文档注释按照这种格式构造良好时，它们可以呈现为漂亮的网页。

[以下是如何在 Go 中编写您的第一个程序](https://www.digitalocean.com/community/tutorials/how-to-write-your-first-program-in-go)`greeting.go`中添加到扩展 Hello World 程序的一些注释：

```go title="greeting.go"
// This is a doc comment for greeting.go.
//  - prompt user for name.
//   - wait for name
//    - print name.
// This is the second paragraph of this doc comment.
// `gofmt` (and `go doc`) will insert a blank line before it.
package main

import (
 "fmt"
 "strings"
)

func main() {
 // This is not a doc comment. Gofmt will NOT format it.
 //  - prompt user for name
 //   - wait for name
 //    - print name
 // This is not a "second paragraph" because this is not a doc comment.
 // It's just more lines to this non-doc comment.
 fmt.Println("Please enter your name.")
 var name string
 fmt.Scanln(&name)
 name = strings.TrimSpace(name)
 fmt.Printf("Hi, %s! I'm Go!", name)
}
```

{==

上面的注释`package main`是文档评论。它试图使用文档注释格式的段落和列表的概念，但它不太正确。该`gofmt`工具会将其塑造成格式。运行`gofmt greeting.go`将打印以下内容：

==}

```go
// This is a doc comment for greeting.go.
//   - prompt user for name.
//   - wait for name.
//   - print name.
//
// This is the second paragraph of this doc comment.
// `gofmt` (and `go doc`) will insert a blank line before it.
package main

import (
 "fmt"
 "strings"
)

func main() {
 // This is not a doc comment. `gofmt` will NOT format it.
 //  - prompt user for name
 //   - wait for name
 //    - print name
 // This is not a "second paragraph" because this is not a doc comment.
 // It's just more lines to this non-doc comment.
 fmt.Println("Please enter your name.")
 var name string
 fmt.Scanln(&name)
 name = strings.TrimSpace(name)
 fmt.Printf("Hi, %s! I'm Go!", name)
}
```


!!! Notice

    1. 文档注释第一段中列出的项目现已对齐。
    2. 现在，第一段和第二段之间有一个空行。
    3. 其中的评论`main()`尚未格式化，因为`gofmt`我们发现它不是文档评论。（但如前所述，`gofmt`确实将所有注释对齐到与代码相同的缩进。）


运行`go doc greeting.go`将格式化并打印文档注释，但不会打印其中的注释`main()`：

``` title="Output"
This is a doc comment for greeting.go.
  - prompt user for name.
  - wait for name.
  - print name.

This is the second paragraph of this doc comment. `gofmt` (and `go doc`) will
insert a blank line before it.
```

如果您一致且适当地使用此文档注释格式，您的包的用户将会感谢您易于阅读的文档。

阅读有关文档注释的[官方参考页面，](https://go.dev/doc/comment)了解有关如何写好文档的所有信息。

## 快速禁用代码

您是否曾经编写过一些新代码，导致您的应用程序变慢，或者更糟糕的是，破坏了一切？这是另一次使用C风格`/*`和`*/`标签。`/*`您可以通过在有问题的代码之前和`*/`之后放置一个来快速禁用有问题的代码。将这些标签包裹在有问题的代码周围，将其变成*块注释*。然后，当您解决了它引起的任何问题后，您可以通过删除这两个标签来重新启用代码。

```go title="problematic.go"
. . .
func main() {
    x := initializeStuff()
    /* This code is causing problems, so we're going to comment it out for now
    someProblematicCode(x)
    */
    fmt.Println("This code is still running!")
}
```

对于较长的代码块，使用这些标签比添加到每行有问题的代码的开头要方便得多。作为惯例，用于`//`普通注释和文档注释，它们将无限期地存在于您的代码中。仅在测试期间临时使用`/*`和`*/`标签（或用于包注释，如前所述）。不要在程序中长时间保留注释掉的代码片段。

## 结论

通过在所有 Go 程序中编写富有表现力的注释，您可以：

1. 防止你的合作者破坏东西。
2. 帮助未来的自己，他有时会忘记为什么代码最初是这样编写的。
3. 为您的包的用户提供一个参考，他们无需深入研究您的代码即可阅读。
