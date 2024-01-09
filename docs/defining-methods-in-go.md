# 在 Go 中定义方法

## 介绍

[函数]允许您将逻辑组织成可重复的过程，这些过程在每次运行时可以使用不同的参数。在定义函数的过程中，您经常会发现多个函数可能每次都对同一段数据进行操作。Go语言可以识别这种模式，并允许你定义特殊的函数，称为方法，其目的是对某种特定类型的实例进行操作，称为接收器。将方法添加到类型中不仅可以传递数据是什么，还可以传递如何使用该数据。

## 定义方法

定义方法的语法类似于定义函数的语法。唯一的区别是在`func`关键字之后添加了一个额外的参数，用于指定方法的接收者。receiver是要在其上定义方法的类型的声明。下面的示例在结构类型上定义方法：

```go
package main

import "fmt"

type Creature struct {
 Name     string
 Greeting string
}

func (c Creature) Greet() {
 fmt.Printf("%s says %s", c.Name, c.Greeting)
}

func main() {
 sammy := Creature{
  Name:     "Sammy",
  Greeting: "Hello!",
 }
 Creature.Greet(sammy)
}
```

如果你运行这段代码，输出将是：

```
Output
Sammy says Hello!
```

我们创建了一个名为`Creature`的结构体，其中有`string`字段用于`Name`和`Greeting`。这个`Creature`定义了一个方法，`Greet`。在receiver声明中，我们将`Creature`的实例赋给了变量`c`，这样当我们在`Creature`中组装问候消息时，就可以引用`fmt.Printf`的字段。

在其他语言中，方法调用的接收者通常由关键字（例如`this`或`self`）引用。Go将接收器视为一个变量，所以你可以随意命名它。社区对此参数的首选样式是接收器类型第一个字符的小写版本。在这个例子中，我们使用`c`，因为接收器类型是`Creature`。

在`main`的主体中，我们创建了`Creature`的实例，并为其`Name`和`Greeting`字段指定了值。我们在这里调用了`Greet`方法，方法是用`.`连接类型名和方法名，并提供`Creature`的实例作为第一个参数。

Go提供了另一种更方便的方法来调用结构体实例上的方法，如本例所示：

```
package main

import "fmt"

type Creature struct {
 Name     string
 Greeting string
}

func (c Creature) Greet() {
 fmt.Printf("%s says %s", c.Name, c.Greeting)
}

func main() {
 sammy := Creature{
  Name:     "Sammy",
  Greeting: "Hello!",
 }
 sammy.Greet()
}
```

如果你运行这个，输出将与前面的例子相同：

```
Output
Sammy says Hello!
```

这个例子和上一个例子一样，但是这次我们使用点表示法来调用`Greet`方法，使用存储在`Creature`变量中的`sammy`作为接收器。这是第一个例子中函数调用的一种简写表示法。标准库和Go社区非常喜欢这种风格，以至于你很少会看到前面展示的函数调用风格。

下一个例子显示了为什么点表示法更流行的一个原因：

```
package main

import "fmt"

type Creature struct {
 Name     string
 Greeting string
}

func (c Creature) Greet() Creature {
 fmt.Printf("%s says %s!\n", c.Name, c.Greeting)
 return c
}

func (c Creature) SayGoodbye(name string) {
 fmt.Println("Farewell", name, "!")
}

func main() {
 sammy := Creature{
  Name:     "Sammy",
  Greeting: "Hello!",
 }
 sammy.Greet().SayGoodbye("gophers")

 Creature.SayGoodbye(Creature.Greet(sammy), "gophers")
}
```

如果你运行这段代码，输出看起来像这样：

```
Output
Sammy says Hello!!
Farewell gophers !
Sammy says Hello!!
Farewell gophers !
```

我们修改了前面的示例，引入了另一个名为`SayGoodbye`的方法，并将`Greet`更改为返回`Creature`，以便我们可以在该实例上调用更多方法。在`main`的主体中，我们首先使用点表示法调用`Greet`变量上的方法`SayGoodbye`和`sammy`，然后使用函数调用风格。

这两种样式都输出相同的结果，但是使用点表示法的示例可读性更强。点链还告诉我们方法将被调用的顺序，函数式风格颠倒了这个顺序。在`SayGoodbye`调用中添加一个参数进一步模糊了方法调用的顺序。点表示法的清晰性是它是Go中调用方法的首选风格的原因，无论是在标准库中还是在整个Go生态系统中的第三方包中。

在类型上定义方法，而不是定义对某个值进行操作的函数，对Go编程语言有其他特殊意义。方法是接口背后的核心概念。

## 接口

当你在Go语言中为任何类型定义一个方法时，这个方法会被添加到该类型的方法集中。方法集是与该类型相关联的函数的集合，作为方法，并由Go编译器用于确定是否可以将某个类型分配给具有接口类型的变量。接口类型是编译器使用的方法的规范，以保证类型为这些方法提供实现。任何类型的方法与接口定义中的方法具有相同的名称，相同的参数和相同的返回值，都被称为实现了该接口，并被允许分配给具有该接口类型的变量。以下是标准库中`fmt.Stringer`接口的定义：

```go
type Stringer interface {
  String() string
}
```

对于实现`fmt.Stringer`接口的类型，它需要提供一个返回`String()`的`string`方法。实现这个接口可以让你的类型在你传递类型的实例给`fmt`包中定义的函数时，完全按照你的意愿打印出来（有时也被称为“漂亮打印”）。下面的示例定义实现此接口的类型：

```go
package main

import (
 "fmt"
 "strings"
)

type Ocean struct {
 Creatures []string
}

func (o Ocean) String() string {
 return strings.Join(o.Creatures, ", ")
}

func log(header string, s fmt.Stringer) {
 fmt.Println(header, ":", s)
}

func main() {
 o := Ocean{
  Creatures: []string{
   "sea urchin",
   "lobster",
   "shark",
  },
 }
 log("ocean contains", o)
}
```

当你运行代码时，你会看到这样的输出：

```
Output
ocean contains : sea urchin, lobster, shark
```

这个例子定义了一个名为`Ocean`的新结构类型。`Ocean`被称为实现了`fmt.Stringer`接口，因为`Ocean`定义了一个名为`String`的方法，该方法不带参数并返回`string`。在`main`中，我们定义了一个新的`Ocean`，并将其传递给一个`log`函数，该函数首先输出一个`string`，然后输出实现`fmt.Stringer`的任何内容。Go编译器允许我们在这里传递`o`，因为`Ocean`实现了`fmt.Stringer`请求的所有方法。在`log`中，我们使用`fmt.Println`，当它遇到`String`作为其参数之一时，它会调用`Ocean`的`fmt.Stringer`方法。

如果`Ocean`没有提供`String()`方法，Go会产生编译错误，因为`log`方法请求一个`fmt.Stringer`作为它的参数。错误看起来像这样：

```
Output
src/e4/main.go:24:6: cannot use o (type Ocean) as type fmt.Stringer in argument to log:
        Ocean does not implement fmt.Stringer (missing String method)
```

Go还将确保提供的`String()`方法与`fmt.Stringer`接口请求的方法完全匹配。如果不这样做，它将产生一个错误，看起来像这样：

```
Output
src/e4/main.go:26:6: cannot use o (type Ocean) as type fmt.Stringer in argument to log:
        Ocean does not implement fmt.Stringer (wrong type for String method)
                have String()
                want String() string
```

在到目前为止的例子中，我们已经在值接收器上定义了方法。也就是说，如果我们使用方法的函数调用，第一个参数，引用方法定义的类型，将是该类型的值，而不是[指针]。因此，我们对提供给该方法的实例所做的任何修改都将在该方法完成执行时被丢弃，因为接收到的值是数据的副本。也可以在指向类型的指针接收器上定义方法。

## 指针类型的接收

在指针接收器上定义方法的语法与在值接收器上定义方法的语法几乎相同。不同之处在于接收器声明中的类型名称前面加了一个星号（`*`）。下面的示例在指向类型的指针接收器上定义一个方法：

```go
package main

import "fmt"

type Boat struct {
 Name string

 occupants []string
}

func (b *Boat) AddOccupant(name string) *Boat {
 b.occupants = append(b.occupants, name)
 return b
}

func (b Boat) Manifest() {
 fmt.Println("The", b.Name, "has the following occupants:")
 for _, n := range b.occupants {
  fmt.Println("\t", n)
 }
}

func main() {
 b := &Boat{
  Name: "S.S. DigitalOcean",
 }

 b.AddOccupant("Sammy the Shark")
 b.AddOccupant("Larry the Lobster")

 b.Manifest()
}
```

运行此示例时，您将看到以下输出：

```
Output
The S.S. DigitalOcean has the following occupants:
  Sammy the Shark
  Larry the Lobster
```

这个例子定义了一个带有`Boat`和`Name`的`occupants`类型。我们希望强制其他包中的代码只使用`AddOccupant`方法添加占用者，因此我们通过将字段名称的第一个字母小写来使`occupants`字段不可导出。我们还想确保调用`AddOccupant`会导致`Boat`的实例被修改，这就是为什么我们在指针接收器上定义了`AddOccupant`。指针用作对某个类型的特定实例的引用，而不是该类型的副本。知道将使用指向`AddOccupant`的指针调用`Boat`可以保证任何修改都将持久化。

在`main`中，我们定义了一个新变量`b`，它将保存一个指向`Boat`（`*Boat`）的指针。我们在这个实例上调用了两次`AddOccupant`方法来添加两个乘客。`Manifest`方法是在`Boat`值上定义的，因为在它的定义中，接收器被指定为`(b Boat)`。在`main`中，我们仍然可以调用`Manifest`，因为Go能够自动解引用指针以获得`Boat`值。`b.Manifest()`相当于`(*b).Manifest()`。

当试图将值赋给接口类型的变量时，方法是定义在指针接收器上还是值接收器上具有重要意义。

## 指针接收器和接口

当你给一个具有接口类型的变量赋值时，Go编译器会检查被赋值的类型的方法集，以确保它具有接口期望的方法。指针接收器和值接收器的方法集是不同的，因为接收指针的方法可以修改它们的接收器，而接收值的方法不能。

下面的示例演示如何定义两个方法：一个在类型的指针接收器上，另一个在其值接收器上。但是，只有指针接收器才能满足本例中定义的接口：

```go
package main

import "fmt"

type Submersible interface {
 Dive()
}

type Shark struct {
 Name string

 isUnderwater bool
}

func (s Shark) String() string {
 if s.isUnderwater {
  return fmt.Sprintf("%s is underwater", s.Name)
 }
 return fmt.Sprintf("%s is on the surface", s.Name)
}

func (s *Shark) Dive() {
 s.isUnderwater = true
}

func submerge(s Submersible) {
 s.Dive()
}

func main() {
 s := &Shark{
  Name: "Sammy",
 }

 fmt.Println(s)

 submerge(s)

 fmt.Println(s)
}
```

当你运行代码时，你会看到这样的输出：

```
Output
Sammy is on the surface
Sammy is underwater
```

这个例子定义了一个名为`Submersible`的接口，该接口需要具有`Dive()`方法的类型。然后我们定义了一个带有`Shark`字段的`Name`类型和一个`isUnderwater`方法来跟踪`Shark`的状态。我们在指针接收器上定义了一个`Dive()`方法，它将`Shark`修改为`isUnderwater`。我们还定义了值接收器的`true`方法，这样它就可以通过使用我们之前看到的`String()`接受的`Shark`接口，使用`fmt.Println`干净地打印`fmt.Stringer`的状态。我们还使用了一个带`fmt.Println`参数的函数`submerge`。

使用`Submersible`接口而不是`*Shark`允许`submerge`函数仅依赖于类型提供的行为。这使得`submerge`函数更加可重用，因为您不必为`submerge`、`Submarine`或任何其他我们尚未想到的未来水生生物编写新的`Whale`函数。只要它们定义了一个`Dive()`方法，它们就可以与`submerge`函数一起使用。

在`main`中，我们定义了一个变量`s`，它是一个指向`Shark`的指针，并立即用`s`打印了`fmt.Println`。这显示了输出的第一部分`Sammy is on the surface`。我们将`s`传递给`submerge`，然后再次调用`fmt.Println`，并将`s`作为其参数，以查看输出的第二部分`Sammy is underwater`。

如果我们将`s`改为`Shark`而不是`*Shark`，Go编译器将产生错误：

```
Output
cannot use s (type Shark) as type Submersible in argument to submerge:
 Shark does not implement Submersible (Dive method has pointer receiver)
```

Go编译器告诉我们`Shark`确实有一个`Dive`方法，它只是在指针接收器上定义的。当您在自己的代码中看到此消息时，解决方法是在分配值类型的变量之前使用`&`操作符传递指向接口类型的指针。

## 结论

在Go中声明方法最终与定义接收不同类型变量的函数没有什么不同。[使用指针]的规则相同。Go语言为这个极其常见的函数定义提供了一些便利，并将这些便利收集到可以通过接口类型进行推理的方法集合中。有效地使用方法将允许您在代码中使用接口来提高可测试性，并为将来的代码读者留下更好的组织。
