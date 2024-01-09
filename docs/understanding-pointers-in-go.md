# 了解 Go 中的指针

## 介绍

当你在Go中编写软件时，你将编写函数和方法。将数据作为参数传递给这些函数。有时，函数需要数据的本地副本，而您希望原始数据保持不变。例如，如果您是一家银行，并且您有一个函数可以根据用户选择的储蓄计划向用户显示其余额的变化，那么您不希望在客户选择计划之前更改客户的实际余额;您只希望在计算中使用它。这被称为按值传递，因为你将变量的值发送给函数，而不是变量本身。

其他时候，您可能希望函数能够更改原始变量中的数据。例如，当银行客户向其帐户存款存款时，您希望存款函数能够访问实际余额，而不是副本。在这种情况下，您不需要将实际数据发送给函数;您只需要告诉函数数据在内存中的位置。一种称为指针的数据类型保存数据的内存地址，但不保存数据本身。内存地址告诉函数在哪里找到数据，但不是数据的值。您可以将指针传递给函数，而不是传递数据，然后函数可以在原地更改原始变量。这被称为通过引用传递，因为变量的值没有传递给函数，只是传递给它的位置。

在本文中，您将创建并使用指针来共享变量对内存空间的访问。

## 定义和使用指针

当你使用一个指向变量的指针时，有几个不同的语法元素你需要理解。 第一个是使用&符号（`&`）。 如果你在变量名前放置一个&号，你是在声明你想要获取地址，或者指向该变量的指针。第二个语法元素是星号（`*`）或解引用运算符的使用。当你声明一个指针变量时，你在变量名后面加上指针指向的变量的类型，前缀是`*`，就像这样：

```go
var myPointer *int32 = &someint
```

这将创建`myPointer`作为指向`int32`变量的指针，并使用`someint`的地址替换指针。指针实际上并不包含`int32`，只是一个地址。

让我们来看看一个指向`string`的指针。 下面的代码声明了一个字符串的值和一个指向字符串的指针：

main.go

```go
package main

import "fmt"

func main() {
 var creature string = "shark"
 var pointer *string = &creature

 fmt.Println("creature =", creature)
 fmt.Println("pointer =", pointer)
}
```

使用以下命令运行程序：

```bash
go run main.go
```

当你运行程序时，它会打印出变量的值，以及变量的存储地址（指针地址）。内存地址是一个十六进制数，并不意味着是人类可读的。在实践中，你可能永远不会输出一个内存地址来查看它。因为每个程序在运行时都是在自己的内存空间中创建的，所以每次运行时指针的值都会不同，并且与这里显示的输出不同：

```
Output
creature = shark
pointer = 0xc0000721e0
```

我们定义的第一个变量命名为`creature`，并将其设置为值为`string`的`shark`。 然后我们创建了另一个名为`pointer`的变量。 这一次，我们将`pointer`变量的值设置为`creature`变量的地址。我们通过使用&符号（`&`）将值的地址存储在变量中。这意味着`pointer`变量存储的是`creature`变量的地址，而不是实际值。

这就是为什么当我们打印出`pointer`的值时，我们收到了`0xc0000721e0`的值，这是`creature`变量当前存储在计算机内存中的地址。

如果你想打印出从`pointer`变量指向的变量的值，你需要解引用该变量。下面的代码使用`*`运算符来解引用`pointer`变量并检索其值：

main.go

```go
package main

import "fmt"

func main() {
 var creature string = "shark"
 var pointer *string = &creature

 fmt.Println("creature =", creature)
 fmt.Println("pointer =", pointer)

 fmt.Println("*pointer =", *pointer)
}
```

如果运行此代码，您将看到以下输出：

```
Output
creature = shark
pointer = 0xc000010200
*pointer = shark
```

我们添加的最后一行现在取消引用`pointer`变量，并打印出存储在该地址的值。

如果你想修改存储在`pointer`变量位置的值，你也可以使用解引用操作符：

main.go

```go
package main

import "fmt"

func main() {
 var creature string = "shark"
 var pointer *string = &creature

 fmt.Println("creature =", creature)
 fmt.Println("pointer =", pointer)

 fmt.Println("*pointer =", *pointer)

 *pointer = "jellyfish"
 fmt.Println("*pointer =", *pointer)
}
```

运行以下代码以查看输出：

```
Output
creature = shark
pointer = 0xc000094040
*pointer = shark
*pointer = jellyfish
```

我们通过在变量名前面使用星号（`pointer`）来设置`*`变量引用的值，然后提供一个新值`jellyfish`。 正如你所看到的，当我们打印解引用的值时，它现在被设置为`jellyfish`。

你可能没有意识到，但我们实际上也改变了`creature`变量的值。 这是因为`pointer`变量实际上指向`creature`变量的地址。这意味着，如果我们改变`pointer`变量指向的值，我们也会改变`creature`变量的值。

main.go

```go
package main

import "fmt"

func main() {
 var creature string = "shark"
 var pointer *string = &creature

 fmt.Println("creature =", creature)
 fmt.Println("pointer =", pointer)

 fmt.Println("*pointer =", *pointer)

 *pointer = "jellyfish"
 fmt.Println("*pointer =", *pointer)

 fmt.Println("creature =", creature)
}
```

输出如下所示：

```
Output
creature = shark
pointer = 0xc000010200
*pointer = shark
*pointer = jellyfish
creature = jellyfish
```

虽然这段代码演示了指针的工作方式，但这并不是Go语言中使用指针的典型方式。更常见的是在定义函数参数和返回值时使用它们，或者在定义自定义类型的方法时使用它们。 让我们看看如何将指针与函数一起使用来共享对变量的访问。

同样，请记住，我们打印`pointer`的值是为了说明它是一个指针。 实际上，除了引用底层值来检索或更新该值之外，您不会使用指针的值。

## 函数指针接收器

当你写一个函数时，你可以定义参数通过值传递，或者通过引用传递。 按值传递意味着该值的副本被发送到函数，并且该函数内对该参数的任何更改都只影响该函数内的该变量，而不是传递它的位置。但是，如果你通过引用传递，意味着你传递一个指向该参数的指针，你可以在函数内部更改值，也可以更改传入的原始变量的值。你可以阅读更多关于如何[在Go中定义和调用函数]的内容。

决定什么时候传递指针，而不是什么时候发送一个值，就是要知道你是否希望值改变。 如果你不想改变这个值，就把它作为一个值发送。 如果你希望你传递给变量的函数能够改变它，那么你应该把它作为指针传递。

为了了解其中的区别，让我们先来看看一个通过`value`传入参数的函数：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func main() {
 var creature Creature = Creature{Species: "shark"}

 fmt.Printf("1) %+v\n", creature)
 changeCreature(creature)
 fmt.Printf("3) %+v\n", creature)
}

func changeCreature(creature Creature) {
 creature.Species = "jellyfish"
 fmt.Printf("2) %+v\n", creature)
}
```

输出如下所示：

```
Output
1) {Species:shark}
2) {Species:jellyfish}
3) {Species:shark}
```

首先，我们创建了一个名为`Creature`的自定义类型。 它有一个名为`Species`的字段，这是一个字符串。 在`main`函数中，我们创建了一个名为`creature`的新类型的实例，并将`Species`字段设置为`shark`。 然后，我们打印出变量，以显示存储在`creature`变量中的当前值。

接下来，我们调用`changeCreature`并传入`creature`变量的副本。

函数`changeCreature`被定义为接受一个名为`creature`的参数，并且它是我们前面定义的类型`Creature`。然后我们将`Species`字段的值更改为`jellyfish`并将其打印出来。 请注意，在`changeCreature`函数中，`Species`的值现在是`jellyfish`，并打印出`2) {Species:jellyfish}`。 这是因为我们可以在函数作用域中修改值。

然而，当`main`函数的最后一行打印出`creature`的值时，`Species`的值仍然是`shark`。 值没有改变的原因是因为我们通过值传递了变量。 这意味着在内存中创建了该值的副本，并将其传递给`changeCreature`函数。 这允许我们有一个函数，可以根据需要对传入的任何参数进行更改，但不会影响函数外部的任何变量。

接下来，让我们将`changeCreature`函数改为通过引用获取参数。 我们可以通过使用星号（`creature`）操作符将类型从`*`改为指针来实现这一点。 而不是传递一个`creature`，我们现在传递一个指针到一个`creature`，或一个`*creature`。在前面的示例中，`creature`是一个`struct`，其`Species`值为`shark`。`*creature`是一个指针，而不是一个结构体，所以它的值是一个内存位置，这就是我们传递给`changeCreature()`的内容。

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func main() {
 var creature Creature = Creature{Species: "shark"}

 fmt.Printf("1) %+v\n", creature)
 changeCreature(&creature)
 fmt.Printf("3) %+v\n", creature)
}

func changeCreature(creature *Creature) {
 creature.Species = "jellyfish"
 fmt.Printf("2) %+v\n", creature)
}
```

运行此代码以查看以下输出：

```
Output
1) {Species:shark}
2) &{Species:jellyfish}
3) {Species:jellyfish}
```

请注意，现在当我们在`Species`函数中将`jellyfish`的值更改为`changeCreature`时，它也会更改`main`函数中定义的原始值。 这是因为我们通过引用传递了`creature`变量，这允许访问原始值并可以根据需要更改它。

因此，如果你希望一个函数能够改变一个值，你需要通过引用传递它。要通过引用传递，你传递的是指向变量的指针，而不是变量本身。

然而，有时候你可能没有为指针定义一个实际的值。在这种情况下，程序可能会出现[死机]。让我们看看这是如何发生的，以及如何为潜在的问题做计划。

## 零指针

Go中的所有变量都是[零值]。即使对于指针也是如此。如果你声明了一个指向类型的指针，但没有赋值，那么零值将是`nil`。 `nil`是一种表示变量“没有初始化”的方式。

在下面的程序中，我们定义了一个指向`Creature`类型的指针，但是我们从来没有实例化一个`Creature`类型的实际实例，并将它的地址分配给`creature`指针变量。 值将是`nil`，我们不能引用任何将在`Creature`类型上定义的字段或方法：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func main() {
 var creature *Creature

 fmt.Printf("1) %+v\n", creature)
 changeCreature(creature)
 fmt.Printf("3) %+v\n", creature)
}

func changeCreature(creature *Creature) {
 creature.Species = "jellyfish"
 fmt.Printf("2) %+v\n", creature)
}
```

输出如下所示：

```
Output
1) <nil>
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x8 pc=0x109ac86]

goroutine 1 [running]:
main.changeCreature(0x0)
        /Users/corylanou/projects/learn/src/github.com/gopherguides/learn/_training/digital-ocean/pointers/src/nil.go:18 +0x26
 main.main()
         /Users/corylanou/projects/learn/src/github.com/gopherguides/learn/_training/digital-ocean/pointers/src/nil.go:13 +0x98
  exit status 2
```

当我们运行程序时，它打印出`creature`变量的值，该值为`<nil>`。 然后我们调用`changeCreature`函数，当该函数试图设置`Species`字段的值时，它会死机。 这是因为没有实际创建的变量的实例。 正因为如此，程序没有实际存储值的位置，所以程序会死机。

在Go语言中，如果你接收到一个参数作为指针，你会在对它执行任何操作之前检查它是否为nil，以防止程序恐慌。

这是检查`nil`的常见方法：

```go
if someVariable == nil {
 // print an error or return from the method or fuction
}
```

实际上，你要确保没有`nil`指针被传递到你的函数或方法中。 如果你这样做，你可能只是想返回，或返回一个错误，以表明一个无效的参数被传递给函数或方法。下面的代码演示了如何检查`nil`：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func main() {
 var creature *Creature

 fmt.Printf("1) %+v\n", creature)
 changeCreature(creature)
 fmt.Printf("3) %+v\n", creature)
}

func changeCreature(creature *Creature) {
 if creature == nil {
  fmt.Println("creature is nil")
  return
 }

 creature.Species = "jellyfish"
 fmt.Printf("2) %+v\n", creature)
}
```

我们在`changeCreature`中添加了一个检查，以查看`creature`参数的值是否为`nil`。 如果是，我们打印出“creature is nil”，然后从函数中返回。 否则，我们继续并更改`Species`字段的值。 如果我们运行这个程序，我们将得到以下输出：

```
Output
1) <nil>
creature is nil
3) <nil>
```

请注意，虽然我们仍然有`nil`变量的`creature`值，但我们不再恐慌，因为我们正在检查这种情况。

最后，如果我们创建一个`Creature`类型的实例并将其赋给`creature`变量，程序现在将按预期更改值：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func main() {
 var creature *Creature
 creature = &Creature{Species: "shark"}

 fmt.Printf("1) %+v\n", creature)
 changeCreature(creature)
 fmt.Printf("3) %+v\n", creature)
}

func changeCreature(creature *Creature) {
 if creature == nil {
  fmt.Println("creature is nil")
  return
 }

 creature.Species = "jellyfish"
 fmt.Printf("2) %+v\n", creature)
}
```

现在我们有了一个`Creature`类型的实例，程序将运行，我们将得到以下预期输出：

```
Output
1) &{Species:shark}
2) &{Species:jellyfish}
3) &{Species:jellyfish}
```

当您使用指针时，程序可能会死机。 为了避免恐慌，您应该在尝试访问任何定义在其上的字段或方法之前检查指针值是否为`nil`。

接下来，让我们看看使用指针和值如何影响类型上的方法定义。

## 方法指针接收器

go中的接收器是在方法声明中定义的参数。 看看下面的代码：

```go
type Creature struct {
 Species string
}

func (c Creature) String() string {
 return c.Species
}
```

这个方法中的接收器是`c Creature`。 它声明了`c`的实例是`Creature`类型的，你将通过该实例变量引用该类型。

就像函数的行为根据你是将参数作为指针还是值发送而不同一样，方法也有不同的行为。最大的区别是，如果你定义了一个带有值接收器的方法，你就不能对定义该方法的那个类型的实例进行更改。

有时候你希望你的方法能够更新你正在使用的变量的实例。 为了实现这一点，您需要将接收器设置为指针。

让我们给我们的`Reset`类型添加一个`Creature`方法，它将`Species`字段设置为空字符串：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func (c Creature) Reset() {
 c.Species = ""
}

func main() {
 var creature Creature = Creature{Species: "shark"}

 fmt.Printf("1) %+v\n", creature)
 creature.Reset()
 fmt.Printf("2) %+v\n", creature)
}
```

如果我们运行程序，我们将得到以下输出：

```
Output
1) {Species:shark}
2) {Species:shark}
```

请注意，即使在`Reset`方法中，我们将`Species`的值设置为空字符串，但当我们在`creature`函数中打印出`main`变量的值时，该值仍然设置为`shark`。这是因为我们定义了`Reset`方法有一个`value`接收器。 这意味着该方法只能访问`creature`变量的副本。

如果我们想修改方法中`creature`变量的实例，我们需要将它们定义为具有`pointer`接收器：

main.go

```go
package main

import "fmt"

type Creature struct {
 Species string
}

func (c *Creature) Reset() {
 c.Species = ""
}

func main() {
 var creature Creature = Creature{Species: "shark"}

 fmt.Printf("1) %+v\n", creature)
 creature.Reset()
 fmt.Printf("2) %+v\n", creature)
}
```

请注意，我们现在在定义`*`方法时，在`Creature`类型前面添加了一个星号（`Reset`）。这意味着传递给`Creature`方法的`Reset`的实例现在是一个指针，因此当我们进行更改时，它将影响该变量的原始实例。

```
Output
1) {Species:shark}
2) {Species:}
```

`Reset`方法现在已经更改了`Species`字段的值。

## 结论

将函数或方法定义为按值传递或按引用传递将影响程序的哪些部分能够对其他部分进行更改。控制变量何时可以改变将允许您编写更健壮和可预测的软件。 现在你已经了解了指针，你可以看到它们是如何在接口中使用的。
