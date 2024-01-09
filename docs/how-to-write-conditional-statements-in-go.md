# 如何在 Go 中编写条件语句

### [介绍](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#introduction)

条件语句是每种编程语言的一部分。使用条件语句，我们可以让代码有时运行，有时不运行，这取决于当时程序的条件。

当我们完全执行程序的每一条语句时，我们并不是要求程序去评估特定的条件。通过使用条件语句，程序可以确定是否满足某些条件，然后被告知下一步该做什么。

让我们来看看一些使用条件语句的例子：

- 如果学生在考试中获得超过65%的成绩，报告她的成绩通过;如果没有，报告她的成绩不及格。
- 如果他的账户里有钱，就计算利息;如果没有，就收取罚款。
- 如果他们买10个或更多的橙子，计算5%的折扣;如果他们买的更少，那就不要。

通过评估条件并根据是否满足这些条件来分配代码运行，我们正在编写条件代码。

本教程将带您完成在Go编程语言中编写条件语句。

## [If语句](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#if-statements)

我们将从`if`语句开始，它将评估语句是真还是假，并仅在语句为真的情况下运行代码。

在纯文本编辑器中，打开一个文件并编写以下代码：

```go title="grade.go"
package main

import "fmt"

func main() {
 grade := 70

 if grade >= 65 {
  fmt.Println("Passing grade")
 }
}
```


在这段代码中，我们有了变量`grade`，并给它一个整数值`70`。然后我们使用`if`语句来评估变量等级是否大于或等于（`>=`）到`65`。如果它满足这个条件，我们告诉程序打印出[字符串](https://www.digitalocean.com/community/tutorials/an-introduction-to-working-with-strings-in-go)`Passing grade`。

保存程序为`grade.go`，并使用命令`go run grade.go`从[终端窗口在本地编程环境](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-go)中运行它。

在本例中，等级70确实满足大于或等于65的条件，因此一旦运行程序，您将收到以下输出：

```
Output Passing grade
```

现在让我们通过将`grade`变量的值更改为`60`来更改此程序的结果：

```go title="grade.go"
package main

import "fmt"

func main() {
 grade := 60

 if grade >= 65 {
  fmt.Println("Passing grade")
 }
}
```


当我们保存并运行这段代码时，我们将不会收到任何输出，因为条件没有得到满足，我们也没有告诉程序执行另一条语句。

再给予一个例子，让我们计算一个银行账户余额是否低于0。让我们创建一个名为`account.go`的文件并编写以下程序：

account.go

```go
package main

import "fmt"

func main() {
 balance := -5

 if balance < 0 {
  fmt.Println("Balance is below 0, add funds now or you will be charged a penalty.")
 }
}
```


当我们使用`go run account.go`运行程序时，我们将收到以下输出：

```
Output Balance is below 0, add funds now or you will be charged a penalty.
```

在程序中，我们用小于0的值`balance`初始化变量`-5`。由于余额满足了`if`语句（`balance < 0`）的条件，所以一旦我们保存并运行代码，我们将收到字符串输出。同样，如果我们将余额更改为0或正数，我们将不会收到任何输出。

## [Else语句](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#else-statements)

很可能我们希望程序在`if`语句计算结果为false时也能做一些事情。在我们的成绩示例中，我们需要输出成绩是及格还是不及格。

为此，我们将在上面的等级条件中添加一个`else`语句，其构造如下：

```go title="grade.go"
package main

import "fmt"

func main() {
 grade := 60

 if grade >= 65 {
  fmt.Println("Passing grade")
 } else {
  fmt.Println("Failing grade")
 }
}
```


由于等级变量的值为`60`，所以`if`语句的计算结果为false，因此程序不会打印出`Passing grade`。接下来的`else`语句告诉程序无论如何都要做一些事情。

当我们保存并运行程序时，我们将收到以下输出：

```
Output Failing grade
```

如果我们重写程序，给予等级`65`或更高的值，我们将得到输出`Passing grade`。

为了向银行账户示例添加`else`语句，我们像这样重写代码：

```go title="account.go"
package main

import "fmt"

func main() {
 balance := 522

 if balance < 0 {
  fmt.Println("Balance is below 0, add funds now or you will be charged a penalty.")
 } else {
  fmt.Println("Your balance is 0 or above.")
 }
}
```


```
Output Your balance is 0 or above.
```

在这里，我们将`balance`变量值更改为正数，以便打印`else`语句。为了得到要打印的第一个`if`语句，我们可以将值重写为负数。

通过将`if`语句与`else`语句组合，您正在构造一个由两部分组成的条件语句，该条件语句将告诉计算机执行某些代码，无论是否满足`if`条件。

## [Else if语句](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#else-if-statements)

到目前为止，我们已经为条件语句提供了一个[布尔](https://www.digitalocean.com/community/tutorials/understanding-boolean-logic-in-go)选项，每个`if`语句的值要么为真，要么为假。在许多情况下，我们需要一个评估两种以上可能结果的程序。为此，我们将使用一个else if语句，它在Go中被写为`else if`。`else if` or else if语句看起来像`if`语句，并将计算另一个条件。

在银行账户程序中，我们可能希望有三个离散的输出，用于三种不同的情况：

- 余额低于0
- 余额等于0
- 余额大于0

`else if`语句将被放置在`if`语句和`else`语句之间，如下所示：

```go title="account.go"
package main

import "fmt"

func main() {
 balance := 522

 if balance < 0 {
  fmt.Println("Balance is below 0, add funds now or you will be charged a penalty.")
 } else if balance == 0 {
  fmt.Println("Balance is equal to 0, add funds soon.")
 } else {
  fmt.Println("Your balance is 0 or above.")
 }
}
```


现在，一旦我们运行程序，有三种可能的输出：

- 如果变量`balance`等于`0`，我们将收到来自`else if`语句的输出（`Balance is equal to 0, add funds soon.`）
- 如果变量`balance`被设置为正数，我们将收到来自`else`语句（`Your balance is 0 or above.`）的输出。
- 如果变量`balance`被设置为负数，则输出将是来自`if`语句（`Balance is below 0, add funds now or you will be charged a penalty`）的字符串。

但是，如果我们想要有三种以上的可能性呢？我们可以通过在代码中写入多个`else if`语句来实现这一点。

在`grade.go`程序中，让我们重写代码，以便有几个字母等级对应于数字等级范围：

- 90分或以上相当于A级
- 80-89相当于B级
- 70-79相当于C级
- 65-69相当于D级
- 64或以下相当于F级

要运行这段代码，我们需要一个`if`语句，三个`else if`语句和一个`else`语句来处理所有失败的情况。

让我们重写前面例子中的代码，让字符串打印出每个字母等级。我们可以保持我们的`else`声明相同。

```go title="grade.go"
package main

import "fmt"

func main() {
 grade := 60

 if grade >= 90 {
  fmt.Println("A grade")
 } else if grade >= 80 {
  fmt.Println("B grade")
 } else if grade >= 70 {
  fmt.Println("C grade")
 } else if grade >= 65 {
  fmt.Println("D grade")
 } else {
  fmt.Println("Failing grade")
 }
}
```


由于`else if`语句将按顺序计算，因此我们可以保持语句的基本性。此程序正在完成以下步骤：

1. 如果分数大于90，程序将打印`A grade`，如果分数小于90，程序将继续下一条语句...
2. 如果成绩大于或等于80，程序将打印`B grade`，如果成绩小于或等于79，程序将继续下一条语句。
3. 如果成绩大于或等于70，程序将打印`C grade`，如果成绩小于或等于69，程序将继续下一条语句。
4. 如果成绩大于或等于65，程序将打印`D grade`，如果成绩小于或等于64，程序将继续下一条语句。
5. 程序将打印`Failing grade`，因为未满足上述所有条件。

## [嵌套的If语句](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#nested-if-statements)

一旦你对`if`、`else if`和`else`语句感到满意，你就可以继续使用嵌套的条件语句。我们可以使用嵌套的`if`语句来检查第一个条件是否为真。为此，我们可以在另一个if-else语句中包含一个if-else语句。让我们看看嵌套的`if`语句的语法：

```go
if statement1 { // outer if statement
 fmt.Println("true")

 if nested_statement { // nested if statement
  fmt.Println("yes")
 } else { // nested else statement
  fmt.Println("no")
 }

} else { // outer else statement
 fmt.Println("false")
}
```


这段代码可能会产生一些输出：

- 如果`statement1`的计算结果为true，那么程序将评估`nested_statement`是否也为true。如果两种情况都为真，则输出将为：

> ```
> Outputtrue
> yes
> ```

- 但是，如果`statement1`的计算结果为true，而`nested_statement`的计算结果为false，那么输出将是：

> ```
> Outputtrue
> no
> ```

- 如果`statement1`的计算结果为false，嵌套的if-else语句将不会运行，所以`else`语句将单独运行，输出将是：

> ```
> Outputfalse
> ```

我们也可以在整个代码中嵌套多个`if`语句：

```go
if statement1 { // outer if
 fmt.Println("hello world")

 if nested_statement1 { // first nested if
  fmt.Println("yes")

 } else if nested_statement2 { // first nested else if
  fmt.Println("maybe")

 } else { // first nested else
  fmt.Println("no")
 }

} else if statement2 { // outer else if
 fmt.Println("hello galaxy")

 if nested_statement3 { // second nested if
  fmt.Println("yes")
 } else if nested_statement4 { // second nested else if
  fmt.Println("maybe")
 } else { // second nested else
  fmt.Println("no")
 }

} else { // outer else
 statement("hello universe")
}
```


在这段代码中，除了`if`语句外，每个`if`语句中都有一个嵌套的`else if`语句。这将允许在每个条件下有更多的选择。

让我们看一个使用`if`程序的嵌套`grade.go`语句的例子。我们可以先检查一个分数是否通过（大于或等于65%），然后评估数字分数应该等于哪个字母分数。但是，如果成绩不及格，我们不需要遍历字母成绩，而是可以让程序报告成绩不及格。我们修改后的代码与嵌套的`if`语句看起来像这样：

```go title="grade.go"
package main

import "fmt"

func main() {
 grade := 92
 if grade >= 65 {
  fmt.Print("Passing grade of: ")

  if grade >= 90 {
   fmt.Println("A")

  } else if grade >= 80 {
   fmt.Println("B")

  } else if grade >= 70 {
   fmt.Println("C")

  } else if grade >= 65 {
   fmt.Println("D")
  }

 } else {
  fmt.Println("Failing grade")
 }
}
```


如果我们将变量`grade`设置为整数值`92`运行代码，则满足第一个条件，并且程序将打印出`Passing grade of:`。接下来，它将检查等级是否大于或等于90，由于也满足此条件，它将打印出`A`。

如果我们在运行代码时将`grade`变量设置为`60`，那么第一个条件就不满足了，所以程序将跳过嵌套的`if`语句并向下移动到`else`语句，程序将打印出`Failing grade`。

当然，我们可以添加更多的选项，并使用第二层嵌套的if语句。也许我们会想分别评估A+、A和A-的等级。我们可以先检查成绩是否合格，然后检查成绩是否在90分或以上，然后检查成绩是否超过96分，以获得A+：

```go title="grade.go"
...
if grade >= 65 {
 fmt.Print("Passing grade of: ")

 if grade >= 90 {
  if grade > 96 {
   fmt.Println("A+")

  } else if grade > 93 && grade <= 96 {
   fmt.Println("A")

  } else {
   fmt.Println("A-")
  }
...
```


在此代码中，对于设置为`grade`的`96`变量，程序将运行以下内容：

1. 检查等级是否大于或等于65（true）
2. 打印输出`Passing grade of:`
3. 检查等级是否大于或等于90（true）
4. 检查等级是否大于96（false）
5. 检查等级是否大于93且小于或等于96（true）
6. 图纸`A`
7. 保留这些嵌套的条件语句并继续执行剩余的代码

因此，96级的程序输出如下所示：

```
Output Passing grade of: A
```

嵌套的`if`语句可以为您的代码添加多个特定级别的条件。

## [结论](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-go#conclusion)

通过使用像`if`语句这样的条件语句，你可以更好地控制程序的执行。条件语句告诉程序评估是否满足某个条件。如果满足条件，它将执行特定的代码，但如果不满足条件，程序将继续向下移动到其他代码。
