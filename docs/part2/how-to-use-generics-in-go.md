# 如何在 Go 中使用泛型

## 介绍

在Go 1.18中，该语言引入了一个名为泛型类型的新特性（通常称为较短的术语，泛型），这已经在Go开发人员的愿望清单上有一段时间了。在编程中，[泛型类型](https://en.wikipedia.org/wiki/Generic_programming)是可以与多个其他类型结合使用的类型。通常在Go语言中，如果你想对同一个变量使用两种不同的类型，你但是，使用`io.Reader`可能会使使用这些类型变得具有挑战性，因为您需要在其他几个潜在类型之间进行转换以与它们进行交互。使用泛型类型可以让您直接与类型交互，从而使代码更清晰、易读。

在本教程中，您将创建一个与一副扑克牌交互的程序。您将开始创建一个使用`interface{}`与卡片交互的卡片组，然后将其更新为使用泛型类型。在这些更新之后，您将使用泛型将第二种类型的牌添加到您的牌组中，然后您将更新您的牌组以将其泛型类型约束为仅支持牌类型。最后，您将创建一个使用您的卡片并支持泛型类型的函数。

## 先决条件

要遵循本教程，您需要：

- Go 1.18或更高版本已安装。要设置此设置，请按照[如何]为您的操作系统安装Go教程。
- 对Go语言有扎实的理解，如[变量]、[函数]、[`struct`类型]、[`for`循环]和[切片]。如果您

## Go Collections Without Generics

Go语言的一个强大特性是它能够使用接口灵活地表示许多类型。许多用Go语言编写的代码可以很好地使用接口提供的功能。这就是为什么Go语言存在了这么长时间而不支持泛型的原因之一。

在本教程中，您将创建一个Go程序，模拟从`PlayingCard`牌中随机获得`Deck`牌。在本节中，您将使用`interface{}`来允许`Deck`与任何类型的卡进行交互。在本教程的后面部分，您将更新程序以使用泛型，这样您就可以更好地理解它们之间的差异，并认识到何时一个比另一个更好。

编程语言中的类型系统通常可以分为两类：类型化和类型检查。一种语言可以使用强类型或弱类型，以及静态或动态类型检查。一些语言混合使用这些，但Go非常适合强类型和静态检查的语言。强类型意味着Go确保变量中的值与变量的类型相匹配，因此您不能将`int`值存储在`string`变量中。作为一个静态检查的类型系统，Go的编译器将在编译程序时而不是在程序运行时检查这些类型规则。

使用像Go这样的强类型、静态检查的语言的好处是，编译器可以在程序发布之前让你知道任何潜在的错误，避免某些“无效类型”的运行时错误。但是，这确实给Go程序增加了一个限制，因为在编译程序之前，你必须知道你打算使用的类型。处理这个问题的一种方法是使用`interface{}`类型。`interface{}`类型适用于任何值的原因是因为它没有为接口定义任何必需的方法（由空的`{}`表示），所以任何类型都匹配接口。

要开始使用`interface{}`来代表您的卡片创建程序，您需要一个目录来保存程序的目录。在本教程中，您将使用名为`projects`的目录。

首先，创建`projects`目录并导航到它：

```bash
mkdir projects
cd projects
```

接下来，创建项目目录并导航到该目录。在本例中，使用目录`httpclient`：

```bash
mkdir generics
cd generics
```

在`httpclient`目录中，使用`nano`或您最喜欢的编辑器打开`main.go`文件：

```bash
nano main.go
```

在`main.go`文件中，开始添加您的`package`声明和`import`您需要的包：

main.go

```go
package main

import (
 "fmt"
 "math/rand"
 "os"
 "time"
)
```

`package main`声明告诉Go将程序编译为二进制文件，这样你就可以直接运行它，而`import`语句告诉Go你将在后面的代码中使用哪些包。

现在，定义你的`PlayingCard`类型及其相关的函数和方法：

main.go

```go
...

type PlayingCard struct {
 Suit string
 Rank string
}

func NewPlayingCard(suit string, card string) *PlayingCard {
 return &PlayingCard{Suit: suit, Rank: card}
}

func (pc *PlayingCard) String() string {
 return fmt.Sprintf("%s of %s", pc.Rank, pc.Suit)
}
```

在此代码段中，您定义了一个名为`PlayingCard`的结构，该结构具有属性`Suit`和`Rank`，以表示一副52张扑克牌中的牌。`Suit`将是`Diamonds`、`Hearts`、`Clubs`或`Spades`中的一个，`Rank`将是`A`、`2`、`3`，依此类推，直到`K`。

您还定义了一个`NewPlayingCard`函数作为`PlayingCard` `struct`的构造函数，以及一个`String`方法，它将使用`fmt.Sprintf`返回牌的等级和花色。

接下来，使用`Deck`和`AddCard`方法创建您的`RandomCard`类型，以及一个`NewPlayingCardDeck`函数来创建一个装满所有52张扑克牌的`*Deck`：

main.go

```go
...

type Deck struct {
 cards []interface{}
}

func NewPlayingCardDeck() *Deck {
 suits := []string{"Diamonds", "Hearts", "Clubs", "Spades"}
 ranks := []string{"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"}

 deck := &Deck{}
 for _, suit := range suits {
  for _, rank := range ranks {
   deck.AddCard(NewPlayingCard(suit, rank))
  }
 }
 return deck
}

func (d *Deck) AddCard(card interface{}) {
 d.cards = append(d.cards, card)
}

func (d *Deck) RandomCard() interface{} {
 r := rand.New(rand.NewSource(time.Now().UnixNano()))

 cardIdx := r.Intn(len(d.cards))
 return d.cards[cardIdx]
}
```

在上面定义的`Deck`中，您创建了一个名为`cards`的字段来保存一张牌。由于您希望这副牌能够容纳多种不同类型的牌，因此您不能将其定义为`[]*PlayingCard`。您将其定义为`[]interface{}`，以便它可以容纳您将来可能创建的任何类型的卡。除了`[]interface{}`上的`Deck`字段，您还创建了一个接受相同`AddCard`类型的`interface{}`方法，以将一张卡片附加到`Deck`的`cards`字段。

您还创建了一个`RandomCard`方法，该方法将从`Deck`的`cards`切片中返回一张随机卡片。此方法使用[`math/rand`](https://pkg.go.dev/math/rand)包生成`0`和`cards`切片中的卡片数量之间的随机数。`rand.New`行使用当前时间作为随机性的来源创建一个新的随机数生成器;否则，随机数可能每次都相同。`r.Intn(len(d.cards))`行使用随机数生成器生成`int`和提供的数字之间的`0`值。由于`Intn`方法不包括数字范围内的参数值，因此不需要从长度中减去`1`来说明从`0`开始。最后，`RandomCard`返回随机数索引处的卡值。

警告：小心你在程序中使用的随机数生成器。[`math/rand`](https://pkg.go.dev/math/rand)包在[加密上不安全](https://en.wikipedia.org/wiki/Cryptographically-secure_pseudorandom_number_generator)，不应该用于安全敏感的程序。然而，[`crypto.rand`](https://pkg.go.dev/crypto/rand)包确实提供了一个可以用于这些目的的随机数生成器。

最后，`NewPlayingCardDeck`函数返回一个填充了一副扑克牌中所有牌的`*Deck`值。您使用两个切片，一个包含所有可用的花色，另一个包含所有可用的点数，然后循环每个值，为每个组合创建一个新的`*PlayingCard`，然后使用`AddCard`将其添加到套牌中。一旦生成了这副牌的牌，就会返回值。

现在你已经设置好了`Deck`和`PlayingCard`，你可以创建你的`main`函数来使用它们来抽牌：

main.go

```go
...

func main() {
 deck := NewPlayingCardDeck()

 fmt.Printf("--- drawing playing card ---\n")
 card := deck.RandomCard()
 fmt.Printf("drew card: %s\n", card)

 playingCard, ok := card.(*PlayingCard)
 if !ok {
  fmt.Printf("card received wasn't a playing card!")
  os.Exit(1)
 }
 fmt.Printf("card suit: %s\n", playingCard.Suit)
 fmt.Printf("card rank: %s\n", playingCard.Rank)
}
```

在`main`函数中，首先使用`NewPlayingCardDeck`函数创建一副新的扑克牌，并将其分配给`deck`变量。然后，你使用`fmt.Printf`打印出你正在抽一张牌，并使用`deck`的`RandomCard`方法从牌组中随机抽取一张牌。之后，您再次使用`fmt.Printf`打印您从甲板上抽出的卡片。

接下来，由于`card`变量的类型是`interface{}`，因此需要使用[类型断言](https://go.dev/tour/methods/15)来获取对卡片的引用，作为其原始的`*PlayingCard`类型。如果`card`变量中的类型不是`*PlayingCard`类型（这应该是你现在编写程序的方式），那么`ok`的值将是`false`，你的程序将使用`fmt.Printf`打印错误消息，并使用`1`退出错误代码`os.Exit`。如果是`*PlayingCard`类型，则使用`playingCard`打印出`Suit`的`Rank`和`fmt.Printf`值。

一旦你保存了所有的更改，你可以使用`go run`和`main.go`来运行你的程序，这是要运行的文件的名称：

```bash
go run main.go
```

在程序的输出中，您应该看到从牌组中随机选择的一张牌，以及牌组和等级：

```
Output
--- drawing playing card ---
drew card: Q of Diamonds
card suit: Diamonds
card rank: Q
```

由于这张牌是从牌堆中随机抽取的，您的输出可能与上面显示的输出不同，但您应该会看到类似的输出。第一行是在从牌堆中抽取随机牌之前打印的，然后第二行是在抽取牌后打印的。你可以看到卡片的输出使用了`PlayingCard`的`String`方法返回的值。最后，您可以看到在将您的`interface{}`卡值断言为`*PlayingCard`值后打印的两行花色和等级输出。

在本节中，您创建了一个使用`Deck`值来存储任何值并与之交互的`interface{}`，以及一个用作`PlayingCard`中的卡片的`Deck`类型。然后，您使用`Deck`和`PlayingCard`s从牌组中随机选择一张牌，并打印出有关该牌的信息。

但是，要访问有关您绘制的`*PlayingCard`值的特定信息，您需要执行一些额外的工作，将`interface{}`类型转换为能够访问`*PlayingCard`和`Suit`字段的`Rank`类型。以这种方式使用`Deck`将有效，但如果将`*PlayingCard`以外的值添加到`Deck`，则也可能导致错误。通过更新你的`Deck`来使用泛型，你可以从Go的强类型和静态类型检查中受益，同时仍然具有接受`interface{}`值提供的灵活性。

## Go Collections With泛型

在上一节中，您使用`interface{}`类型的切片创建了一个集合。但是要使用这些值，您需要做一些额外的工作，将值从`interface{}`转换为这些值的实际类型。但是，使用泛型，您可以创建一个或多个类型参数，这些参数的作用几乎与函数参数相似，但它们可以将类型作为值而不是数据保存。通过这种方式，泛型提供了一种在每次使用泛型类型时用不同类型替换类型参数的方法。这就是泛型类型的名字来源。由于泛型类型可以与多种类型一起使用，而不仅仅是像`io.Reader`或`interface{}`这样的特定类型，因此它足够通用以适应多个用例。

在本节中，您将更新您的`Deck`类型为泛型类型，当您创建`Deck`而不是使用`interface{}`的实例时，它可以使用任何特定类型的卡。

要进行第一次更新，请打开`main.go`文件并删除`os`包导入：

main.go

```go
package main

import (
 "fmt"
 "math/rand"
 // "os" package import is removed
 "time"
)
```

正如您将在后续更新中看到的那样，您将不再需要使用`os.Exit`函数，因此可以安全地删除此导入。

接下来，将`Deck``struct`更新为泛型类型：

main.go

```go
...

type Deck[C any] struct {
 cards []C
}
```

此更新引入了泛型用来在结构声明中创建占位符类型或类型参数的新语法。您几乎可以将这些类型参数视为类似于函数中包含的参数。调用函数时，为每个函数参数提供值。同样，在创建泛型类型值时，您要为类型参数提供类型。

您将看到在您的`struct`、`Deck`的名称之后，您在方括号（`[]`）中添加了一条语句。这些方括号允许您为`struct`定义一个或多个类型参数。

对于`Deck`类型，您只需要一个名为`C`的类型参数来表示牌组中的牌的类型。通过在类型参数中声明`C any`，你的代码说，“创建一个名为`C`的泛型类型参数，我可以在我的`struct`中使用，并允许它是`any`类型”。在幕后，`any`类型实际上是`interface{}`类型的别名。这使得泛型更容易阅读，并且您不需要使用`C interface{}`。您的牌组只需要一个泛型类型来表示牌，但如果您需要其他泛型类型，则可以使用逗号分隔语法添加它们，例如`C any, F any`。类型参数的名称可以是任意的，只要不是保留的，但它们通常都是简短的，并且大写。

最后，在对`Deck`声明的更新中，您更新了`cards`中`struct`切片的类型，以使用`C`类型参数。使用泛型时，可以在通常放置特定类型的任何地方使用类型参数。在本例中，您希望`C`参数表示切片中的每张牌，因此将`[]`切片类型声明放在`C`参数声明之后。

接下来，更新您的`Deck`类型的`AddCard`方法以使用您定义的泛型类型。现在，您将跳过更新`NewPlayingCardDeck`函数，但您将很快回到它：

main.go

```go
...

func (d *Deck[C]) AddCard(card C) {
 d.cards = append(d.cards, card)
}
```

在对`Deck`的`AddCard`方法的更新中，您首先将`[C]`泛型类型参数添加到方法的接收器中。这让Go知道你将在方法声明的其他地方使用的类型参数的名称，并遵循与`struct`声明类似的方括号语法。但是，在这种情况下，您不需要提供`any`约束，因为它已经在`Deck`的声明中提供了。然后，您更新了`card`函数参数，以使用`C`占位符类型而不是原始的`interface{}`类型。这使得该方法使用特定的类型`C`最终将成为。

在更新了`AddCard`方法之后，更新`RandomCard`方法以使用`C`泛型类型：

main.go

```go
...

func (d *Deck[C]) RandomCard() C {
 r := rand.New(rand.NewSource(time.Now().UnixNano()))

 cardIdx := r.Intn(len(d.cards))
 return d.cards[cardIdx]
}
```

这一次，您没有使用`C`泛型类型作为函数参数，而是更新了方法以返回值`C`而不是`interface{}`。除了更新接收器以包含`[C]`之外，这是您需要对函数进行的唯一更新。由于`cards`上的`Deck`字段已经在`struct`声明中更新，因此当此方法返回来自`cards`的值时，它返回的是类型为`C`的值。

现在你的`Deck`类型已经更新为使用泛型，回到你的`NewPlayingCardDeck`函数，并更新它以使用`Deck`类型的泛型`*PlayingCard`类型：

main.go

```go
...

func NewPlayingCardDeck() *Deck[*PlayingCard] {
 suits := []string{"Diamonds", "Hearts", "Clubs", "Spades"}
 ranks := []string{"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"}

 deck := &Deck[*PlayingCard]{}
 for _, suit := range suits {
  for _, rank := range ranks {
   deck.AddCard(NewPlayingCard(suit, rank))
  }
 }
 return deck
}

...
```

`NewPlayingCardDeck`中的大多数代码保持不变，但现在您使用的是`Deck`的泛型版本，您需要在使用`C`时指定您希望用于`Deck`的类型。你可以像往常一样引用你的`Deck`类型，不管它是`Deck`还是像`*Deck`这样的引用，然后通过使用最初声明类型参数时使用的相同方括号来提供应该替换`C`的类型。

对于`NewPlayingCardDeck`返回类型，您仍然像以前一样使用`*Deck`，但这次，您还包括方括号和`*PlayingCard`。通过为类型参数提供`[*PlayingCard]`，你是说你希望在你的`*PlayingCard`声明和方法中使用`Deck`类型来替换`C`的值。这意味着`cards`上的`Deck`字段的类型本质上从`[]C`变为`[]*PlayingCard`。

类似地，当创建`Deck`的新实例时，您还需要提供替换`C`的类型。在通常使用`&Deck{}`对`Deck`进行新引用的地方，您可以将类型包含在方括号中，以`&Deck[*PlayingCard]{}`结束。

现在你的类型已经更新为使用泛型，你可以更新你的`main`函数来利用它们：

main.go

```go
...

func main() {
 deck := NewPlayingCardDeck()

 fmt.Printf("--- drawing playing card ---\n")
 playingCard := deck.RandomCard()
 fmt.Printf("drew card: %s\n", playingCard)
 // Code removed
 fmt.Printf("card suit: %s\n", playingCard.Suit)
 fmt.Printf("card rank: %s\n", playingCard.Rank)
}
```

这一次，您的更新是删除代码，因为您不再需要将`interface{}`值断言为`*PlayingCard`值。当您更新`Deck`的`RandomCard`方法以返回`C`并更新`NewPlayingCardDeck`以返回`*Deck[*PlayingCard]`时，它将`RandomCard`更改为返回`*PlayingCard`值而不是`interface{}`。当`RandomCard`返回`*PlayingCard`时，这意味着`playingCard`的类型也是`*PlayingCard`而不是`interface{}`，您可以立即访问`Suit`或`Rank`字段。

若要在将更改保存到`main.go`后查看程序的运行情况，请再次使用`go run`命令：

```bash
go run main.go
```

您应该会看到类似于以下输出的输出，但抽到的牌可能会有所不同：

```
Output
--- drawing playing card ---
drew card: 8 of Hearts
card suit: Hearts
card rank: 8
```

即使输出与使用`interface{}`的前一个版本的程序相同，代码本身也更干净一些，避免了潜在的错误。您不再需要对`*PlayingCard`类型进行断言，从而避免了额外的错误处理。此外，通过说你的`Deck`实例只能包含`*PlayingCard`，除了`*PlayingCard`之外，不可能有其他值被添加到`cards`切片。

在本节中，您将`Deck``struct`更新为通用类型，从而对套牌的每个实例可以包含的牌类型提供更多控制。您还更新了`AddCard`和`RandomCard`方法，以接受泛型参数或返回泛型值。然后，您更新了`NewPlayingCardDeck`以返回包含`*Deck`卡的`*PlayingCard`。最后，您删除了`main`函数中的错误处理，因为您不再需要它了。

现在您的`Deck`已更新为通用，可以使用它来保存您想要的任何类型的卡。在下一节中，您将通过向程序中添加一种新类型的卡来利用这种灵活性。

## 对多个类型使用泛型

一旦你创建了一个泛型类型，比如你的`Deck`，你就可以将它与任何其他类型一起使用。当您创建泛型`Deck`的实例并希望它与`*PlayingCard`类型一起使用时，您唯一需要做的就是在创建值时指定该类型。为了支持不同的类型，您可以将`*PlayingCard`类型与您想要使用的新类型交换。

在本节中，您将创建一个新的`TradingCard``struct`类型来表示不同类型的卡。然后，您将更新您的程序以创建`Deck`4#s。

要创建`TradingCard`类型，请再次打开`main.go`文件并添加定义：

main.go

```go
...

import (
 ...
)

type TradingCard struct {
 CollectableName string
}

func NewTradingCard(collectableName string) *TradingCard {
 return &TradingCard{CollectableName: collectableName}
}

func (tc *TradingCard) String() string {
 return tc.CollectableName
}
```

这个`TradingCard`类似于你的`PlayingCard`，但是它没有`Suit`和`Rank`字段，而是有一个`CollectableName`字段来跟踪交易卡的名称。它还包括`NewTradingCard`构造函数和`String`方法，类似于`PlayingCard`。

现在，为一个填充有`NewTradingCardDeck`s的`Deck`创建`*TradingCard`构造函数：

main.go

```go
...

func NewPlayingCardDeck() *Deck[*PlayingCard] {
 ...
}

func NewTradingCardDeck() *Deck[*TradingCard] {
 collectables := []string{"Sammy", "Droplets", "Spaces", "App Platform"}

 deck := &Deck[*TradingCard]{}
 for _, collectable := range collectables {
  deck.AddCard(NewTradingCard(collectable))
 }
 return deck
}
```

当您创建或返回`*Deck`时，您已经用`*PlayingCard`替换了`*TradingCard`，但这是您需要对甲板进行的唯一更改。你有一片特殊的[DigitalOcean]收藏品，然后你循环将每个`*TradingCard`添加到甲板上。`deck`的`AddCard`方法仍然以相同的方式工作，但这次它接受来自`*TradingCard`的`NewTradingCard`值。如果你试图从`NewPlayingCard`中传递一个值，编译器会给予你一个错误，因为它会期待一个`*TradingCard`，但你提供了一个`*PlayingCard`。

最后，更新你的`main`函数，创建一个新的`Deck`/`*TradingCard`，随机抽取一张带有`RandomCard`的卡片，并打印出卡片的信息：

main.go

```go
...

func main() {
 playingDeck := NewPlayingCardDeck()
 tradingDeck := NewTradingCardDeck()

 fmt.Printf("--- drawing playing card ---\n")
 playingCard := playingDeck.RandomCard()
 ...
 fmt.Printf("card rank: %s\n", playingCard.Rank)

 fmt.Printf("--- drawing trading card ---\n")
 tradingCard := tradingDeck.RandomCard()
 fmt.Printf("drew card: %s\n", tradingCard)
 fmt.Printf("card collectable name: %s\n", tradingCard.CollectableName)
}
```

在上一次更新中，您使用`NewTradingCardDeck`创建了一个新的交易卡组，并将其存储在`tradingDeck`中。然后，由于您仍然使用与以前相同的`Deck`类型，您可以使用`RandomCard`从甲板上随机获得一张交易卡并打印出卡片。您还可以直接在`CollectableName`上引用和打印出`tradingCard`字段，因为您使用的泛型`Deck`已将`C`定义为`*TradingCard`。

此更新还显示了使用泛型的价值。要支持全新的卡片类型，您根本无需更改`Deck`。类型参数`Deck`允许您在创建`Deck`的实例时提供要使用的卡类型，并且从那时起，与该`Deck`值的任何交互都使用`*TradingCard`类型而不是`*PlayingCard`类型。

要查看更新后的代码，请保存更改并使用`go run`再次运行程序：

```bash
go run main.go
```

然后，查看您的输出：

```
Output
--- drawing playing card ---
drew card: Q of Diamonds
card suit: Diamonds
card rank: Q
--- drawing trading card ---
drew card: App Platform
card collectable name: App Platform
```

一旦程序完成运行，您应该会看到类似于上面输出的输出，只是使用了不同的卡。您将看到两张牌被抽中：原始扑克牌和您添加的新交易牌。

在本节中，您添加了一个新的`TradingCard`类型，以表示与原始`PlayingCard`不同的卡类型。一旦添加了`TradingCard`类型，就创建了`NewTradingCardDeck`构造函数来创建并填充交易卡组。最后，您更新了您的`main`方法，以使用新的交易卡组并打印出有关正在抽取的随机卡的信息。

除了创建一个新的函数`NewTradingCardDeck`来用不同的卡片填充`Deck`之外，您不需要对`Deck`进行任何其他更新来支持全新的卡片类型。这就是泛型类型的力量。您可以编写一次代码，并将其重用于多种其他类型的类似数据。然而，当前的`Deck`的一个问题是，由于您拥有的`C any`声明，它可以用于任何类型。这可能是你想要的东西，这样你就可以使用`int`创建一组`&Deck[int]{}`值。但是如果你想让你的`Deck`只包含卡片，你需要一种方法来限制`C`允许的数据类型。

## 限制泛型类型

通常，您不希望或不需要对泛型使用的类型进行任何限制，因为您不必关心特定的数据。但是，在其他时候，您需要能够限制泛型使用的类型。例如，如果您正在创建一个泛型`Sorter`类型，您可能希望将其泛型类型限制为具有`Compare`方法的泛型类型，以便`Sorter`可以比较它所持有的项。如果你不包括这个限制，这些值甚至可能没有`Compare`方法，你的`Sorter`将不知道如何比较它们。

在本节中，您将为您的卡创建一个新的`Card`接口来实现，然后更新您的`Deck`以仅允许添加`Card`类型。

要开始更新，请打开`main.go`文件并添加`Card`接口：

main.go

```go
...

import (
...
)

type Card interface {
 fmt.Stringer

 Name() string
}
```

您的`Card`接口的定义与您过去使用过的任何其他[Go接口]相同;使用泛型没有特殊要求。在这个`Card`接口中，你

接下来，更新你的`TradingCard`和`PlayingCard`类型，在现有的`Name`方法之外添加一个新的`String`方法，这样它们就实现了`Card`接口：

main.go

```go
...

type TradingCard struct {
 ...
}

...

func (tc *TradingCard) Name() string {
 return tc.String()
}

...

type PlayingCard struct {
 ...
}

...

func (pc *PlayingCard) Name() string {
 return pc.String()
}
```

`TradingCard`和`PlayingCard`已经有了实现`String`接口的`fmt.Stringer`方法。因此，要实现`Card`接口，只需添加新的`Name`方法。此外，由于已经实现了返回卡片名称的`fmt.Stringer`，因此您可以返回`String`的`Name`方法的结果。

现在，更新你的`Deck`，使它只允许`Card`类型用于`C`：

main.go

```go
...

type Deck[C Card] struct {
 cards []C
}
```

在此更新之前，类型限制（称为类型约束）有`C any`，这不是一个很大的限制。由于`any`和`interface{}`的意思相同，所以它允许Go语言中的任何类型都可以用于`C`类型参数。现在你已经用新的`any`接口替换了`Card`，Go编译器将确保在你编译程序时用于`C`的任何类型都实现了`Card`。

既然你已经添加了这个限制，你现在可以在你的`Card`类型的方法中使用`Deck`提供的任何方法。如果你想让`RandomCard`也打印出被抽牌的名字，它可以访问`Name`方法，因为它是`Card`接口的一部分。您将在下一节中看到它的实际应用。

这几个更新是您需要做的唯一更新，以限制您的`Deck`类型仅使用`Card`值。保存更改后，使用`go run`运行更新后的程序：

```bash
go run main.go
```

然后，一旦你的程序完成运行，检查输出：

```
Output
--- drawing playing card ---
drew card: 5 of Clubs
card suit: Clubs
card rank: 5
--- drawing trading card ---
drew card: Droplets
card collectable name: Droplets
```

你会看到，除了选择不同的卡，输出没有改变。由于您的更新仅将值限制为您已经使用的类型，因此程序的功能没有更改。

在本节中，您添加了一个新的`Card`接口，并更新了`TradingCard`和`PlayingCard`以实现该接口。您还更新了`Deck`的类型约束，将其类型参数限制为仅实现`Card`接口的类型。

不过，到目前为止，您只创建了一个泛型`struct`类型。除了创建泛型类型，Go还允许创建泛型函数。

## 创建泛型函数

Go语言中的泛型函数与Go语言中的其他泛型类型有着非常相似的语法。当您考虑到其他泛型类型具有类型参数时，生成泛型函数就是向这些函数添加第二组参数。

在本节中，您将创建一个新的`printCard`泛型函数，并使用该函数打印出所提供的卡的名称。

要实现新函数，请打开`main.go`文件并进行以下更新：

main.go

```go
...

func printCard[C any](card C) {
 fmt.Println("card name:", card.Name())
}

func main() {
 ...
 fmt.Printf("card collectable name: %s\n", tradingCard.CollectableName)

 fmt.Printf("--- printing cards ---\n")
 printCard[*PlayingCard](playingCard)
 printCard(tradingCard)
}
```

在`printCard`函数中，您将看到泛型类型参数的熟悉的方括号语法，后面是括号中的常规函数参数。然后，在`main`函数中，使用`printCard`函数打印出`*PlayingCard`和`*TradingCard`。

您可能会注意到，对`printCard`的一个调用包含`[*PlayingCard]`类型参数，而第二个调用不包含相同的`[*TradingCard]`类型参数。Go编译器能够通过你传递给函数参数的值来确定想要的类型参数，所以在这种情况下，类型参数是可选的。如果你愿意，你也可以删除`[*PlayingCard]`类型参数。

现在，保存您的更改并使用`go run`：：再次运行程序。

```bash
go run main.go
```

不过，这一次，你会看到一个编译器错误：

```
Output
# command-line-arguments
./main.go:87:33: card.Name undefined (type C has no field or method Name)
```

在`printCard`函数的类型参数中，您将`any`作为`C`的类型约束。当Go编译程序时，它只希望看到由`any`接口定义的方法，而那里没有。这就是对类型参数使用特定类型约束的好处。为了访问你的卡片类型的`Name`方法，你需要告诉Go用于`C`参数的唯一类型是`Card`s。

最后一次更新您的`main.go`文件，将`any`类型约束替换为`Card`类型约束：

main.go

```go
...

func printCard[C Card](card C) {
 fmt.Println("card name:", card.Name())
}
```

然后，保存您的更改并使用`go run`运行程序：

```bash
go run main.go
```

你的程序现在应该成功运行了：

```
Output
--- drawing playing card ---
drew card: 6 of Hearts
card suit: Hearts
card rank: 6
--- drawing trading card ---
drew card: App Platform
card collectable name: App Platform
--- printing cards ---
card name: 6 of Hearts
card name: App Platform
```

在这个输出中，您将看到两张卡片都被绘制和打印出来，就像您熟悉的那样，但是现在`printCard`函数也打印出卡片，并使用它们的`Name`方法来获取要打印的名称。

在本节中，您创建了一个新的泛型`printCard`函数，它可以接受任何`Card`值并打印名称。您还看到了使用类型约束`any`而不是`Card`或其他特定值如何影响可用的方法。

## 结论

在本教程中，您创建了一个新程序，其中的`Deck`可以从牌组中返回一张随机牌作为`interface{}`值，而`PlayingCard`类型则表示牌组中的一张扑克牌。然后，您更新了`Deck`类型以支持泛型类型参数，并且能够删除一些错误检查，因为泛型类型确保了这种类型的错误不会再发生。在那之后，你创建了一个新的`TradingCard`类型来代表你的`Deck`可以支持的不同类型的卡，以及创建每种类型的卡的套牌，并从每个套牌中返回一张随机卡。接下来，您向`Deck`添加了类型约束，以确保只有实现了`Card`接口的类型才能添加到deck中。最后，您创建了一个通用的`printCard`函数，它可以使用`Card`方法打印任何`Name`值的名称。

在代码中使用泛型可以大大减少为同一代码支持多种类型所需的代码量，但也有可能过度使用泛型。在使用泛型而不是接口作为值时，存在性能和代码易读性的权衡，因此如果您可以使用接口而不是泛型，那么最好首选接口。然而，泛型仍然是非常强大的工具，当在它们擅长的情况下使用时，可以使开发人员的生活更轻松。
