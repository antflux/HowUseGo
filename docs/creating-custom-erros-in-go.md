# 创建自定义错误

### 导言

Go 在标准库中提供了两种创建错误的方法，即[`errors.New`和`fmt.Errorf`](https://www.digitalocean.com/community/tutorials/handling-errors-in-go#creating-errors)。在向用户或未来的自己调试时传达更复杂的错误信息时，有时这两种机制不足以充分捕获和报告所发生的错误。为了传达更复杂的错误信息并获得更多功能，我们可以实现标准库接口类型[`error`](https://golang.org/pkg/builtin/#error).New 和 errors.Errorf。

语法如下

```go
type error interface {
  Error() string
}
```

[`内置`](https://golang.org/pkg/builtin/)软件包将`error`定义为一个接口，其中包含一个以字符串形式返回错误信息的`Error()`方法。通过实现该方法，我们可以将定义的任何类型转换为我们自己的错误。

让我们试着运行下面的示例，看看`错误`接口的实现：

```go
package main

import (
 "fmt"
 "os"
)

type MyError struct{}

func (m *MyError) Error() string {
 return "boom"
}

func sayHello() (string, error) {
 return "", &MyError{}
}

func main() {
 s, err := sayHello()
 if err != nil {
  fmt.Println("unexpected error: err:", err)
  os.Exit(1)
 }
 fmt.Println("The string:", s)
}
```

我们将看到以下输出：

```
Output
unexpected error: err: boom
exit status 1
```

在这里，我们创建了一个新的空结构类型`MyError`，并定义了`Error()`方法。`Error()`方法返回字符串`"boom"`。

在`main()` 中，我们调用了函数`sayHello`，它返回一个空字符串和一个新的`MyError` 实例。由于`sayHello`将始终返回错误，因此`main()`中 if 语句正文中的`fmt.Println`调用将始终执行。然后，我们使用`fmt.Println`打印短前缀字符串`"unexpected error:"（意外错误：`）以及在`err`变量中保存的`MyError`实例。

请注意，我们不必直接调用`Error()`，因为`fmt`软件包能够自动检测到这是`错误`的实现。它会以[透明方式](https://en.wikipedia.org/wiki/Transparency_(human–computer_interaction))调用`Error()`来获取字符串`"` `boom` `"`，并将其与前缀字符串`"un unexpected error: err: "`连接起来。

## [在自定义错误中收集详细信息](https://www.digitalocean.com/community/tutorials/creating-custom-errors-in-go#collecting-detailed-information-in-a-custom-error)

有时，自定义错误是捕获详细错误信息的最简洁方法。例如，假设我们想捕获 HTTP 请求产生的错误状态代码；运行以下程序可查看`error`的实现，它允许我们干净利落地捕获该信息：

```go
package main

import (
 "errors"
 "fmt"
 "os"
)

type RequestError struct {
 StatusCode int

 Err error
}

func (r *RequestError) Error() string {
 return fmt.Sprintf("status %d: err %v", r.StatusCode, r.Err)
}

func doRequest() error {
 return &RequestError{
  StatusCode: 503,
  Err:        errors.New("unavailable"),
 }
}

func main() {
 err := doRequest()
 if err != nil {
  fmt.Println(err)
  os.Exit(1)
 }
 fmt.Println("success!")
}
```

我们将看到以下输出：

```
输出状态 503：错误不可用 退出状态 1
```

在本例中，我们创建了`RequestError`的新实例，并使用标准库中的`errors.New`函数提供了状态代码和错误信息。然后，我们使用`fmt.Println`将其打印出来，就像前面的示例一样。

在`RequestError` 的`Error()`方法中，我们使用`fmt.Sprintf`函数使用创建错误时提供的信息构建一个字符串。

## [类型断言和自定义错误](https://www.digitalocean.com/community/tutorials/creating-custom-errors-in-go#type-assertions-and-custom-errors)

`error`接口只公开了一个方法，但我们可能需要访问`error`实现的其他方法来正确处理错误。例如，我们可能有几个自定义的`error`实现，它们都是临时的，可以重试--通过`Temporary()`方法的存在来表示。

接口提供了一个狭窄的视图，让我们可以看到类型所提供的更广泛的方法集，因此我们必须使用*类型断言*来更改该视图所显示的方法，或将其完全删除。

下面的示例增强了之前显示的`RequestError`，增加了一个`Temporary()`方法，该方法将指示调用者是否应该重试请求：

```go
package main

import (
 "errors"
 "fmt"
 "net/http"
 "os"
)

type RequestError struct {
 StatusCode int

 Err error
}

func (r *RequestError) Error() string {
 return r.Err.Error()
}

func (r *RequestError) Temporary() bool {
 return r.StatusCode == http.StatusServiceUnavailable // 503
}

func doRequest() error {
 return &RequestError{
  StatusCode: 503,
  Err:        errors.New("unavailable"),
 }
}

func main() {
 err := doRequest()
 if err != nil {
  fmt.Println(err)
  re, ok := err.(*RequestError)
  if ok {
   if re.Temporary() {
    fmt.Println("This request can be tried again")
   } else {
    fmt.Println("This request cannot be tried again")
   }
  }
  os.Exit(1)
 }

 fmt.Println("success!")
}
```

我们将看到以下输出：

```
输出不可用 此请求可重试 退出状态 1
```

在`main()` 中，我们调用`doRequest`(`)`，它会向我们返回一个`错误`接口。我们首先打印`Error()`方法返回的错误信息。接下来，我们使用类型断言`re, ok := err.(*RequestError)`，尝试公开`RequestError`的所有方法。如果类型断言成功，我们就使用`Temporary()`方法查看此错误是否为临时错误。由于`doRequest()`设置的`StatusCode`是`503`，符合`http.StatusServiceUnavailable`，因此返回`true`并打印出`"此请求可以重试"`。在实际操作中，我们会发出另一个请求，而不是打印一条消息。

## [包装错误](https://www.digitalocean.com/community/tutorials/creating-custom-errors-in-go#wrapping-errors)

常见的错误来自程序外部，如数据库、网络连接等。这些错误提供的错误信息无法帮助任何人找到错误的根源。在错误信息的开头用额外的信息对错误进行包装，可以为成功调试提供一些所需的上下文。

下面的示例演示了我们如何将一些上下文信息附加到从其他函数返回的隐含`错误`中：

```go
package main

import (
 "errors"
 "fmt"
)

type WrappedError struct {
 Context string
 Err     error
}

func (w *WrappedError) Error() string {
 return fmt.Sprintf("%s: %v", w.Context, w.Err)
}

func Wrap(err error, info string) *WrappedError {
 return &WrappedError{
  Context: info,
  Err:     err,
 }
}

func main() {
 err := errors.New("boom!")
 err = Wrap(err, "main")

 fmt.Println(err)
}
```

我们将看到以下输出：

```
输出main: boom！
```

`WrappedError`是一个包含两个`字段`的结构体：一个是`字符串`形式的上下文信息，另一个是该`WrappedError`提供更多信息的`错误`。当调用`Error()`方法时，我们再次使用`fmt.Sprintf`打印上下文信息，然后打印`错误``（fmt`.Sprintf 也知道隐式调用`Error()`方法）。

在`main()` 中，我们使用`errors.New` 创建一个错误，然后使用我们定义的`Wrap`函数对该错误进行包装。这样，我们就可以指出该`错误`是在`"main "`中生成的。此外，由于我们的`WrappedError`也是一个`错误`，因此我们可以封装其他的`WrappedError`，`这样`我们就可以看到一个链，帮助我们追踪错误的源头。在标准库的帮助下，我们甚至可以在错误中嵌入完整的堆栈跟踪。

## [结论](https://www.digitalocean.com/community/tutorials/creating-custom-errors-in-go#conclusion)

由于`错误`接口只是一个单一的方法，因此我们看到，我们可以非常灵活地为不同情况提供不同类型的错误。这可以包括从将多个信息作为错误的一部分进行通信，一直到实现[指数回退](https://en.wikipedia.org/wiki/Exponential_backoff)等所有内容。虽然 Go 中的错误处理机制表面上看似简单，但我们可以使用这些自定义错误来处理常见和不常见的情况，从而实现相当丰富的处理功能。

Go 还有另一种交流意外行为的机制，即恐慌。在错误处理系列的下一篇文章中，我们将探讨恐慌是什么以及如何处理它们。
