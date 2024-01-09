# 如何在自己的项目中使用私有 Go 模块

### 导言

Go 生态系统的一个有利方面是大量模块都是开源的。因为它们是开源的，所以可以自由访问、检查、使用和学习。不过，有时出于各种原因，例如为了保持公司内部的专有业务逻辑，有必要制作私有 Go 模块。

在本教程中，您将发布私有 Go 模块，设置访问私有模块的身份验证，并在项目中使用私有 Go 模块。

## 先决条件

- 已安装 Go 1.16 或更高版本。要进行此设置，请按照操作系统的 "[如何安装 Go]"教程进行操作。
- 了解[如何]分发 Go 模块，请参阅如何分发 Go 模块教程。
- 熟悉 Git，您可以通过《[如何使用 Git：参考指南]》。
- 一个名为`mysecret`的空 GitHub 私有仓库，用于存放已发布的私有模块。要开始使用，请按照[GitHub 文档创建一个仓库](https://docs.github.com/en/get-started/quickstart/create-a-repo)。
- [GitHub 个人访问令牌](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)，具有读取仓库内容的权限。您可以用它允许 Go 访问您的私人仓库。

## 分发专用模块

与许多编程语言不同，Go 通过资源库而非中央软件包服务器发布模块。这种方法的一个好处是，发布私有模块与发布公共模块非常相似。Go 私有模块不需要完全独立的私有包服务器，而是通过私有源代码库发布。由于大多数源代码托管选项都支持开箱即用，因此无需设置额外的私有服务器。

要使用私有模块，您需要有访问私有 Go 模块的权限。在本节中，您将创建并发布一个私有模块，以便在本教程稍后部分使用该模块从另一个 Go 程序访问私有模块。

要创建新的私有 Go 模块，首先要克隆该模块所在的私有 GitHub 仓库。作为先决条件的一部分，你创建了一个名为 `mysecret`的私有空仓库，这就是你将用于私有模块的仓库。这个仓库可以克隆到电脑上的任何地方，但许多开发者倾向于为自己的项目建立一个目录。本教程将使用名为`projects` 的目录。

创建`projects`目录并导航至该目录：

```bash
mkdir projects
cd projects
```

在`projects`目录下，运行`git clone`你的私有 `mysecret`仓库克隆到您的电脑上：

```bash
git clone git@github.com:your_github_username/mysecret.git
```

Git 会确认它已克隆了你的模块，并可能警告你克隆了一个空仓库。如果是这样，您不必担心：

```bash
Output
Cloning into 'mysecret'...
warning: You appear to have cloned an empty repository.
```

接下来，使用`cd`进入新的 `mysecret`目录，并使用`go mod init` 和你的私有仓库名称创建一个新的 Go 模块：

```bash
cd mysecret
go mod init github.com/your_github_username/mysecret
```

模块创建完成后，现在是时候添加一个可以从其他项目中使用的函数了。使用`nano` 或你喜欢的文本编辑器，打开一个与版本库同名的文件，如`mysecret.go`。文件名并不重要，可以是任何名字，但使用与版本库相同的名字可以让我们在处理新模块时更容易确定首先要查看哪个文件：

```bash
nano mysecret.go
```

在`mysecret.go`文件中，将软件包命名为与版本库相同的名称，然后添加一个`SecretProcess`函数，用于在调用时打印`Running the secret process!`的行：

```go title="projects/mysecret/mysecret.go"
package mysecret

import "fmt"

func SecretProcess() {
	fmt.Println("Running the secret process!")
}
```

现在您已经创建了自己的私有模块，您将把它发布到您的私有资源库中供他人使用。由于您的私人资源库最初只允许您访问，因此您可以控制谁可以访问您的私人模块。您可以限制自己访问，但也可以让朋友或同事访问。

由于私有和公共 Go 模块都是源代码库，发布私有 Go 模块的过程与发布公共模块相同。要发布新模块，请使用`git add`命令在当前目录下`添加`改动，然后使用`git commit`命令将这些改动提交到本地仓库：

```bash
git add .
git commit -m "Initial private module implementation"
```

您将看到 Git 发出的初始提交成功的确认信息，以及提交中包含的文件摘要：

```
Output
[main (root-commit) bda059d] Initial private module implementation
 2 files changed, 10 insertions(+)
 create mode 100644 go.mod
 create mode 100644 mysecret.go
```

现在唯一剩下的部分就是把你的改动移到 GitHub 仓库了。与公共模块类似，使用`git push`命令发布代码：

```bash
git push
```

然后，Git 会推送您的更改，并将它们提供给任何可以访问您的私有仓库的人：

```
git push origin main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 404 bytes | 404.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:your_github_username/mysecret.git
 * [new branch]      main -> main
```

与公共 Go 模块一样，您也可以为私有 Go 模块添加版本。[如何发布 Go]模块教程》中的 "[发布新模块版本]"部分包含了如何做到这一点的信息。

在本节中，你创建了一个带有`SecretProcess`函数的新模块，并将其发布到你的私有`mysecret`GitHub 仓库，使其成为一个私有 Go 模块。不过，为了从其他 Go 程序访问该模块，你需要配置 Go，使它知道如何访问该模块。

## 配置转到访问专用模块

Go 模块通常从源代码库中发布，但 Go 团队也运行一些[中央 Go 模块服务](https://proxy.golang.org/)，以确保在原始代码库发生故障时模块继续存在。默认情况下，Go 会被配置为使用这些服务，但当你试图下载私有模块时，这些服务可能会造成问题，因为它们无法访问这些模块。要告诉 Go 某些导入路径是私有的，并且它不应该尝试使用中央 Go 服务，可以使用`GOPRIVATE`环境变量。`GOPRIVATE`环境变量是一个以逗号分隔的导入路径前缀列表，当遇到这些前缀时，Go 工具会尝试直接访问它们，而不是通过中央服务。你刚刚创建的私有模块就是这样一个例子。

为了使用私有模块，您需要在`GOPRIVATE`变量中设置哪条路径为私有，从而告诉 Go 哪条路径为私有。在设置`GOPRIVATE`变量值时，有几种选择。一种选择是将`GOPRIVATE`设为`github.com`。但这可能不是你想要的，因为这会告诉 Go 不要使用任何托管在`github.com` 上的模块的中心服务，包括那些不属于你的模块。

下一个选择是只将`GOPRIVATE`设为自己的用户路径，比如`github.com/your_github_username`。这样就解决了将 GitHub 全部视为私有的问题，但在某些时候，你可能会创建一些公共模块，希望通过 Go 模块镜像下载。这样做不会造成任何问题，也是完全合理的选择，但你也可以选择更具体的方式。

最具体的选项是将`GOPRIVATE`设置为与模块路径完全匹配，例如：`github.com/your_github_username/mysecret`。这解决了前面两个选项的问题，但也带来了一个问题，即你需要在`GOPRIVATE`中单独添加你的每个私有源，如图所示：

```
GOPRIVATE=github.com/your_github_username/mysecret,github.com/your_github_username/othersecret

```

选择最适合自己的方案，需要根据自己的情况权衡利弊。

由于你现在只有一个私有模块，我们将使用完整的版本库名称作为值。要在当前终端设置`GOPRIVATE=github.com/your_github_username/mysecret`环境变量，请使用 export 命令：

```bash
export GOPRIVATE=github.com/your_github_username/mysecret
```

如果要仔细检查是否已设置，可以使用`env`命令和`grep`命令来检查`GOPRIVATE`名称：

```bash
env | grep GOPRIVATE
```

```
Output
GOPRIVATE=github.com/your_github_username/mysecret
```

尽管 Go 现在知道你的模块是私有的，但还不足以使用该模块。如果你试图`go get`私有模块放入另一个模块，很可能会看到类似以下的错误：

```bash
go get github.com/your_github_username/mysecret
```

```
Output
go get: module github.com/your_github_username/mysecret: git ls-remote -q origin in /Users/your_github_username/go/pkg/mod/cache/vcs/2f8c...b9ea: exit status 128:
	fatal: could not read Username for 'https://github.com': terminal prompts disabled
Confirm the import path was entered correctly.
If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

此错误信息表示 Go 尝试下载模块，但遇到了它仍无法访问的问题。由于 Git 被用来下载模块，它通常会要求你输入凭据。但在这种情况下，Go 会为你调用 Git，所以无法提示输入凭据。此时，要访问模块，你需要提供一种方法，让 Git 在不需要你立即输入的情况下获取你的凭据。

## 为 HTTPS 提供专用模块证书

告诉 Git 如何代表你登录的一种方法是`.netrc`文件。`.netrc`文件位于用户的主目录下，包含各种主机名以及这些主机的登录凭证。它被许多工具广泛使用，包括 Git。

默认情况下，当`go get`尝试下载模块时，它会首先尝试使用 HTTPS。不过，如上例所示，它无法提示你输入用户名和密码。要向 Git 提供用户名和密码，你需要在主目录下创建一个包含`github.com`的`.netrc`。

要在 Linux、MacOS 或 Windows Subsystem for Linux (WSL) 上创建`.netrc`文件，请在主目录`(~/`) 中打开`.netrc`文件以便编辑：

```bash
nano ~/.netrc
```

接下来，在文件中创建一个新条目。`machine`值应该是你要设置证书的主机名，这里是`github.com`。`login`值应该是你的 GitHub 用户名。最后，`password`值应该是你创建的[GitHub 个人访问令牌](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)。

~/.netrc

```
machine github.com
login your_github_username
password your_github_access_token
```

如果您愿意，也可以将整个条目放在文件的一行中：

~/.netrc

```
machine github.com login your_github_username password your_github_access_token

```

{==

**注意：**如果使用[Bitbucket](https://bitbucket.org/)作为源代码托管，则除了`bitbucket.org` 之外，可能还需要为`api` `.`bitbucket.org 添加第二个条目。过去，Bitbucket 为多种类型的版本控制提供托管服务，因此 Go 会在尝试下载之前使用 API 检查版本库的类型。虽然这种情况已不复存在，但 API 检查仍然存在。如果遇到此问题，错误信息示例可能如下所示：

```
go get bitbucket.org/your_github_username/mysecret: reading https://api.bitbucket.org/2.0/repositories/your_bitbucket_username/protocol?fields=scm: 403 Forbidden
	server response: Access denied. You must have write or admin access.
```

如果在尝试下载私有模块时看到`403 Forbidden`错误，请仔细检查 Go 尝试连接的主机名。这可能表明你需要在`.netrc`文件中添加另一个主机名，如`api.bitbucket.org`。

==}

现在，你的环境已设置为使用 HTTPS 身份验证下载私有模块。尽管 HTTPS 是 Go 和 Git 尝试下载模块的默认方式，但也可以告诉 Git 使用 SSH 代替。使用 SSH 而非 HTTPS 非常有用，你可以使用推送私有模块时使用的 SSH 密钥。如果不想创建个人访问令牌，还可以在设置[CI/CD]环境时使用[部署密钥](https://docs.github.com/en/developers/overview/managing-deploy-keys)。

## 为 SSH 提供专用模块证书

要使用 SSH 密钥而非 HTTPS 作为私有 Go 模块的身份验证方式，Git 提供了一个名为`insteadOf` 的配置选项。通过`insteadOf`选项，你可以说 "与其 "使用`https://github.com/`作为所有 Git 请求的 URL，不如使用`ssh://git@github.com/。`

在 Linux、MacOS 和 WSL 上，该配置位于`.gitconfig`文件中。你可能已经熟悉这个文件了，因为你的提交电子邮件地址和姓名也是在这里配置的。要编辑该文件，请使用`nano` 或你最喜欢的文本编辑器，打开主目录下的`~/.gitconfig`文件：

```bash
nano ~/.gitconfig
```

打开文件后，按下面的示例编辑文件，为`ssh://git@github.com/`添加一个`url`部分：

```bash title="~/.gitconfig"
[user]
	email = your_github_username@example.com
	name = Sammy the Shark
	
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```

`url`部分相对于`user`部分的顺序并不重要，如果文件中除了您刚刚添加的`url`部分外没有其他内容，您也不必担心。`user`部分中`email`和`name`字段的顺序也无关紧要。

这个新章节告诉 Git，任何以`https://github.com/`开头的 URL 前缀都应替换为`ssh://git@github.com/`。由于 Go 默认使用 HTTPS，这也会影响你的`go get`命令。以你的私有模块为例，这意味着 Go 会把`github.com/your_github_username/mysecret`导入路径变成 URL`https://github.com/your_github_username/mysecret。`当 Git 遇到这个 URL 时，它会发现该 URL 与`insteadOf`引用的`https://github.com/`前缀相匹配，并将生成的 URL 变成`ssh://git@github.com/your_github_username/mysecret。`

只要`ssh://git@`URL 也适用于 GitHub 以外的主机，该模式也可用于其他域。

在本节中，你通过更新`.gitconfig`文件并添加`url`部分，配置 Git 使用 SSH 下载 Go 模块。现在，私人模块的身份验证已经设置完毕，你可以在 Go 程序中访问它了。

## 使用专用模块

在前面的章节中，你配置了 Go，以便通过 HTTPS、SSH 或两者访问你的私有 Go 模块。现在，Go 可以访问你的私有模块，它的使用方式与你过去使用的任何公共模块类似。在本节中，您将创建一个使用私有模块的新 Go 模块。

在用于项目的目录（如`projects`）中，创建一个名为 `projects`目录：

```bash
mkdir myproject
```

创建目录后，使用`cd`转到该目录，然后根据项目所在的版本库 URL（如`github.com/your_github_username/myproject`），使用 go`mod`init 为项目初始化一个新的 Go 模块。如果不打算将项目推送到其他版本库，模块名称可以是 `myproject`或其他任何名称，但最好使用完整的 URL，因为大多数共享模块都需要使用 URL。

```bash
cd myproject
go mod init github.com/your_github_username/myproject
```

```bash
Output
go: creating new go.mod: module github.com/your_github_username/myproject
```

现在，用`nano` 或你最喜欢的文本编辑器打开`main.go`，创建项目的第一个代码文件：

```bash
nano main.go
```

在该文件中，设置您将调用私有模块的初始`主`函数：

```go title="projects/myproject/main.go"
package main

import "fmt"

func main() {
	fmt.Println("My new project!")
}
```

要立即运行项目并确保一切设置正确，可以使用`go run` 命令并向其提供`main.go`文件：

```bash
go run main.go
```

```
Output
My new project!
```

接下来，使用`go get` 将私有模块添加为新项目的依赖关系，就像添加公共模块一样：

```bash
go get github.com/your_github_username/mysecret
```


然后，`go`工具会下载您的私有模块代码，并使用与最新提交哈希值和提交时间相匹配的版本字符串将其添加为依赖关系：

```
Output
go: downloading github.com/your_github_username/mysecret v0.0.0-20210920195630-bda059d63fa2
go get: added github.com/your_github_username/mysecret v0.0.0-20210920195630-bda059d63fa2
```

最后，再次打开 `main.go`文件并更新，在`main`函数中添加对私有模块`SecretProcess`函数的调用。你还需要更新`import`语句，将你的`github.com/your_github_username/mysecret`私有模块也作为导入：

```go title="projects/myproject/main.go"
package main

import (
	"fmt"

	"github.com/your_github_username/mysecret"
)

func main() {
	fmt.Println("My new project!")
	mysecret.SecretProcess()
}
```

要查看使用私有模块运行的最终项目，请再次使用`go run`命令，并将`main.go`文件作为参数：

```bash
go run main.go
```

你会看到原始代码中的 "`My new project!`"一行，但现在你还会看到导入的`mysecret`模块中的 "`Running the secret process!！`"一行：

```
Output
My new project!
Running the secret process!
```

在本节中，你使用`go init`创建了一个新的 Go 模块，以访问你之前发布的私有模块。创建好模块后，你就可以使用`go get`下载私有模块，就像下载公共 Go 模块一样。最后，使用 go`run`编译并运行使用私有模块的 Go 程序。

## 结论

在本教程中，您创建并发布了一个私有 Go 模块。您还设置了 HTTPS 和 SSH 身份验证，以访问您的私有 Go 模块。最后，您在一个新项目中使用了私有模块。

有关 Go 模块的更多信息，Go 项目有[一系列博文](https://blog.golang.org/using-go-modules)详细介绍了 Go 工具如何与模块交互并理解模块。Go 项目还在《[Go 模块参考](https://golang.org/ref/mod)》（Go Modules Reference）中提供了非常详细的 Go 模块技术参考。

除了`GOPRIVATE`环境变量外，在使用私有 Go 模块时还可以使用更多变量。有关这些变量的详细信息，请参阅[Go 模块参考](https://golang.org/ref/mod)中的[私有模块](https://golang.org/ref/mod#private-modules)部分。

如果你想更详细地了解`.netrc`文件，[GNU 网站](https://www.gnu.org/software/inetutils/manual/html_node/The-_002enetrc-file.html)上的 . `netrc`包含了所有可用关键字的列表。此外，[`git-config`文档](https://git-scm.com/docs/git-config#Documentation/git-config.txt-urlltbasegtinsteadOf)还提供了更多关于您使用的`insteadOf`配置选项以及其他可用选项如何工作的信息。
