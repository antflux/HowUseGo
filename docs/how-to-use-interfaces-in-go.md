# 如何使用接口

### 介绍

编写灵活、可重用和模块化的代码对于开发通用程序至关重要。以这种方式工作可以避免在多个地方进行相同的更改，从而确保代码更容易维护。如何做到这一点因语言而异。例如，[继承](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming))是Java、C++、C#等语言中使用的一种常见方法。

开发人员也可以通过[组合来](https://en.wikipedia.org/wiki/Object_composition)实现相同的设计目标。组合是一种将对象或数据类型组合成更复杂的对象或数据类型的方法。这是Go用来促进代码重用、模块化和灵活性的方法。Go中的接口提供了一种组织复杂组合的方法，学习如何使用它们将允许您创建通用的，可重用的代码。

在本文中，我们将学习如何组合具有常见行为的自定义类型，这将允许我们重用代码。我们还将学习如何为我们自己的自定义类型实现接口，这些自定义类型将满足从另一个包定义的接口。

## 定义一种行为

组合的核心实现之一是接口的使用。接口定义了类型的行为。Go标准库中最常用的接口之一是[`fmt.Stringer`](https://golang.org/pkg/fmt/#Stringer)接口：

```go
type Stringer interface {
    String() string
}
```

第一行代码定义了一个名为`type`的`Stringer`。他说，这是一个#3。就像定义一个结构体一样，Go使用花括号（`interface`）来包围接口的定义。与定义结构相比，我们只定义接口的行为;也就是说，“这个类型可以做什么”。

在`Stringer`接口的情况下，唯一的行为是`String()`方法。该方法不接受参数并返回一个字符串。

接下来，让我们看看一些具有`fmt.Stringer`行为的代码：

main.go

```go
package main

import "fmt"

type Article struct {
 Title string
 Author string
}

func (a Article) String() string {
 return fmt.Sprintf("The %q article was written by %s.", a.Title, a.Author)
}

func main() {
 a := Article{
  Title: "Understanding Interfaces in Go",
  Author: "Sammy Shark",
 }
 fmt.Println(a.String())
}
```

我们要做的第一件事是创建一个名为`Article`的新类型。此类型有一个`Title`和一个`Author`字段，并且都是字符串[数据类型]：

main.go

```go
...
type Article struct {
 Title string
 Author string
}
...
```

接下来，我们在`method`类型上定义一个名为`String`的[`Article`]。`String`方法将返回一个表示`Article`类型的字符串：

main.go

```go
...
func (a Article) String() string {
 return fmt.Sprintf("The %q article was written by %s.", a.Title, a.Author)
}
...
```

然后，在我们的`main`[函数]中，我们创建了一个`Article`类型的实例，并将其分配给名为`a`的[变量]。我们为`"Understanding Interfaces in Go"`字段提供`Title`的值，为`"Sammy Shark"`字段提供`Author`的值：

main.go

```go
...
a := Article{
 Title: "Understanding Interfaces in Go",
 Author: "Sammy Shark",
}
...
```

然后，我们通过调用`String`并传入`fmt.Println`方法调用的结果来打印出`a.String()`方法的结果：

main.go

```go
...
fmt.Println(a.String())
```

运行程序后，您将看到以下输出：

```
Output
The "Understanding Interfaces in Go" article was written by Sammy Shark.
```

到目前为止，我们还没有使用接口，但我们确实创建了一个具有行为的类型。该行为与`fmt.Stringer`接口匹配。接下来，让我们看看如何使用该行为来使代码更具可重用性。

## 限定界面

现在我们已经用所需的行为定义了类型，我们可以看看如何使用该行为。

然而，在我们这样做之前，让我们看看如果我们想从函数中的`String`类型调用`Article`方法，我们需要做什么：

main.go

```
package main

import "fmt"

type Article struct {
 Title string
 Author string
}

func (a Article) String() string {
 return fmt.Sprintf("The %q article was written by %s.", a.Title, a.Author)
}

func main() {
 a := Article{
  Title: "Understanding Interfaces in Go",
  Author: "Sammy Shark",
 }
 Print(a)
}

func Print(a Article) {
 fmt.Println(a.String())
}
```

在这段代码中，我们添加了一个名为`Print`的新函数，它接受一个`Article`作为参数。请注意，`Print`函数所做的唯一事情就是调用`String`方法。因此，我们可以定义一个接口来传递给函数：

main.go

```
package main

import "fmt"

type Article struct {
 Title string
 Author string
}

func (a Article) String() string {
 return fmt.Sprintf("The %q article was written by %s.", a.Title, a.Author)
}

type Stringer interface {
 String() string
}

func main() {
 a := Article{
  Title: "Understanding Interfaces in Go",
  Author: "Sammy Shark",
 }
 Print(a)
}

func Print(s Stringer) {
 fmt.Println(s.String())
}
```

在这里，我们创建了一个名为`Stringer`的接口：

main.go

```go
...
type Stringer interface {
 String() string
}
...
```

`Stringer`接口只有一个方法，名为`String()`，返回`string`。[方法]是一个特殊的函数，它的作用域是Go语言中的特定类型。与函数不同，方法只能从定义它的类型的实例中调用。

然后，我们更新了`Print`方法的签名，使其接受一个`Stringer`，而不是一个具体的类型`Article`。因为编译器知道`Stringer`接口定义了`String`方法，所以它只接受同时具有`String`方法的类型。

现在我们可以对任何满足`Print`接口的东西使用`Stringer`方法。让我们创建另一个类型来演示这一点：

main.go

```
package main

import "fmt"

type Article struct {
 Title  string
 Author string
}

func (a Article) String() string {
 return fmt.Sprintf("The %q article was written by %s.", a.Title, a.Author)
}

type Book struct {
 Title  string
 Author string
 Pages  int
}

func (b Book) String() string {
 return fmt.Sprintf("The %q book was written by %s.", b.Title, b.Author)
}

type Stringer interface {
 String() string
}

func main() {
 a := Article{
  Title:  "Understanding Interfaces in Go",
  Author: "Sammy Shark",
 }
 Print(a)

 b := Book{
  Title:  "All About Go",
  Author: "Jenny Dolphin",
  Pages:  25,
 }
 Print(b)
}

func Print(s Stringer) {
 fmt.Println(s.String())
}
```

我们现在添加第二种类型，称为`Book`。它还定义了`String`方法。这意味着它也满足`Stringer`接口。因此，我们也可以将其发送到我们的`Print`函数：

```
Output
The "Understanding Interfaces in Go" article was written by Sammy Shark.
The "All About Go" book was written by Jenny Dolphin. It has 25 pages.
```

到目前为止，我们已经演示了如何只使用一个接口。但是，一个接口可以定义多个行为。接下来，我们将看到如何通过声明更多的方法来使接口更加通用。

## 界面中的多个行为

编写Go代码的核心租户之一是编写小的，简洁的类型，并将它们组合成更大，更复杂的类型。在编写接口时也是如此。为了了解如何构建接口，我们首先只定义一个接口。我们将定义两个形状，`Circle`和`Square`，它们都将定义一个名为`Area`的方法。此方法将返回它们各自形状的几何面积：

main.go

```go
package main

import (
 "fmt"
 "math"
)

type Circle struct {
 Radius float64
}

func (c Circle) Area() float64 {
 return math.Pi * math.Pow(c.Radius, 2)
}

type Square struct {
 Width  float64
 Height float64
}

func (s Square) Area() float64 {
 return s.Width * s.Height
}

type Sizer interface {
 Area() float64
}

func main() {
 c := Circle{Radius: 10}
 s := Square{Height: 10, Width: 5}

 l := Less(c, s)
 fmt.Printf("%+v is the smallest\n", l)
}

func Less(s1, s2 Sizer) Sizer {
 if s1.Area() < s2.Area() {
  return s1
 }
 return s2
}
```

因为每个类型都声明了`Area`方法，所以我们可以创建一个定义该行为的接口。我们创建以下`Sizer`接口：

main.go

```go
...
type Sizer interface {
 Area() float64
}
...
```

然后我们定义一个名为`Less`的函数，它接受两个`Sizer`并返回最小的一个：

main.go

```go
...
func Less(s1, s2 Sizer) Sizer {
 if s1.Area() < s2.Area() {
  return s1
 }
 return s2
}
...
```

请注意，我们不仅接受两个参数作为类型`Sizer`，而且还返回结果作为`Sizer`。这意味着我们不再返回`Square`或`Circle`，而是返回`Sizer`的接口。

最后，我们打印出面积最小的部分：

```
Output
{Width:5 Height:10} is the smallest
```

接下来，让我们为每个类型添加另一个行为。这一次我们将添加返回字符串的`String()`方法。这将满足`fmt.Stringer`接口：

main.go

```
package main

import (
 "fmt"
 "math"
)

type Circle struct {
 Radius float64
}

func (c Circle) Area() float64 {
 return math.Pi * math.Pow(c.Radius, 2)
}

func (c Circle) String() string {
 return fmt.Sprintf("Circle {Radius: %.2f}", c.Radius)
}

type Square struct {
 Width  float64
 Height float64
}

func (s Square) Area() float64 {
 return s.Width * s.Height
}

func (s Square) String() string {
 return fmt.Sprintf("Square {Width: %.2f, Height: %.2f}", s.Width, s.Height)
}

type Sizer interface {
 Area() float64
}

type Shaper interface {
 Sizer
 fmt.Stringer
}

func main() {
 c := Circle{Radius: 10}
 PrintArea(c)

 s := Square{Height: 10, Width: 5}
 PrintArea(s)

 l := Less(c, s)
 fmt.Printf("%v is the smallest\n", l)

}

func Less(s1, s2 Sizer) Sizer {
 if s1.Area() < s2.Area() {
  return s1
 }
 return s2
}

func PrintArea(s Shaper) {
 fmt.Printf("area of %s is %.2f\n", s.String(), s.Area())
}
```

因为`Circle`和`Square`类型都实现了`Area`和`String`方法，我们现在可以创建另一个接口来描述更广泛的行为集。为此，我们将创建一个名为`Shaper`的接口。我们将`Sizer`接口和`fmt.Stringer`接口组成如下：

main.go

```go
...
type Shaper interface {
 Sizer
 fmt.Stringer
}
...
```

注意事项：尝试以`er`结尾来命名接口被认为是惯用的，例如`fmt.Stringer`，`io.Writer`等。这就是为什么我们将接口命名为`Shaper`，而不是`Shape`。

现在我们可以创建一个名为`PrintArea`的函数，它将`Shaper`作为参数。这意味着我们可以在传入的值上调用`Area`和`String`方法：

main.go

```go
...
func PrintArea(s Shaper) {
 fmt.Printf("area of %s is %.2f\n", s.String(), s.Area())
}
```

如果我们运行程序，我们将收到以下输出：

```
Output
area of Circle {Radius: 10.00} is 314.16
area of Square {Width: 5.00, Height: 10.00} is 50.00
Square {Width: 5.00, Height: 10.00} is the smallest
```

我们现在已经看到了如何创建较小的接口，并根据需要将它们构建成较大的接口。虽然我们可以从较大的接口开始，并将其传递给所有函数，但最佳做法是只将最小的接口发送给所需的函数。 这通常会产生更清晰的代码，因为任何接受特定较小接口的东西都只打算使用该定义的行为。

例如，如果我们将`Shaper`传递给`Less`函数，我们可以假设它将同时调用`Area`和`String`方法。 然而，由于我们只打算调用`Area`方法，它使`Less`函数变得清晰，因为我们知道我们只能调用传递给它的任何参数的`Area`方法。

## 结论

我们已经看到了如何创建较小的接口并将其构建为较大的接口，从而允许我们仅共享函数或方法所需的内容。我们还了解到，我们可以从其他接口组合我们的接口，包括从其他包定义的接口，而不仅仅是我们的包。
