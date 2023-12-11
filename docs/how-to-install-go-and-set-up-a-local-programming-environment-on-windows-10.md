# 先决条件

您需要一台连接到互联网的 Windows 10 计算机，并具有管理员权限。

## 步骤1 —— 打开和配置 PowerShell

您将在大部分安装和设置过程中使用命令行界面，这是与计算机进行交互的非图形方式。与其点击按钮，您将输入文本并通过文本从计算机获得反馈。命令行，也称为 shell，可以帮助您修改和自动化日常计算机任务的许多操作，是软件开发人员的基本工具。

PowerShell 是微软提供的一个命令行 shell 接口的程序。通过运行 cmdlet（发音为 command-let），也就是能够执行操作的 .NET 软件框架的专用类，执行管理员任务。PowerShell 在 2016 年 8 月开源，现在跨平台，在 Windows 和 UNIX 系统（包括 Mac 和 Linux）上都可以使用。

要找到 Windows PowerShell，您可以右键单击屏幕左下角的开始菜单图标。当菜单弹出时，点击搜索，然后在搜索栏中键入 PowerShell。在出现选项时，右键单击桌面应用程序的 Windows PowerShell。在本教程中，请选择以管理员身份运行。当出现询问“是否允许此应用对计算机进行更改”的对话框时，点击是。

一旦您这样做了，您将看到一个基于文本的界面，其中有一串词，看起来像这样：

```
Copy code
Windows 10 PowerShell
```

通过输入以下命令退出系统文件夹：

```
bashCopy code
cd ~
```

然后您将位于主目录，例如 `PS C:\Users\sammy`。

为了继续安装过程，您首先必须通过 PowerShell 设置权限。默认情况下配置为以最安全模式运行，管理员可以设置几个权限级别：

- Restricted 是默认的执行策略。在此模式下，您将无法运行脚本，PowerShell 只能作为交互式 shell 使用。
- AllSigned 允许运行所有由受信任发布者签名的脚本和配置文件，这意味着您可能会打开机器来运行由受信任发布者签名的恶意脚本的风险。
- RemoteSigned 允许运行从互联网下载并由受信任发布者签名的脚本和配置文件，同样会让机器容易受到来自这些受信任脚本的恶意攻击。
- Unrestricted 将运行从互联网下载的所有脚本和配置文件，只要您确认了该文件是从互联网下载的。在这种情况下，不需要数字签名，因此可能会打开机器运行未签名和潜在恶意的从互联网下载的脚本的风险。

在本教程中，您将使用 RemoteSigned 执行策略为当前用户设置权限。这将允许 PowerShell 接受受信任的脚本，而不会使权限变得像 Unrestricted 那样宽泛。在 PowerShell 中输入以下内容：

```
bashCopy code
Set-ExecutionPolicy -Scope CurrentUser
```

PowerShell 将提示您提供执行策略。输入以下内容以使用 RemoteSigned：

```
bashCopy code
RemoteSigned
```

按 ENTER 后，您将被要求确认更改执行策略。键入字母 y 以允许更改生效。您可以通过请求在整个计算机上查看当前权限来确认这一点：

```
bashCopy code
Get-ExecutionPolicy -List
```

您应该会收到类似于以下的输出：

```
mathematicaCopy code
Output
        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser    RemoteSigned
 LocalMachine       Undefined
```

这证实了当前用户可以运行从互联网下载的受信任脚本。您现在可以继续下载设置 Go 编程环境所需的文件。

# 步骤2 —— 安装 Chocolatey 包管理器

包管理器是一组软件工具，用于自动化安装过程。这包括初始安装、升级和配置软件以及根据需要删除软件。它们将软件安装在一个中心位置，并可以以常用格式维护系统上的所有软件包。

Chocolatey 是专为 Windows 构建的命令行包管理器，类似于 Linux 上的 apt-get。在开源版本中可用，Chocolatey 将帮助您快速安装应用程序和工具。您将使用它来下载您的开发环境所需的内容。

在安装脚本之前，请阅读脚本以确认您是否满意它对您的计算机所做更改。为此，请使用 .NET 脚本框架在终端窗口中下载并显示 Chocolatey 脚本。

首先，创建一个名为 `$script` 的 WebClient 对象，该对象与 Internet Explorer 共享互联网连接设置：

```
bashCopy code
$script = New-Object Net.WebClient
```

通过将 `$script` 对象与 `Get-Member` 类一起传递，查看可用选项：

```
bashCopy code
$script | Get-Member
```

这将返回该 WebClient 对象的所有成员（属性和方法）：

```
csharpCopy code
. . .
[secondary_label Snippet of Output]
DownloadFileAsync         Method     void DownloadFileAsync(uri address, string fileName), void DownloadFileAsync(ur...
DownloadFileTaskAsync     Method     System.Threading.Tasks.Task DownloadFileTaskAsync(string address, string fileNa...
DownloadString            Method     string DownloadString(string address), string DownloadString(uri address) #method we will use
DownloadStringAsync       Method     void DownloadStringAsync(uri address), void DownloadStringAsync(uri address, Sy...
DownloadStringTaskAsync   Method     System.Threading.Tasks.Task[string] DownloadStringTaskAsync(string address), Sy…
. . .
```

检查输出后，您可以找到用于在 PowerShell 窗口中显示脚本和签名的 `DownloadString` 方法。使用此方法检查脚本：

```
bashCopy code
$script.DownloadString("https://chocolatey.org/install.ps1")
```

检查脚本后，通过在 PowerShell 中键入以下内容来安装 Chocolatey：

```
bashCopy code
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
```

`iwr` 或 `Invoke-WebRequest` cmdlet 允许您从 Web 中提取数据。这将将脚本传递给 `iex`，或 `Invoke-Expression` cmdlet，它将执行脚本内容并运行 Chocolatey 包管理器的安装。

允许 PowerShell 安装 Chocolatey。一旦完全安装，您可以使用 `choco` 命令开始安装其他工具。

如果将来需要升级 Chocolatey，请运行以下命令：

```
bashCopy code
choco upgrade chocolatey
```

安装了包管理器，您可以安装 Go 编程环境所需的其余内容。

## 步骤3 —— 安装文本编辑器 Nano（可选）

在此步骤中，您将安装 Nano，一种使用命令行界面的文本编辑器。您可以使用 Nano 直接在 PowerShell 中编写程序。这不是强制性的步骤，因为您也可以使用带有图形用户界面的文本编辑器，如 Notepad。本教程建议使用 Nano，因为它将帮助您习惯使用 PowerShell。

使用 Chocolatey 安装 Nano：

```
bashCopy code
choco install -y nano
```

`-y` 标志会自动确认您要在不提示确认的情况下运行脚本。

安装了 Nano 后，您可以使用 `nano` 命令创建新的文本文件。您将在本教程后面的步骤中使用它来编写您的第一个 Go 程序。

# 步骤4 —— 安装 Go

与上一步类似，在此步骤中，您将使用 Chocolatey 安装 Go：

```
bashCopy code
choco install -y golang
```

注意：由于 go 是一个非常小的单词，因此在搜索互联网上与 Go 相关的文章时，通常使用 golang 这个术语。Golang 一词来源于 Go 的域名，即 golang.org。

PowerShell 将会安装 Go，并在此过程中在 PowerShell 中生成输出。一旦安装完成，您应该会看到以下输出：

```
bashCopy code
Output
Environment Vars (like PATH) have changed. Close/reopen your shell to
see the changes (or in powershell/cmd.exe just type `refreshenv`).
The install of golang was successful.
 Software installed as 'msi', install location is likely default.

Chocolatey installed 1/1 packages.
See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).
```

安装完成后，您可以关闭并重新打开 PowerShell 作为管理员，然后检查本地计算机上可用的 Go 版本：

```
bashCopy code
go version
```

您将收到类似以下的输出：

```
bashCopy code
Output
go version go1.12.1 windows/amd643.7.0
```

一旦安装了 Go，您可以为开发项目设置一个工作区。

## 步骤5 —— 创建您的 Go 工作区

现在您已经安装了 Chocolatey、Nano 和 Go，可以创建您的编程工作区了。

Go 工作区将在其根目录包含两个目录：

- src：包含 Go 源文件的目录。源文件是使用 Go 编程语言编写的文件。这些源文件由 Go 编译器用于创建可执行的二进制文件。
- bin：包含由 Go 工具构建和安装的可执行文件的目录。可执行文件是在系统上运行并执行任务的二进制文件。这通常是由源代码编译的程序或另一个下载的 Go 源代码编译的程序。

src子目录可能包含多个版本控制存储库（如Git、Mercurial和Bazaar）。当您的程序导入第三方库时，您将看到诸如github.com或golang.org的目录。如果您使用像github.com这样的代码存储库，您还将在该目录下放置您的项目和源文件。这允许在项目中进行规范的代码导入。规范的导入是指引用完全合格的包的导入，例如github.com/digitalocean/godo。

这是一个典型工作区的样子：

```
pythonCopy code
.
├── bin
│   ├── buffalo                                      # 可执行命令
│   ├── dlv                                          # 可执行命令
│   └── packr                                        # 可执行命令
└── src
    └── github.com
        └── digitalocean
            └── godo
                ├── .git                            # Git 存储库元数据
                ├── account.go                      # 包源代码
                ├── account_test.go                 # 测试源代码
                ├── ...
                ├── timestamp.go
                ├── timestamp_test.go
                └── util
                    ├── droplet.go
                    └── droplet_test.go
```

截至1.8版本，Go的默认工作区目录是您用户主目录下的go子目录，或者$HOME/go。如果您使用的Go版本早于1.8，仍然建议使用$HOME/go位置作为您的工作区。

执行以下命令以导航到$HOME目录：

```
bashCopy code
cd $HOME
```

接下来，为Go工作区创建目录结构：

```
bashCopy code
mkdir go/bin, go/src
```

这将确保现在有以下目录结构：

```
bashCopy code
└── $HOME
    └── go
        ├── bin
        └── src
```

在Go 1.8之前，需要设置一个名为$GOPATH的本地环境变量。虽然不再明确需要这样做，但仍然被认为是一种良好的实践，因为许多第三方工具仍然依赖于该变量的设置。

由于您使用Chocolatey进行安装，这个环境变量应该已经设置。您可以使用以下命令验证：

```
rubyCopy code
$env:GOPATH
```

您应该看到以下输出，其中您的用户名将替代'sammy'：

```
goCopy code
Output
C:\Users\sammy\go
```

当Go编译和安装工具时，它们将放在$GOPATH/bin目录中。为了方便起见，通常将工作区的bin子目录添加到您的$PATH中。您可以使用PowerShell中的setx命令执行此操作：

```
bashCopy code
setx PATH "$($env:path);$GOPATH\bin"
```

现在，您可以在系统的任何位置运行通过Go工具编译或下载的任何程序。

既然您已经创建了工作区的根目录并设置了$GOPATH环境变量，您将使用以下目录结构创建将来的项目。此示例假定您将github.com用作存储库：

```
bashCopy code
$GOPATH/src/github.com/username/project
```

如果您正在使用https://github.com/digitalocean/godo项目，您将其放置在以下目录：

```
bashCopy code
$GOPATH/src/github.com/digitalocean/godo
```

以这种方式组织项目将使项目能够通过go get工具使用。它还将有助于后期的可读性。

您可以使用go get命令验证这一点，以获取godo库：

```
arduinoCopy code
go get github.com/digitalocean/godo
```

注意：如果您尚未安装git，Windows将打开一个对话框询问是否要安装它。点击是以继续并按照安装说明操作。

通过列出目录，您可以看到它成功下载了godo包：

```
rubyCopy code
ls $env:GOPATH/src/github.com/digitalocean/godo
```

您将收到类似于以下内容的输出：

```
luaCopy code
Output
    Directory: C:\Users\sammy\go\src\github.com\digitalocean\godo

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/10/2019   2:59 PM                util
-a----        4/10/2019   2:59 PM              9 .gitignore
-a----        4/10/2019   2:59 PM             69 .travis.yml
-a----        4/10/2019   2:59 PM           1592 account.go
-a----        4/10/2019   2:59 PM           1679 account_test.go
-rw-r--r--  1 sammy  staff   2892 Apr  5 15:56 CHANGELOG.md
-rw-r--r--  1 sammy  staff   1851 Apr  5 15:56 CONTRIBUTING.md
.
.
.
-a----        4/10/2019   2:59 PM           5076 vpcs.go
-a----        4/10/2019   2:59 PM           4309 vpcs_test.go
```

在这一步中，您创建了一个Go工作区并配置了必要的环境变量。在下一步中，您将使用一些代码测试工作区。

**第6步 — 创建一个简单的程序**

现在您已经设置了Go工作区，请创建一个简单的“Hello, World！”程序。这将确保您的工作区已正确配置，并且还为您熟悉Go提供了机会。因为您正在创建一个单一的Go源文件，而不是一个实际的项目，所以您不需要在您的工作区中执行此操作。

从您的主目录开始，打开命令行文本编辑器，如nano，并创建一个新文件：

```
goCopy code
nano hello.go
```

一旦文本文件在nano中打开，键入您的程序：

**hello.go**

```
goCopy code
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

通过按下CTRL和X键退出nano。在提示是否保存文件时，按Y，然后按ENTER。