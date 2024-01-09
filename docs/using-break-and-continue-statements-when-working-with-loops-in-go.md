# 在 Go 中使用循环时使用 Break 和 continue 语句

### [介绍]

在Go中使用for循环可以让你以一种高效的方式自动化和重复任务。

学习如何控制循环的操作和流程将允许在程序中定制逻辑。 你可以使用`break`和`continue`语句来控制你的循环。

## [Break语句]

在Go中，`break`语句终止当前循环的执行。`break`几乎总是与[条件`if`语句]配对。

让我们看一个在`break`循环中使用`for`语句的例子：

break.go

```go
package main

import "fmt"

func main() {
 for i := 0; i < 10; i++ {
  if i == 5 {
   fmt.Println("Breaking out of loop")
   break // break here
  }
  fmt.Println("The value of i is", i)
 }
 fmt.Println("Exiting program")
}
```


这个小程序创建了一个`for`循环，当`i`小于`10`时，它将执行循环。

在`for`循环中，有一个`if`语句。 `if`语句测试`i`的条件，以查看该值是否小于`5`。如果`i`的值不等于`5`，则循环继续并打印出`i`的值。如果`i`的值等于`5`，循环将执行`break`语句，打印它是`Breaking out of loop`，并停止执行循环。在程序结束时，我们打印出`Exiting program`来表示我们已经退出了循环。

当我们运行这段代码时，我们的输出将如下所示：

```
Output  The value of i is 0
The value of i is 1
The value of i is 2
The value of i is 3
The value of i is 4
Breaking out of loop
Exiting program
```

这表明，一旦整数`i`被计算为等于5，循环就会中断，因为程序被告知使用`break`语句这样做。

### [嵌套循环]

重要的是要记住，`break`语句只会停止最里面的循环的执行。 如果你有一组嵌套的循环，你需要在每个循环中有一个break。

nested.go

```go
package main

import "fmt"

func main() {
 for outer := 0; outer < 5; outer++ {
  if outer == 3 {
   fmt.Println("Breaking out of outer loop")
   break // break here
  }
  fmt.Println("The value of outer is", outer)
  for inner := 0; inner < 5; inner++ {
   if inner == 2 {
    fmt.Println("Breaking out of inner loop")
    break // break here
   }
   fmt.Println("The value of inner is", inner)
  }
 }
 fmt.Println("Exiting program")
}
```


在这个程序中，我们有两个循环。虽然这两个循环都重复了205次，但每个循环都有一个条件`if`语句和一个`break`语句。如果`outer`的值等于`3`，则外部循环将中断。 如果`inner`的值是`2`，则内部循环将中断。

如果我们运行程序，我们可以看到输出：

```
Output  The value of outer is 0
The value of inner is 0
The value of inner is 1
Breaking out of inner loop
The value of outer is 1
The value of inner is 0
The value of inner is 1
Breaking out of inner loop
The value of outer is 2
The value of inner is 0
The value of inner is 1
Breaking out of inner loop
Breaking out of outer loop
Exiting program
```

请注意，每次内部循环中断时，外部循环不会中断。这是因为`break`只会破坏调用它的最内层循环。

我们已经看到了如何使用`break`来停止循环的执行。 接下来，让我们看看如何继续循环的迭代。

## [Continue语句]

当你想跳过循环的剩余部分，返回到循环的顶部并继续新的迭代时，使用`continue`语句。

与`break`语句一样，`continue`语句通常与条件`if`语句一起使用。

使用与前面[Break语句]部分相同的`for`循环程序，我们将使用`continue`语句而不是`break`语句：

continue.go

```go
package main

import "fmt"

func main() {
 for i := 0; i < 10; i++ {
  if i == 5 {
   fmt.Println("Continuing loop")
   continue // break here
  }
  fmt.Println("The value of i is", i)
 }
 fmt.Println("Exiting program")
}
```


使用`continue`语句而不是`break`语句的区别在于，当变量`i`被评估为等同于`5`时，尽管中断，我们的代码仍将继续。让我们看看我们的输出：

```
Output  The value of i is 0
The value of i is 1
The value of i is 2
The value of i is 3
The value of i is 4
Continuing loop
The value of i is 6
The value of i is 7
The value of i is 8
The value of i is 9
Exiting program
```

在这里我们看到，行`The value of i is 5`从未出现在输出中，但循环在该点之后继续打印数字6-10的行，然后离开循环。

您可以使用`continue`语句来避免深度嵌套的条件代码，或者通过消除您想要拒绝的频繁出现的情况来优化循环。

`continue`语句使程序跳过循环中出现的某些因素，但随后继续循环的其余部分。

## [结论]

Go语言中的`break`和`continue`语句可以让你在代码中更有效地使用`for`循环。
