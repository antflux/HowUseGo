# 如何分发 Go 模块

## [导言

许多现代编程语言都允许开发人员发布现成的库，供他人在自己的程序中使用，Go 也不例外。有些语言使用中央资源库来安装这些库，而 Go 则从用于创建库的同一版本控制资源库中分发这些库。Go 还使用一种称为[semantic versioning](https://semver.org/)）的版本系统，向用户显示何时以及在哪些方面进行了修改。这可以帮助用户了解是否可以快速升级到某个模块的更新版本，并确保他们的软件可以继续使用该模块。

在本教程中，您将创建并发布一个新模块，学习使用语义版本管理，并发布模块的语义版本。

## 先决条件

- 已安装 Go 1.16 或更高版本。要进行此设置，请按照操作系统的 "[如何安装 Go]"教程进行操作。
- 了解 Go 模块，请参阅[如何使用 Go 模块]教程。
- 熟悉 Git，您可以通过《[如何使用 Git：参考指南]》。
- 一个名为`pubmodule`的空 GitHub 公共仓库，用于存放已发布的模块。要开始[创建仓库]，请参考GitHub 文档。

## 创建要发布的模块

与许多其他编程语言不同，Go 模块直接从其所在的源代码库发布，而不是从独立的软件包库发布。这使得用户更容易找到在其代码中引用的模块，模块维护者也更容易发布其模块的新版本。在本节中，你将创建一个新模块，然后将其发布，供其他用户使用。

要开始创建模块，您需要在作为先决条件的一部分创建的空仓库上使用`git clone`下载初始仓库。这个仓库可以克隆到电脑上的任何地方，但许多开发者倾向于为自己的项目建立一个目录。在本教程中，你将使用名为`projects` 的目录。

创建`projects`目录并导航至该目录：

```bash
mkdir projects
cd projects
```

在`projects`目录下运行`git clone`将仓库克隆到电脑上：

```bash
git clone git@github.com:your_github_username/pubmodule.git
```

克隆模块会将空模块下载到 `pubmodule`目录`中`。你可能会收到克隆了一个空仓库的警告，但这不用担心：

```
Output
Cloning into 'pubmodule'...
warning: You appear to have cloned an empty repository.
```

然后，切换到下载的目录：

```bash
cd pubmodule
```

进入模块目录后，使用`go mod init`创建新模块，并将版本库的位置作为模块名传入。确保模块名称与版本库位置相匹配非常重要，因为当模块在其他项目中使用时，`go`工具会以此找到下载模块的位置：

```bash
go mod init github.com/your_github_username/pubmodule
```

Go 会确认模块已创建，让你知道它已创建了`go.mod`文件：

```
Output
go: creating new go.mod: module github.com/your_github_username/pubmodule
```

最后，使用你最喜欢的文本编辑器（如`nano`）创建并打开一个与版本库同名的文件：`pubmodule.go`。

```bash
nano pubmodule.go
```

这个文件的名称可以是任何名称，但使用与软件包相同的名称会让你在处理一个陌生软件包时更容易知道从哪里开始。不过，软件包名称本身应该与版本库名称相同。这样，当别人引用你软件包中的方法或类型时，它就会与版本库的名称相匹配，比如`pubmodule.MyFunction`。这将使他们在以后需要引用时更容易知道软件包的来源。

接下来，在软件包中添加一个`Hello`方法，返回字符串`Hello, You!`。任何导入软件包的人都可以使用这个函数：

```go title="projects/pubmodule/pubmodule.go"
package pubmodule

func Hello() string {
  return "Hello, You!"
}
```

现在，你已经使用`go mod init`创建了一个新模块，模块名称与远程仓库`（github.com/your_github_username/pubmodule`）一致。你还在模块中添加了一个名为`pubmodule.go`的文件，其中包含一个模块用户可以调用的名为`Hello`的函数。接下来，你将发布你的模块，让其他人也能使用。

## 发布模块

创建了本地模块并准备好向其他用户发布后，就可以发布模块了。由于 Go 模块是通过与它们存储在同一个代码库中的代码发布的，因此你需要将代码提交到本地 Git 代码库，然后推送到你位于`github.com/your_github_username/pubmodule` 的代码库中。

在将代码提交到本地 Git 仓库之前，最好先确保不会提交任何你不希望提交的文件，因为这些文件会在你将代码推送到 GitHub 时被公开发布。在`pubmodule`目录下使用`git status`命令会显示所有将要提交的文件和改动：

```bash
git status
```

输出结果将与此类似：

```
Output
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
 go.mod
 pubmodule.go
```

你应该能看到`go mod init`命令创建的`go.mod`文件，以及你创建`Hello`函数的`pubmodule.go`文件。根据你创建版本库的方式，你可能会有一个与此输出不同的分支名称。最常见的分支名称是`main`或`master`。

确定只找到要找的文件后，就可以用`git add`将文件分期，然后用`git commit` 将它们提交到版本库：

```bash
git add .
git commit -m "Initial Commit"
```

输出结果将与此类似：

```
Output
[main (root-commit) 931071d] Initial Commit
 2 files changed, 8 insertions(+)
 create mode 100644 go.mod
 create mode 100644 pubmodule.go
```

最后，使用`git push`命令将模块推送到 GitHub 仓库：

```bash
git push
```

输出结果将与此类似：

```
Output
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 367 bytes | 367.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:your_github_username/pubmodule.git
 * [new branch]      main -> main
```

运行`git push`命令后，你的模块就会被推送到你的版本库中，可供其他人使用。如果您没有发布任何版本，Go 会使用您仓库默认分支中的代码作为模块的代码。至于你的默认分支是叫`main`、`master` 还是别的什么名字，这并不重要，重要的是你的版本库默认分支的设置。

在本节中，你将创建的本地 Go 模块发布到 GitHub 仓库，供其他人使用。虽然你现在有了一个已发布的模块，但维护公共模块的另一项工作是确保模块的用户可以使用模块的稳定版本。你很可能想对模块进行修改并添加功能，但如果你在模块中不使用版本就进行修改，可能会不小心破坏使用你模块的人的代码。为了解决这个问题，你可以在模块开发到一个新的里程碑时添加版本。不过，在添加新版本时，一定要选择一个有意义的版本号，以便用户知道是否可以立即升级。

## 语义版本管理

一个有意义的版本号能让用户了解与之交互的公共接口（或 API）发生了多大变化。Go 通过一种称为[语义版本管理（semantic versioning](https://semver.org/)）的版本管理方案来传达这些变更，简称 "SemVer"。(语义版本控制使用版本字符串来传达代码变更的含义，这就是语义版本控制名称的由来）。Go 的模块系统遵循 SemVer 来确定哪些版本比你当前使用的版本更新，以及某个模块的更新版本是否可以安全地自动升级到。

语义版本法赋予版本字符串中的每个数字以含义。SemVer 中的典型版本包含三个主要数字：主要版本、次要版本和补丁版本。每个数字都与一个`.`组合在一起，形成版本，如`1.2.3`。这些数字按大版本在前、小版本在后、补丁版本在后的顺序排列。这样，当查看一个版本时，就能知道哪个版本更新，因为特定位置的数字比以前的版本要高。例如，`2.2.3`版比`1.2.3`版新，因为主版本更高。同样，`1.4.3`版比 `1.2.10`版新，因为次版本更高。尽管在补丁版本中`10`高于`3`，但次要版本`4`高于`2`，因此该版本优先。例如，将次要版本 `1.3.10` 提高到`1.4.0`，将主要版本`2.4.1`提高到 `3.0.0`。

使用这些规则，Go 可以在运行`go get` 时决定使用哪个版本的模块。举个例子，假设你有一个使用`1.4.3`版本模块`github.com/your_github_username/pubmodule` 的项目。如果你依赖于`pubmodule`的稳定性，你可能只想自动升级补丁版本（`.3`）。如果运行 `go get -u=patch github.com/your_github_username/pubmodule`，Go 会看到你想升级模块的补丁版本，并只查找主次版本为`1.4`的新版本。

在创建模块的新版本时，必须考虑模块的公共 API 发生了哪些变化。语义版本字符串的每个部分都向您和您的用户传达了 API 变化的范围。这些变更类型通常分为三个不同的类别，与版本的每个组成部分相对应。最小的更改会增加补丁版本，中等的更改会增加次要版本，最大的更改会增加主要版本。使用这些类别来决定增加哪个版本号，可以帮助你避免破坏自己的代码和依赖于你的模块的其他人的代码。

### 版本号

SemVer 版本中的第一个数字是主版本号`（1.4.3`）。主要版本号是发布模块新版本时需要考虑的最重要的数字。主要版本号的变更是对公共应用程序接口进行向后兼容变更的信号。 向后不兼容的变更指的是模块中的任何变更，这些变更会导致他人在不做任何其他变更的情况下升级程序，从而导致程序崩溃。破坏可能是指由于函数名称改变而导致无法构建，或者库工作方式的改变导致同一方法返回`"v1 "`而非`"1"`。但这只适用于您的公共 API，即任何其他人可以使用的导出类型或方法。如果该版本只包含库用户不会注意到的改进，则不需要进行重大版本变更。记住哪些变更属于这一类别的一种方法是，任何被视为 "更新 "或 "删除 "的变更都属于主要版本升级。

{==

**注意：**与 SemVer 中其他类型的数字不同，主要版本`0`具有额外的特殊意义。主版本`0`被视为 "开发中 "版本。任何主版本为`0`的 SemVer 都不被视为稳定版本，API 中的任何内容都可能随时发生变化。创建新模块时，最好从主版本`0`开始，只更新次版本和补丁版本，直到完成模块的初步开发。一旦模块的公共应用程序接口完成更改并被用户认为稳定后，就可以开始使用`1.0.0` 版本了。

==}

以下面的代码为例，说明重大版本变更可能是什么样子。您有一个名为`UserAddress`的函数，它目前接受一个`string`作为参数，并返回一个`string`：

```go
func UserAddress(username string) string {
 // return user address as a string
}
```

虽然该函数目前返回一个`字符串`，但您可能会认为，如果该函数返回一个类似`*Address` 的`结构`，对您和您的用户来说会更好。这样，您就可以包含已经拆分开来的其他数据，例如邮政编码：

```go
type Address struct {
 Address    string
 PostalCode string
}

func UserAddress(username string) *Address {
 // return user address and postal code struct
}
```

这将是一个重大版本变更的例子，因为它需要用户修改自己的代码才能使用。如果您决定完全移除`UserAddress`，情况也是一样，因为用户需要更新他们的代码才能使用替代版本。

另一个重大版本变更的例子是为`UserAddress`函数添加一个新参数，即使它返回的仍然是`string`：

```go
func UserAddress(username string, uppercase bool) string {
 // return user address as a string, uppercase if bool is true
}
```

如果用户使用`UserAddress`函数，这一更改也要求他们更新代码，因此也需要对版本进行重大升级。

不过，并非所有对代码的修改都会如此剧烈。有时，您会对公共 API 进行更改，添加新的函数或值，但不会更改任何现有的函数或值。

### 次要版本号

SemVer 版本中的第二个数字是次版本号`（1.4.3`）。次版本号是对公共应用程序接口进行向后兼容变更的信号。向后兼容的变更是指不影响当前使用您的模块的代码或项目的任何变更。与主版本号类似，它只影响公共 API。记住哪些变更属于此类的方法是，任何被视为 "新增 "而非 "更新 "的变更都属于此类。

使用主要版本号中的同一示例，想象你有一个名为`UserAddress`的方法，它返回一个`string`：

```go
func UserAddress(username string) string {
 // return user address as a string
}
```

不过，这次你并没有将`UserAddress`更新为返回`*Address`，而是决定添加一个名为`UserAddressDetail` 的全新方法：

```go
type Address struct {
 Address    string
 PostalCode string
}

func UserAddress(username string) string {
 // return user address as a string
}

func UserAddressDetail(username string) *Address {
 // return user address and postal code struct
}
```

如果用户更新到这一版本的模块，添加这个新的`UserAddressDetail`函数并不需要用户做任何更改，因此这将被视为小版本号的增加。他们可以继续使用`UserAddress`，只需在希望包含`UserAddressDetail` 的附加信息时更新代码即可。

不过，公共 API 变动可能并不是您发布模块新版本的唯一时机。漏洞是软件开发中不可避免的一部分，而补丁版本号就是为了掩盖这些漏洞。

### 补丁版本号

补丁版本号是 SemVer 版本`（1.4.3`）中的最后一个数字。补丁版本变更是指**不影响**模块**公共 API** 的任何变更。不影响模块公共应用程序接口的更改通常是错误修复或安全修复。再次使用前面例子中的`UserAddress`函数，假设你发布的模块在函数返回的`string`中缺少了部分地址。如果你发布了一个新版本的模块来修复这个错误，它只会增加补丁版本。该版本不会改变用户使用`UserAddress`公共 API 的方式，而只会改变返回数据的正确性。

正如本节所述，精心选择新版本号是赢得用户信任的重要途径。使用语义版本向用户展示更新到新版本所需的工作量，这样就不会意外地让用户惊喜地发现更新会破坏他们的程序。在考虑了对模块所做的更改并确定了下一个版本号后，您就可以发布新版本并供用户使用了。

## 发布新模块版本

在发布模块的新版本之前，您需要根据计划做出的更改更新模块。如果没有任何更改，您将无法确定要增加语义版本的哪个部分。对于本教程中的模块，您将添加一个新的 "`Goodbye "`方法来补充 "`Hello`"方法，然后发布新版本供用户使用。

首先，打开`pubmodule.go`文件，在公共 API 中添加新的`Goodbye`方法：

```go title="pubmodule/pubmodule.go"
package pubmodule

func Hello() string {
  return "Hello, You!"
}

func Goodbye() string {
  return "Goodbye for now!"
}
```

保存更改后，您需要运行`git status` 查看哪些更改会被提交：

```bash
git status
```

输出结果将与此类似，显示模块中唯一的变化是您添加到`pubmodule.go` 中的方法：

```
Output
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
 modified:   pubmodule.go

no changes added to commit (use "git add" and/or "git commit -a")
```

接下来，将更改添加到暂存文件，并使用`git add`和`git commit` 将更改提交到本地仓库：

```bash
git add .
git commit -m "Add Goodbye method"
```

输出结果将与此类似：

```
Output
[main 3235010] Add Goodbye method
 1 file changed, 4 insertions(+)
```

提交更改后，您需要将其推送到 GitHub 仓库。在一个较大的软件项目中，或与其他开发人员合作完成一个项目时，这个步骤通常会略有不同。在开发一个新功能时，开发人员会创建一个 Git 分支来放置修改，直到新功能稳定并准备好发布。一旦如此，另一名开发人员就会审查该分支中的变更，以增加第二双眼睛，发现第一名开发人员可能遗漏的问题。审查完成后，该分支将被合并到默认分支（如`master`分支`main`分支）中。在两次版本发布之间，默认分支会积累这些类型的变更，直到发布新版本的时候。

由于您的模块在这里不会经历这一过程，因此将您所做的更改推送到版本库将模拟更改的累积：

```bash
git push
```

输出结果将与此类似：

```
Output
numerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 369 bytes | 369.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:your_github_username/pubmodule.git
   931071d..3235010  main -> main
```

输出结果显示，默认分支中的新代码已可供用户使用。

到此为止，您所做的一切都与最初发布模块时一样。不过，现在发布新版本的一个重要环节来了：选择新的版本号。

如果你看一下你对模块所做的改动，公共 API 的唯一改动（或者说任何改动）就是在模块中添加了`Goodbye`方法。由于用户可以从只有`Hello`函数的上一版本更新，而无需做出任何改动，因此这一改动属于向后兼容改动。在语义版本学中，对公共应用程序接口的向后兼容变更意味着次版本号的增加。不过，这是你发布的模块的第一个版本，所以没有以前的版本可以增加。如果将`0.0.0`视为 "无版本"，那么增加次版本号将导致版本`0.1.0`，即模块的下一个版本。

有了版本号，就可以用它和 Git 标签一起发布新版本了。当开发者使用 Git 来跟踪他们的源代码时，即使使用的不是 Go 语言，一个常见的习惯是使用 Git 的标签来跟踪特定版本的代码发布情况。这样，如果需要修改旧版本，就可以使用标签。由于 Go 已经从源代码库中下载了模块，因此它可以利用这种做法，使用相同的版本标记。

要使用这些标签发布自己模块的新版本，您需要用`git tag`命令标记您要发布的代码。作为`git tag`命令的参数，您还需要提供版本标签。要创建版本标签，请以表示版本的前缀`v` 开头，然后紧接着添加您的 SemVer。以您的模块为例，最终的版本标签将是`v0.1.0`。现在，运行`git tag`给模块加上版本标签：

```bash
git tag v0.1.0
```

在本地添加版本标签后，仍需将标签推送到 GitHub 仓库，可以使用`git push`with`origin` 来完成：

```bash
git push origin v0.1.0
```

`git push`命令成功执行后，您会看到一个新标签`v0.1.0` 已被创建：

```
Output
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:your_github_username/pubmodule.git
 * [new tag]         v0.1.0 -> v0.1.0
```

上面的输出显示，您的标签已被推送，您的 GitHub 仓库中有一个新的`v0.1.0`标签可供模块用户参考。

现在，只要你用`git 标签`发布了模块的新版本，每当用户运行`go get`获取模块的最新版本时，它就不会再根据默认分支的最新提交哈希值下载版本了。一旦模块有了发布版本，`go`工具就会开始使用这些版本来决定更新模块的最佳方式。与语义版本管理相结合，您就可以在迭代和改进模块的同时，为用户提供一致和稳定的体验。

## 结论

在本教程中，您创建了一个 Go 公共模块，并将其发布到 GitHub 仓库，以便其他人可以使用。您还使用语义版本法为模块确定了最佳版本号。最后，您扩展了模块的功能，并使用语义版本法发布了新版本，同时确信不会破坏依赖该模块的程序。

如果你想了解有关语义版本管理的更多信息，包括如何在版本中添加数字以外的信息，[语义版本管理网站（Semantic Versioning](https://semver.org/)）会有详细介绍。Go文档中也有一个[模块版本编号](https://golang.org/doc/modules/version-numbers)页面，专门解释Go是如何使用SemVer的。
