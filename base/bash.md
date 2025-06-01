# 1. bash-handbook-zh-CN [![CC 4.0][cc-image]][cc-url]

谨以此文档献给那些想学习Bash又无需钻研太深的人。

> **Tip**: 不妨尝试 [**learnyoubash**](https://git.io/learnyoubash) — 一个基于本文档的交互式学习平台！

# 2. 前言

如果你是一个程序员，时间的价值想必心中有数。持续优化工作流是你最重要的工作之一。

在通往高效和高生产力的路上，我们经常不得不做一些重复的劳动，比如：

- 对屏幕截图，并把截图上传到服务器上
- 处理各种各种的文本
- 在不同格式之间转换文件
- 格式化一个程序的输出

就让**Bash**来拯救我们吧。

Bash是一个Unix Shell，作为[Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell)的free software替代品，由[Brian Fox][]为GNU项目编写。它发布于1989年，在很长一段时间，Linux系统和macOS系统都把Bash作为默认的shell。

[Brian Fox]: https://en.wikipedia.org/wiki/Brian_Fox_(computer_programmer)
<!-- 一些MD处理器不能很好的处理URL里面的 '()'，因此这个链接采用这种格式 -->

那么，我们学习这个有着30多年历史的东西意义何在呢？答案很简单：这是当今最强大、可移植性最好的，为所有基于Unix的系统编写高效率脚本的工具之一。这就是你需要学习bash的原因。

在本文中，我会用例子来描述在bash中最重要的思想。希望这篇概略性的文章对你有帮助。

# 3. Shells与模式

bash shell有交互和非交互两种模式。

## 3.1. 交互模式

Ubuntu用户都知道，在Ubuntu中有7个虚拟终端。
桌面环境接管了第7个虚拟终端，于是按下`Ctrl-Alt-F7`，可以进入一个操作友好的图形用户界面。

即便如此，还是可以通过`Ctrl-Alt-F1`，来打开shell。打开后，熟悉的图形用户界面消失了，一个虚拟终端展现出来。

看到形如下面的东西，说明shell处于交互模式下：

    user@host:~$

接着，便可以输入一系列Unix命令，比如`ls`，`grep`，`cd`，`mkdir`，`rm`，来看它们的执行结果。

之所以把这种模式叫做交互式，是因为shell直接与用户交互。

直接使用虚拟终端其实并不是很方便。设想一下，当你想编辑一个文档，与此同时又想执行另一个命令，这种情况下，下面的虚拟终端模拟器可能更适合：

- [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal)
- [Terminator](https://en.wikipedia.org/wiki/Terminator_(terminal_emulator))
- [iTerm2](https://en.wikipedia.org/wiki/ITerm2)
- [ConEmu](https://en.wikipedia.org/wiki/ConEmu)

## 3.2. 非交互模式

- 在非交互模式下，shell从文件或者管道中读取命令并执行。
  - 当shell解释器执行完文件中的最后一个命令，shell进程终止，并回到父进程。

- 可以使用下面的命令让shell以非交互模式运行：

  ```bash
  sh /path/to/script.sh
  bash /path/to/script.sh
  ```

  - 上面的例子中，`script.sh`是一个包含shell解释器可以识别并执行的命令的普通文本文件
  - `sh`和`bash`是shell解释器程序。
  - 你可以使用任何喜欢的编辑器创建`script.sh`（vim，nano，Sublime Text, Atom等等）。

- 注意：
  - 大多数`/bin/sh`都是指向`/bin/bash`
  - 但是 **如果以sh命令启动bash，bash将模拟sh的行为**
  - 也就是`bash --posix`

- 除此之外，你还可以通过`chmod`命令给文件添加可执行的权限，来直接执行脚本文件：

  ```bash
  chmod +x /path/to/script.sh
  ```

- 这种方式要求脚本文件的第一行必须指明运行该脚本的程序，依旧是 **Shebang行** 。
  - `#!`与脚本解释器之间有没有空格，都是可以的。

  ```bash
  #!/bin/bash
  echo "Hello, world!"
  ```

  - 如果你更喜欢用`sh`，把`#!/bin/bash`改成`#!/bin/sh`即可。`#!`被称作[shebang](https://zh.wikipedia.org/wiki/Shebang)。之后，就能这样来运行脚本了：

    ```bash
    /path/to/script.sh
    ```

  - Shebang 行不是必需的，但是建议加上这行。
    - 如果缺少该行，就需要手动将脚本传给解释器。

  - 还可以这样来使用shebang：

    ```bash
    #!/usr/bin/env bash
    echo "Hello, world!"
    ```
    - 上面命令使用env命令（这个命令总是在/usr/bin目录），返回 Bash 可执行文件的位置。
    - 这样做的好处是，系统会自动在`PATH`环境变量中查找你指定的程序（本例中的`bash`）
    - 相比第一种写法，你应该尽量用这种写法，因为程序的路径是不确定的。
    - 这样写还有一个好处，操作系统的`PATH`变量有可能被配置为指向程序的另一个版本。
    - 比如，安装完新版本的`bash`，我们可能将其路径添加到`PATH`中，来“隐藏”老版本。
    - 如果直接用`#!/bin/bash`，那么系统会选择老版本的`bash`来执行脚本，如果用`#!/usr/bin/env bash`，则会使用新版本。

## 3.3. 返回值

- 说明：
  - 每个命令都有一个**返回值**（**返回状态**或者**退出状态**）
  - 命令执行成功的返回值总是`0`（零值）
  - 执行失败的命令，返回一个非0值（错误码）
    - 错误码必须是一个1到255之间的整数。

- `exit`:在编写脚本时，另一个很有用的命令是`exit`
  - 这个命令被用来终止当前的执行，并把返回值交给shell
  - 当`exit`不带任何参数时，它会终止当前脚本的执行并返回在它之前最后一个执行的命令的返回值。

- `$?`:
  - 一个程序运行结束后，shell将其**返回值**赋值给`$?`环境变量
  - 因此`$?`变量通常被用来检测一个脚本执行成功与否。

- `return`:
  - 与使用`exit`来结束一个脚本的执行类似
  - 可以使用`return`命令来结束一个函数的执行并将**返回值**返回给调用者
  - 当然，也可以在函数内部用`exit`，这 _不但_ 会中止函数的继续执行， _而且_ 会终止整个程序的执行。

# 4. bash基本说明

## 4.1. bash 命令种类

- `type` 命令来 check

builtin，alias，command，function

## 4.2. 通知信息

- /etc/issue: 在 login 提示符之前显示，比如tty中
- /etc/motd: 在用户成功登录系统之后显示

## 4.3. 启动环境与配置文件

用户每次使用 Shell，都会开启一个与 Shell 的 Session（对话）。

Session 有两种类型：登录 Session 和非登录 Session，也可以叫做 login shell 和 non-login shell。

### 4.3.1. 登录 Session (login shell)

登录 Session 是用户登录系统以后，系统为用户开启的原始 Session，通常需要用户输入用户名和密码进行登录。

登录 Session 一般进行整个系统环境的初始化，启动的初始化脚本依次如下。

- `/etc/profile`：所有用户的全局配置脚本。
- `/etc/profile.d`目录里面所有`.sh`文件
- 用户个人配置初始化，优先级从搞到底，只执行一个
  - `~/.bash_profile`：用户的个人配置脚本。如果该脚本存在，则执行完就不再往下执行。
  - `~/.bash_login`：如果`~/.bash_profile`没找到，则尝试执行这个脚本（C shell 的初始化脚本）。如果该脚本存在，则执行完就不再往下执行。
  - `~/.profile`：如果`~/.bash_profile`和`~/.bash_login`都没找到，则尝试读取这个脚本（Bourne shell 和 Korn shell 的初始化脚本）。

Linux 发行版更新的时候，会更新`/etc`里面的文件，比如`/etc/profile`，因此不要直接修改这个文件。如果想修改所有用户的登陆环境，就在`/etc/profile.d`目录里面新建`.sh`脚本。

如果想修改你个人的登录环境，一般是写在`~/.bash_profile`里面。下面是一个典型的`.bash_profile`文件。

```
# .bash_profile
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/bin
PATH=$PATH:$HOME/bin

SHELL=/bin/bash
MANPATH=/usr/man:/usr/X11/man
EDITOR=/usr/bin/vi
PS1='\h:\w\$ '
PS2='> '

if [ -f ~/.bashrc ]; then
. ~/.bashrc
fi

export PATH
export EDITOR
```

可以看到，这个脚本定义了一些最基本的环境变量，然后执行了`~/.bashrc`。

`bash`命令的`--login`参数，会强制执行登录 Session 会执行的脚本。

```
$ bash --login
```

`bash`命令的`--noprofile`参数，会跳过上面这些 Profile 脚本。

```
$ bash --noprofile
```

### 4.3.2. 非登陆session (non-login shell)

非登录 Session 是用户进入系统以后，手动新建的 Session，这时不会进行环境初始化。比如，

- 在命令行执行`bash`命令，就会新建一个非登录 Session。
- 从图形界面的窗口管理器登录并不会产生登录 Shell。

非登录 Session 的初始化脚本依次如下。

- `/etc/bash.bashrc`：对全体用户有效。
- `~/.bashrc`：仅对当前用户有效。

对用户来说，`~/.bashrc`通常是最重要的脚本。非登录 Session 默认会执行它，而登录 Session 一般也会通过调用执行它。
每次新建一个 Bash 窗口，就相当于新建一个非登录 Session，所以`~/.bashrc`每次都会执行。
注意， **执行脚本相当于新建一个非互动的 Bash 环境，但是这种情况不会调用`~/.bashrc`** 。

`bash`命令的`--norc`参数，可以禁止在非登录 Session 执行`~/.bashrc`脚本。

```
$ bash --norc
```

`bash`命令的`--rcfile`参数，指定另一个脚本代替`.bashrc`。

```
$ bash --rcfile testrc
```

### 4.3.3. .bash_logout

~/.bash_logout脚本在每次退出 Session 时执行，通常用来做一些清理工作和记录工作，比如删除临时文件，记录用户在本次 Session 花费的时间。

如果没有退出时要执行的命令，这个文件也可以不存在。

## 4.4. PS1 变量

## 4.5. 类型声明 declare / typeset

- `-a` ：将后面名为 variable 的变量定义成为 array 类型
- `-i` ：将后面名为 variable 的变量定义成为整数数字 （integer） 类型
  > bash 环境中的数值运算，默认最多仅能到达整数形态，所以 1/3 结果是 0；

  ```bash
  # 此时，sum为101 而非 字符串 “100+1”
  declare -i sum=100+1
  # 等价于
  sum=$((100+1))
  ```
- `-x` ：用法与 export 一样，就是将后面的 variable 变成环境变量；
- `-r` ：将变量设置成为 readonly 类型，该变量不可被更改内容，也不能 unset

# 5. bash history

## 5.1. 基本说明

- Shell启动时，会从`HISTFILE`（默认是`~/.bash_history`）里读取命令的历史记录。
  - 如果设置了`HISTFILESIZE`变量，那么只会读取`HISTFILESIZE`条历史记录。
  - 如果不想保存历史信息，可以unset shell的`history` option，（`set +o history`），或者unset `HISTFILE`变量。
- Shell退出时，这个shell的最后`HISTSIZE`条历史记录会写到`HISTFILE`里。
  - 如果shell打开了`histappend`选项，会以append的方式回写`HISTFILE`文件， 否则回写时会覆盖`HISTFILE`。
  - 如果保存时历史记录超过了`HISTFILESIZE`，那么`HISTFILE`会被截断（truncated）。
  - 如果不想被截断，可以考虑unset `HISTFILESIZE`，或者把它设成null、非数字的值或者负数值。

在图形界面下，简单地点击shell窗口的关闭按钮可能不会保存这个shell的历史记录。通过执行`exit`命令来退出shell。

可以通过`CTRL-p`和`CTRL-n`来上下浏览历史记录，通过`CTRL-r`反向搜索历史记录。

## 5.2. 环境变量

- `HISTFILE`
  - 历史记录所在的文件。默认是`~/.bash_history`。
  - 不想保存历史信息时，可以unset这个变量。
- `HISTFILESIZE`
  - 设置`HISTFILE`文件里可以保存多少条记录，例如`export HISTFILESIZE=10000`。
- `HISTSIZE`
  - 设置一个shell session可以存多少条历史记录。
- `HISTIGNORE`
  - 保存历史时忽略某些命令。Pattern之间用`:`分隔。
  - 例如`export HISTIGNORE="&:[ ]*:exit"`，会忽略三种类型的历史记录；
    - 分别是： `&`，和上一条历史记录相同的记录；
    - `[ ]*`，以空格开头的历史记录；
    - `exit`，忽略`exit`命令。
- `HISTCONTROL`，
  - 和`HISTIGNORE`类似，也是用来忽略某些历史记录，见`man bash`。
- `HISTCMD`
  - 下一个命令在历史记录中的index。假设当前`HISTCMD`的值是123，然后执行`ls`，再执行`history`，会看到：
- `HISTTIMEFORMAT`
  - 可以在`history`的输出中显示历史记录的时间戳。`HISTTIMEFORMAT`字符串的可用格式见`man strftime`。

## 5.3. 相关bash option

- `history`
  - 可以控制是否保留历史记录。
  - 打开这个选项，`set -o history`；关闭这个选项，`set +o history`。

- `histappend`
  - 打开后，写`HISTFILE`时会用append的方式；否则，使用覆盖的方式。
  - `shopt -s histappend`，打开；`shopt -u histappend`，关闭。
  - append的时候只会把这个shell启动后新增的历史记录写到`HISTFILE`里。
  - 如果`histappend`选项处于关闭状态（默认行为），那么`HISTFILE`只会保留 最后关闭的那个shell的历史（其它shell的历史都被覆盖了）。

- `histreedit`
  - 打开这个option后，如果替换（例如`^xxx^yyy^`）失败，会失败的替换重新输出到shell的输入行上。

- `histverify`
  - 打开后，在做history expansion时会不立即执行它，而是把expansion的结果输出到shell上，让用户有机会在执行前修改它。
  - 比如，打开这个option后，输入`!!`不会立即执行上一个命令，而是把上一个命令打印到shell的输入行上。

（注：可以看到一些选项用`set`来设置，另外一些用`shopt`来设置。这两个都是bash的bulitin command，它们所控制选项有所不同。详见`man bash`。）

## 5.4. History Expansion

通过history expansion，可以重新执行之前执行过的命令或者提取之前执行过的命令的参数。

History Expansion有两种类型，event Designators以及word Designators。 Event designators用来重新执行命令，而word designators用来提取历史命令的参数。

### 5.4.1. event Designators

Event designators以`!`开头，也有一个是以`^`开头的。

```
$ !!        # 执行上一个命令，和!-1等效
$ !-1       # 执行上一个命令
$ !-2       # 执行上上个命令
$ !3        # 执行编号为3的历史记录
$ !xyz      # 执行以xyz开头的最近执行过的命令
$ !?xyz?    # 执行含有xyz的最近执行过的命令
$ ^xxx^yyy^ # 找到最近执行的带xxx的命令，把xxx替换成yyy，然后执行它
```

### 5.4.2. Word Designators

Word designators跟在一个event designators后面，它们之间以`:`分隔。

```
$ echo !!:2   # 提取上一个命令的第2个参数，然后把它传给echo并执行
$ !!:2        # 提取上一个命令的第2个参数，并执行它。如果不想执行，要打开bash的histverify选项
              # 或者用"p" modifier
$ echo !!:0   # 提取上一个命令的名字
$ echo !!:3-5 # 提取第3，4，5个参数
$ echo !!:$   # 提取最后一个参数
$ echo !!:^   # 提取第一个参数
$ echo !!:*   # 提取所有参数，和!!:1-$效果一样
$ echo !!:3*  # 等效于3-$
$ echo !!:3-  # 提取第3个到倒数第二个
```

### 5.4.3. Modifier

Word designators后面还可以跟着一个modifier，它可以进一步修改word designators提取出的字符串。

```
$ echo !!:$:h   # 从末尾开始删除直到遇到/，如果!!:$是个path string，那么"h"相当于只保留dirname
$ echo !!:$:t   # 和"h"相反，保留basename
$ echo !!:$:r   # 去掉后缀，比如abc.xml会变成abc
$ echo !!:$:r:r # 去掉两个后缀，比如abc.gz.tar会变成abc。这个例子也说明了modifier可以连续使用
$ echo !!:$:e   # 只保留后缀，比如abc.xml会变成.xml
$ echo !!:$:gs/abc/xyz/ # 把字符串中的abc替换成xyz，如果不加"g"，则只会替换第一个abc
$ echo !!:$:&   # 重复上一次"s"替换
$ echo !!:*:q   # 给字符串加上单引号；对于单引号内的字符串，bash不会做变量替换
$ echo !!:*:x   # 和"q"一样，只不过如果字符串带有空格，那么会break word。
                # 假如字符串是"a b c"，那么"q"的结果是'a b c'，"x"的结果是'a' 'b' 'c'
$ !!:$:p        # man bash里说：“Print the new command but do not execute it”。
# "p"的一个例子
$ find ./abc -name pom.xml
$ !!:0:p ./xyz !!:2-$
```

注： `!!:$`可以简写成`!$`，也就是`!!:`可以简写成`!`。

## 5.5. 实时更新history

```
export PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"
```

应用了这个`PROMPT_COMMAND`后，每个shell的历史记录都会“实时地”在shell间共享。

- `history -a`会把当前shell的命令append到`HISTFILE`里；
- 然后通过`history -c`清除这个shell的所有历史记录；
- 最后通过`history -r`把`HISTFILE`的内容读取到当前shell里。
- 这个`PROMPT_COMMAND`会在每次执行命令后都会把`HISTFILE`重新加载一遍， 如果`HISTFILE`很大可能会产生一些延迟。
  - （这个`PROMPT_COMMAND`在append时没有性能问题，因为每次append只是append一条记录。）

如果不要求“实时”共享，可以通过在一个shell里执行`history -a`，然后在另一个shell里执行`history -r`的方式来分享一个shell的历史记录。

另外，在shell script里`history`命令默认是禁用的。可以通过下面的方式打开，

# 6. 注释

脚本中可以包含 _注释_。注释是特殊的语句，会被`shell`解释器忽略。它们以`#`开头，到行尾结束。

一个例子：

  ```bash
  #!/bin/bash
  # This script will print your username.
  whoami

  # 多行注释,:和‘间有空格
  : '
  This is a
  multi line
  comment
  '
  ```

> **Tip**: 用注释来说明你的脚本是干什么的，以及 _为什么_ 这样写。

# 7. 变量

- 跟许多程序设计语言一样，你可以在bash中创建变量。
- Bash中没有数据类型。  **变量只能包含数字或者由一个或多个字符组成的字符串**
- 你可以创建三种变量：
  - 局部变量
  - 环境变量
  - 作为 _位置参数_ 的变量

## 7.1. 局部变量

- 说明：
  - 有脚本内局部变量和函数内局部变量
  - **局部变量** 是 **仅在某个脚本内部有效** 的变量
  - 它们不能被其他的程序和脚本访问。

- 声明：
  - 局部变量可以用`=`声明
    - 作为一种约定，变量名`=`变量的值之间 **不应该** 有空格
  - 其值可以用`$`访问到

- 举个例子：

  ```bash
  username="denysdovhan"  # 声明变量
  echo $username          # 输出变量的值
  unset username          # 删除变量
  ```

- 可以用`local`关键字声明属于某个函数的局部变量
  - 这样声明的变量会在函数结束时消失。

  ```bash
  local local_var="I'm a local value"
  ```

## 7.2. 环境变量

- **环境变量**
  - 是对当前shell会话内所有的程序或脚本都可见的变量
- 声明：
  - 创建它们跟创建局部变量类似
  - 但使用的是`export`关键字。

  ```bash
  export GLOBAL_VAR="I'm a global variable"
  ```

- bash中有 _非常多_ 的环境变量
  - 会非常频繁地遇到它们，这里有一张速查表，记录了在实践中最常见的环境变量。

  | Variable     | Description                                                   |
  | :----------- | :------------------------------------------------------------ |
  | `$HOME`      | 当前用户的用户目录                                              |
  | `$PATH`      | 用分号分隔的目录列表，shell会到这些目录中查找命令                 |
  | `$PWD`       | 当前工作目录                                                   |
  | `$RANDOM`    | 0到32767之间的整数                                             |
  | `$UID`       | 数值类型，当前用户的用户ID                                      |
  | `$PS1`       | 主要系统输入提示符                                              |
  | `$PS2`       | 次要系统输入提示符                                              |

  [这里](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html#sect_03_02_04)有一张更全面的Bash环境变量列表。

## 7.3. 位置参数

- **位置参数** 是在调用一个函数并传给它参数时创建的变量。
  > 下表列出了在函数中，位置参数变量和一些其它的特殊变量以及它们的意义。

  | Parameter      | Description                    |
  | :------------- | :----------------------------- |
  | `$0`           | 脚本名称                       |
  | `$1 … $9`      | 第 1 个到第 9 个参数列表       |
  | `${10} … ${N}` | 第 10 个到 N 个参数列表        |
  | `$*` or `$@`   | 除了`$0`外的所有位置参数       |
  | `$#`           | 不包括`$0`在内的位置参数的个数 |
  | `$FUNCNAME`    | 函数名称（仅在函数内部有值）   |

  - 在下面的例子中，位置参数为：`$0='./script.sh'`，`$1='foo'`，`$2='bar'`：

    ```bash
    ./script.sh foo bar
    ```

- `shift`:
  - 会对参数进行左移，即删除`$#`中的第一个参数

## 7.4. 特殊变量

- `$0`	当前脚本的文件名
- `$n`	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是`$1`，第二个参数是`$2`。
- `$#`	传递给脚本或函数的参数个数。
- `$*`	传递给脚本或函数的所有参数。
  - 当被双引号`" "`包含时，`"$*"` 会将所有的参数作为一个整体，以`"$1 $2 … $n"`的形式输出所有参数；
- `$@`	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 `$*` 稍有不同。(类似于arr[*] 和 arr[@] 区别)
  - 当被双引号`" "`包含时，`"$@"` 会将各个参数分开，以`"$1" "$2" … "$n"` 的形式输出所有参数。
- `$?`	上个命令的退出状态，或函数的返回值。
- `$$`	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
- `$_`  上一个命令的最后一个参数

  ```bash
  # decode url
  function urldecode() {
    # : 冒号是无操作
    : "${*//+/ }"
    # $_ 会取到上一个命令的最后一个参数
    echo -e "${_//%/\\x}"
  }
  ```

## 7.5. 默认值

- 变量可以有 _默认_ 值。我们可以用如下语法来指定默认值：

  ```bash
  # 如果变量为空，赋给他们默认值
  : ${VAR:='default'}
  : ${$1:='first'}
  # 或者
  FOO=${FOO:-'default'}
  ```

# 8. Shell扩展

## 8.1. 说明

- 基本说明：
  - Shell 接收到用户输入的命令以后， **会根据空格将用户的输入，拆分成一个个词元（token）** 。
  - 然后，Shell 会扩展词元里面的 **特殊字符** ，扩展完成后才会调用相应的命令。
  - 这种特殊字符的扩展，称为模式扩展（globbing）。
  - 其中有些用到通配符，又称为通配符扩展（wildcard expansion）

- Bash 扩展
  - 大括号扩展 brace expansion
  - 波浪线扩展 tilde expansion
  - 参数和变量扩展 parameter and variable expansion
  - 命令扩展 command substitution
  - 算数扩展 arithmetic expansion
  - 分词 word splitting
  - 路径名扩展 pathname expansion.
    - `? 字符扩展`
    - `* 字符扩展`
    - `方括号扩展`
  - 进程替换 process substitution (需系统支持)

- Bash 是先进行扩展，再执行命令。
  - 因此，扩展的结果是由 Bash 负责的，与所要执行的命令无关。
  - 命令本身并不存在参数扩展，收到什么参数就原样执行。

- 扩展资料

  ```
  模块扩展的英文单词是globbing，这个词来自于早期的 Unix 系统有一个/etc/glob文件，保存扩展的模板。
  后来 Bash 内置了这个功能，但是这个名字就保留了下来。
  模式扩展与正则表达式的关系是，模式扩展早于正则表达式出现，可以看作是原始的正则表达式。
  它的功能没有正则那么强大灵活，但是优点是简单和方便。

  Bash 允许用户关闭扩展。
  $ set -o noglob
  # 或者
  $ set -f
  下面的命令可以重新打开扩展。
  $ set +o noglob
  # 或者
  $ set +f
  ```


## 8.2. IFS

internal field separator，默认为`<space><tab><newline>`

分词时使用，将扩展出来的内容划分为 words。一些命令也使用IFS来切分，比如read

---

```bash
printf "$IFS" | od -b
```

```bash
# 扩展会在命令执行前完成
IFS=":" echo $(echo "$IFS") # 依旧是原始值
IFS=":" echo $(echo "a:b:c") # 依旧是 a:b:c

# 打印 a b c
IFS=":"
echo $(echo "a:b:c")
```

```bash
# 默认值
IFS=$' \t\n'
```

```bash
IFS=ab read x y z
1a2b3
```

## 8.3. 扩展

### 8.3.1. 波浪线扩展

- 波浪线`~`会自动扩展成当前用户的主目录。

  ```bash
  $ echo ~
  /home/me
  ```

- `~/dir`表示扩展成主目录的某个子目录，`dir`是主目录里面的一个子目录名。

  ```bash
  # 进入 /home/me/foo 目录
  $ cd ~/foo
  ```

- `~user`表示扩展成用户`user`的主目录。

  ```bash
  $ echo ~foo
  /home/foo

  $ echo ~root
  /root
  ```

- 如果`~user`的`user`是不存在的用户名，则波浪号扩展不起作用。

  ```bash
  $ echo ~nonExistedUser
  ~nonExistedUser
  ```

- `~+`会扩展成当前所在的目录，等同于`pwd`命令。

  ```bash
  $ cd ~/foo
  $ echo ~+
  /home/me/foo
  ```

- `~-`会扩展为`OLDPWD`，也就是`cd -`时的目录

### 8.3.2. 文件名扩展

#### 8.3.2.1. `?` 字符扩展

- `?`字符代表文件路径里面的 **任意单个字符，不包括空字符** 。
  > 比如，`Data???`匹配所有`Data`后面跟着三个字符的文件名。

  ```bash
  # 存在文件 a.txt 和 b.txt
  $ ls ?.txt
  a.txt b.txt
  ```

  - 上面命令中，`?`表示单个字符，所以会同时匹配`a.txt`和`b.txt`。

- 如果匹配多个字符，就需要多个`?`连用。

  ```bash
  # 存在文件 a.txt、b.txt 和 ab.txt
  $ ls ??.txt
  ab.txt
  ```

  - 上面命令中，`??`匹配了两个字符。

- 注意：`?` 字符扩展属于 **文件名扩展** ， **只有文件确实存在的前提下，才会发生扩展** 。如果文件不存在，扩展就不会发生。

  ```bash
  # 当前目录有 a.txt 文件
  $ echo ?.txt
  a.txt

  # 当前目录为空目录
  $ echo ?.txt
  ?.txt
  ```

  - 上面例子中，如果`?.txt`可以扩展成文件名，`echo`命令会输出扩展后的结果；
  - 如果不能扩展成文件名，`echo`就会原样输出`?.txt`

#### 8.3.2.2. `*` 字符扩展

- `*`字符代表文件路径里面的任意数量的任意字符，包括零个字符。

  ```bash
  # 存在文件 a.txt、b.txt 和 ab.txt
  $ ls *.txt
  a.txt b.txt ab.txt
  ```

  - 上面例子中，`*.txt`代表后缀名为`.txt`的所有文件。

- 如果想输出当前目录的所有文件，直接用`*`即可。

  ```bash
  $ ls *
  ```

- `*` **可以匹配空字符** ，下面是一个例子。

  ```bash
  # 存在文件 a.txt、b.txt 和 ab.txt
  $ ls a*.txt
  a.txt ab.txt

  $ ls *b*
  b.txt ab.txt
  ```

- 注意，`*` **不会匹配隐藏文件** （以`.`开头的文件），即`ls *`不会输出隐藏文件。
  - 如果要匹配隐藏文件，需要写成`.*`。

  ```bash
  # 显示所有隐藏文件
  $ echo .*
  ```

  - 如果要匹配隐藏文件，同时要排除`.`和`..`这两个特殊的隐藏文件，可以与方括号扩展结合使用，写成`.[!.]*`。

    ```bash
    $ echo .[!.]*
    ```

- 注意，`*`字符扩展属于文件名扩展， **只有文件确实存在的前提下才会扩展。如果文件不存在，就会原样输出** 。

  ```bash
  # 当前目录不存在 c 开头的文件
  $ echo c*.txt
  c*.txt
  ```

  - 上面例子中，当前目录里面没有`c`开头的文件，导致`c*.txt`会原样输出。

- `*`只匹配当前目录，不会匹配子目录。

  ```bash
  # 子目录有一个 a.txt
  # 无效的写法
  $ ls *.txt

  # 有效的写法
  $ ls */*.txt
  ```

  - 上面的例子，文本文件在子目录，`*.txt`不会产生匹配，必须写成`*/*.txt`。有几层子目录，就必须写几层星号。

- Bash 4.0 引入了一个参数`globstar`，当该参数打开时，允许`**`匹配零个或多个子目录
  - 因此，`**/*.txt`可以匹配顶层的文本文件和任意深度子目录的文本文件。
  - 详细介绍请看后面`shopt`命令的介绍。

#### 8.3.2.3. 方括号扩展

- 方括号扩展的形式是`[...]`， **只有文件确实存在的前提下才会扩展** 。
  - 如果文件不存在，就会原样输出。
  - 括号之中的任意一个字符。比如，`[aeiou]`可以匹配五个元音字母中的任意一个。

  ```bash
  # 存在文件 a.txt 和 b.txt
  $ ls [ab].txt
  a.txt b.txt

  # 只存在文件 a.txt
  $ ls [ab].txt
  a.txt
  ```

  - 上面例子中，`[ab]`可以匹配`a`或`b`，前提是确实存在相应的文件。

- 方括号扩展属于 **文件名匹配** ，即扩展后的结果必须符合现有的文件路径。
  - 如果不存在匹配，就会保持原样，不进行扩展。

  ```bash
  # 不存在文件 a.txt 和 b.txt
  $ ls [ab].txt
  ls: 无法访问'[ab].txt': 没有那个文件或目录
  ```

  - 上面例子中，由于扩展后的文件不存在，`[ab].txt`就原样输出了，导致`ls`命名报错。

- 方括号扩展还有两种变体：`[^...]`和`[!...]`。
  - 它们表示匹配不在方括号里面的字符，这两种写法是等价的。
  - 比如，`[^abc]`或`[!abc]`表示匹配除了`a`、`b`、`c`以外的字符。

  ```bash
  # 存在 aaa、bbb、aba 三个文件
  $ ls ?[!a]?
  aba bbb
  ```

  - 上面命令中，`[!a]`表示文件名第二个字符不是`a`的文件名，所以返回了`aba`和`bbb`两个文件。
  - 注意，如果需要匹配`[`字符，可以放在方括号内，比如`[[aeiou]`。如果需要匹配连字号`-`，只能放在方括号内部的开头或结尾，比如`[-aeiou]`或`[aeiou-]`。

---

- 方括号扩展有一个简写形式`[start-end]`，表示匹配一个连续的范围。
  - 比如，`[a-c]`等同于`[abc]`，`[0-9]`匹配`[0123456789]`。

  ```bash
  # 存在文件 a.txt、b.txt 和 c.txt
  $ ls [a-c].txt
  a.txt
  b.txt
  c.txt

  # 存在文件 report1.txt、report2.txt 和 report3.txt
  $ ls report[0-9].txt
  report1.txt
  report2.txt
  report3.txt
  ...
  ```

- 下面是一些常用简写的例子。
  - `[a-z]`：所有小写字母。
  - `[a-zA-Z]`：所有小写字母与大写字母。
  - `[a-zA-Z0-9]`：所有小写字母、大写字母与数字。
  - `[abc]*`：所有以`a`、`b`、`c`字符之一开头的文件名。
  - `program.[co]`：文件`program.c`与文件`program.o`。
  - `BACKUP.[0-9][0-9][0-9]`：所有以`BACKUP.`开头，后面是三个数字的文件名。

- 这种简写形式有一个否定形式`[!start-end]`，表示匹配不属于这个范围的字符。
  - 比如，`[!a-zA-Z]`表示匹配非英文字母的字符。

  ```bash
  $ ls report[!1–3].txt
  report4.txt report5.txt
  ```

  - 上面代码中，`[!1-3]`表示排除 1、2 和 3。

### 8.3.3. 变量扩展

- Bash 将美元符号`$`开头的词元视为变量，将其扩展成变量值，详见《Bash 变量》一章。

  ```bash
  $ echo $SHELL
  /bin/bash
  ```

- 变量名除了放在美元符号后面，也可以放在`${}`里面。

  ```bash
  $ echo ${SHELL}
  /bin/bash
  ```

- `${!string*}`或`${!string@}`返回所有匹配给定字符串`string`的变量名。

  ```bash
  $ echo ${!S*}
  SECONDS SHELL SHELLOPTS SHLVL SSH_AGENT_PID SSH_AUTH_SOCK
  ```

- 上面例子中，`${!S*}`扩展成所有以`S`开头的变量名。

### 8.3.4. 大括号扩展

- 大括号扩展`{...}`表示分别扩展成大括号里面的所有值，各个值之间使用逗号分隔。
  - 比如，`{1,2,3}`扩展成`1 2 3`。

   ```bash
   $ echo {1,2,3}
   1 2 3

   $ echo d{a,e,i,u,o}g
   dag deg dig dug dog

   $ echo Front-{A,B,C}-Back
   Front-A-Back Front-B-Back Front-C-Back
   ```

- 注意， **大括号扩展不是文件名扩展** 。 **它会扩展成所有给定的值，而不管是否有对应的文件存在** 。

  ```bash
  $ ls {a,b,c}.txt
  ls: 无法访问'a.txt': 没有那个文件或目录
  ls: 无法访问'b.txt': 没有那个文件或目录
  ls: 无法访问'c.txt': 没有那个文件或目录
  ```

  - 上面例子中，即使不存在对应的文件，`{a,b,c}`依然扩展成三个文件名，导致`ls`命令报了三个错误。

- 另一个需要注意的地方是， **大括号内部的逗号前后不能有空格** 。否则，大括号扩展会失效。

  ```bash
  $ echo {1 , 2}
  {1 , 2}
  ```

  - 上面例子中，逗号前后有空格，Bash 就会认为这不是大括号扩展，而是三个独立的参数。

- **逗号前面可以没有值** ，表示扩展的第一项为空。

  ```bash
  $ cp a.log{,.bak}

  # 等同于
  # cp a.log a.log.bak
  ```

- 大括号 **可以嵌套** 。

  ```bash
  $ echo {j{p,pe}g,png}
  jpg jpeg png

  $ echo a{A{1,2},B{3,4}}b
  aA1b aA2b aB3b aB4b
  ```

- 大括号也 **可以与其他模式联用，并且总是先于其他模式进行扩展**

  ```bash
  $ echo /bin/{cat,b*}
  /bin/cat /bin/b2sum /bin/base32 /bin/base64 ... ...

  # 基本等同于
  $ echo /bin/cat;echo /bin/b*
  ```

  - 上面例子中，会先进行大括号扩展，然后进行`*`扩展，等同于执行两条`echo`命令。

- 大括号 **可以用于多字符的模式** ，方括号不行（只能匹配单字符）。

  ```bash
  $ echo {cat,dog}
  cat dog
  ```

- 由于大括号扩展`{...}`不是文件名扩展，所以它总是会扩展的。
  - 这与方括号扩展`[...]`完全不同，如果匹配的文件不存在，方括号就不会扩展。
  - 这一点要注意区分。

  ```bash
  # 不存在 a.txt 和 b.txt
  $ echo [ab].txt
  [ab].txt

  $ echo {a,b}.txt
  a.txt b.txt
  ```

  - 上面例子中，如果不存在`a.txt`和`b.txt`，那么`[ab].txt`就会变成一个普通的文件名，而`{a,b}.txt`可以照样扩展。

### 8.3.5. 子命令置换

- 命令置换允许我们对一个命令求值，并将其值置换到另一个命令或者变量赋值表达式中
- 当一个命令被`反引号` 或`$()`包围时，命令置换将会执行
- 举个例子：

  ```bash
  now=`date +%T`
  # or
  now=$(date +%T)

  echo $now # 19:08:26
  ```

- `$(...)`可以嵌套，比如`$(ls $(pwd))`。
- `$(< file)` 与 `$(cat file)` 作用相同，不过前者会快些，因为不需要调用任何程序

### 8.3.6. 算数扩展

- 在bash中，执行算数运算是非常方便的。算数表达式必须包在`$(( ))`中。算数扩展的格式为：

  ```bash
  result=$(( ((10 + 5*3) - 7) / 2 ))
  echo $result # 9
  ```

- 在算数表达式中，使用变量无需带上`$`前缀：

  ```bash
  x=4
  y=7
  echo $(( x + y ))     # 11
  echo $(( ++x + y++ )) # 12
  echo $(( x + y ))     # 13
  ```

### 8.3.7. 进程替换扩展(Process Substitution)

- 说明
  - 假设有一个工具，它原本接受的参数应该是一个指代某个具体文档的"文件名"，使用Process Substitution后，可以用其他命令的输出来作为文件的内容，让这个工具去处理

- 注意：
  - 此外， Process Substitution 并不能和文件完全等价。
  - Process Substitution 所建立的"对象"，是不能进行写入和随机读取操作的。
  - 不能写入的话应该很好理解，因为如果进行写操作，将会写到那个临时的文件描述符里面去，而这个临时文件描述符会被迅速地释放掉，
  - 而且由于创建的这个"文件"——姑且这么称呼，不是一个 regular file (与之相对的是 special file)，如果在写入时有严格的检查，甚至连写入都会被拒绝；
  - 有时候我们需要对文件进行随机读取，比如 C 语言里的 fseek() 函数的操作，这样的操作将不能在 Process Substitution 产生的临时对象上正常运作。

- 原理
  - 创建一个 **命名管道(named pipe)**
  - 将命令的输出重定向到 命名管道中
  - 将 process substitution 处替换为 命名管道 路径

  ```bash
  echo <( ls /tmp )
  /dev/fd/63

  ls -lh <( ls /tmp )
  lr-x------ 1 wsain wsain 64 Jan  9 20:03 /dev/fd/63 -> 'pipe:[1983586]'

  ls -lh <( ls /tmp ) <( ls /tmp )
  lr-x------ 1 wsain wsain 64 Jan  9 20:03 /dev/fd/62 -> 'pipe:[1983623]'
  lr-x------ 1 wsain wsain 64 Jan  9 20:03 /dev/fd/63 -> 'pipe:[1983621]'
  ```

### 8.3.8. 分词 word splitting (重要)

- 只有没有在双引号里，才会出现分词
- 只有在出现以下扩展时，才会出现分词。
  - `parameter and variable expansion`
  - `command substitution`
  - `arithmetic expansion`
- 分词依据IFS

---

- 注意：赋值变量的时候，变量中会保存原始值

  ```bash
  # 没有分词(因为赋值变量)
  pid=$(ps -elf | grep worker | grep -v grep | tail -3 | awk '{print $4}')
  # 没有分词(因为双引号)
  ❯ echo "$pid"
  2646806
  2646935
  2649821
  # 下面这里出现分词
  ❯ echo $pid
  2646806 2646935 2649821
  ```

## 8.4. 单引号和双引号

- 单引号和双引号之间有很重要的区别
  - 在双引号中，变量引用或者命令置换是会被展开的
  - 在单引号中是不会的

- 举个例子：
  ```bash
  echo "Your home: $HOME" # Your home: /Users/<username>
  echo 'Your home: $HOME' # Your home: $HOME
  ```

- 当局部变量和环境变量包含空格时，它们在引号中的扩展要格外注意
  - 随便举个例子，假如我们用`echo`来输出用户的输入：

  ```bash
  INPUT="A string  with   strange    whitespace."
  echo $INPUT   # A string with strange whitespace.
  echo "$INPUT" # A string  with   strange    whitespace.
  ```
  - 调用第一个`echo`时给了它5个单独的参数 —— `$INPUT`被分成了单独的词，`echo`在每个词之间打印了一个空格
  - 第二种情况，调用`echo`时只给了它一个参数（整个`$INPUT`的值，包括其中的空格）。

- 来看一个更严肃的例子：

  ```bash
  FILE="Favorite Things.txt"
  cat $FILE   # 尝试输出两个文件: `Favorite` 和 `Things.txt`
  cat "$FILE" # 输出一个文件: `Favorite Things.txt`
  ```
  - 尽管这个问题可以通过把FILE重命名成`Favorite-Things.txt`来解决
  - 但是，假如这个值来自某个环境变量，来自一个位置参数，或者来自其它命令（`find`, `cat`, 等等）呢
  - 因此，如果输入 *可能* 包含空格， **务必要用引号把表达式包起来** 。

# 9. 字符串(Parameter expansions)

## 9.1. 转义

特殊字符需要加反斜杠转义：

```
echo \$date
echo \t
echo "\""

# 由于反斜杠在单引号里面变成了普通字符，所以如果单引号之中，还要使用单引号，不能使用转义，
# 需要在外层的单引号前面加上一个美元符号（$），然后再对里层的单引号转义。
echo $'\''
```

标识不可打印字符：

- `\a`：响铃
- `\b`：退格
- `\n`：换行
- `\r`：回车
- `\t`：制表符

```
echo -e "a\tb"
```

换行符是一个特殊字符，表示命令的结束，Bash 收到这个字符以后，就会对输入的命令进行解释执行。
换行符前面加上反斜杠转义，就使得换行符变成一个普通字符，Bash 会将其当作长度为0的空字符处理，从而可以将一行命令写成多行。

```bash
mv \
/path/to/foo \
/path/to/bar

# 等同于

mv /path/to/foo /path/to/bar
```
上面例子中，如果一条命令过长，就可以在行尾使用反斜杠，将其改写成多行。这是常见的多行命令的写法。



## 9.2. 拼接

- 字符串拼接直接写到一起即可(不需要空格)

  ```bash
  a="ab"
  b="cd"
  c=$a$b
  echo "$c"
  ```

## 9.3. 获取长度

- 获取长度

  ```bash
  string="abcde"
  echo ${#string}
  ```

## 9.4. 字符替换

| Code              | Description         |
| ----------------- | ------------------- |
| `${FOO%suffix}`   | Remove suffix       |
| `${FOO#prefix}`   | Remove prefix       |
| ---               | ---                 |
| `${FOO%%suffix}`  | Remove long suffix  |
| `${FOO##prefix}`  | Remove long prefix  |
| ---               | ---                 |
| `${FOO/from/to}`  | Replace first match |
| `${FOO//from/to}` | Replace all         |
| ---               | ---                 |
| `${FOO/%from/to}` | Replace suffix      |
| `${FOO/#from/to}` | Replace prefix      |

```bash
name="John"
echo ${name/J/j}    #=> "john" (substitution)
echo ${food:-Cake}  #=> $food or "Cake"
```

## 9.5. 提取子字符串

- 语法

  | Expression      | Description                    |
  | --------------- | ------------------------------ |
  | `${FOO:0:3}`    | Substring *(position, length)* |
  | `${FOO:(-3):3}` | Substring from the right       |

- 示例

  ```bash
  name="John"
  echo ${name:0:2}    #=> "Jo" (slicing)
  echo ${name::2}     #=> "Jo" (slicing)
  echo ${name::-1}    #=> "Joh" (slicing)
  echo ${name:(-1)}   #=> "n" (slicing from right)
  echo ${name:(-2):1} #=> "h" (slicing from right)
  ```

## 9.6. 默认值

- 语法

  | Expression           | Description                                              |
  | -------------------- | -------------------------------------------------------- |
  | `${FOO:-val}`        | `$FOO`, or `val` if unset (or null)                      |
  | `${FOO:=val}`        | Set `$FOO` to `val` if unset (or null)                   |
  | `${FOO:+val}`        | `val` if `$FOO` is set (and not null)                    |
  | `${FOO:?message}`    | Show error message and exit if `$FOO` is unset (or null) |

  > Omitting the : removes the (non)nullity checks, e.g. ${FOO-val} expands to val if unset otherwise $FOO.
  >
  > : 移除了 null 检查，${FOO-val} 指的是，如果FOO没有设置值，就取val，如果设置为了null，就取null

- 示例

  | 变量设置方式       | str 没有设置         | str 为空字串       | str 已设置非为空字串 |
  | ------------------ | -------------------- | ------------------ | -------------------- |
  | `var=${str-expr}`  | `var=expr`           | var=               | `var=$str`           |
  | `var=${str:-expr}` | `var=expr`           | var=expr           | `var=$str`           |
  | `var=${str+expr}`  | `var=`               | var=expr           | `var=expr`           |
  | `var=${str:+expr}` | `var=`               | var=               | `var=expr`           |
  | `var=${str=expr}`  | `str=expr var=expr`  | str 不变 var=      | `str 不变 var=$str`  |
  | `var=${str:=expr}` | `str=expr var=expr`  | str=expr var=expr  | `str 不变 var=$str`  |
  | `var=${str?expr}`  | `expr 输出至 stderr` | var=               | `var=$str`           |
  | `var=${str:?expr}` | `expr 输出至 stderr` | expr 输出至 stderr | `var=$str`           |

- 示例

  ```bash
  echo ${food:-Cake}  #=> $food or "Cake"
  ```
## 9.7. 删除与替换

| Expression           | Description                  |
| -------------------- | ---------------------------- |
| `${FOO#pattern}`     | 删除从左往右，最短匹配       |
| `${FOO##pattern}`    | 删除从左往右，最长匹配       |
| `${FOO%pattern}`     | 删除从右往左，最短匹配       |
| `${FOO%%pattern}`    | 删除从右往左，最长匹配       |
| `${FOO/str1/str2}`   | 用str2 替换 str1，只替换一次 |
| `${FOO//str1//str2}` | 把所有str1替换为str2         |

## 9.8. Here 文档 和 Here 字符串

# 10. 数组

跟其它程序设计语言一样，bash中的数组变量给了你引用多个值的能力。在bash中，数组下标也是从0开始，也就是说，第一个元素的下标是0。

跟数组打交道时，要注意一个特殊的环境变量`IFS`。**IFS**，全称 **Input Field Separator**，保存了数组中元素的分隔符。它的默认值是一个空格`IFS=' '`。

## 10.1. 数组声明

- 在bash中，可以通过简单地给数组变量的某个下标赋值来创建一个数组：

  ```bash
  fruits[0]=Apple
  fruits[1]=Pear
  fruits[2]=Plum
  ```

- 数组变量也可以通过复合赋值的方式来创建，比如：

  ```bash
  fruits=(Apple Pear Plum)
  ```
  ```bash
  # 触发扩展
  pids=($(ps -elf | grep worker | awk '{print $4}' | sort | uniq | xargs))
  # 但一般推荐使用 read -a 或者 mapfile 拆分命令输出为数组
  read -a pids <<< "$(ps -elf | grep worker | awk '{print $4}' | sort | uniq | xargs)"
  ```

## 10.2. 数组扩展

- 单个数组元素的扩展跟普通变量的扩展类似：

  ```bash
  echo ${fruits[1]} # Pear
  ```

- 整个数组可以通过把数字下标换成`*`或`@`来扩展：

  ```bash
  echo ${fruits[*]} # Apple Pear Plum
  echo ${fruits[@]} # Apple Pear Plum
  ```

- 上面两行有很重要（也很微妙）的区别，假设某数组元素中包含空格：

  ```bash
  fruits[0]=Apple
  fruits[1]="Desert fig"
  fruits[2]=Plum
  ```

- 为了将数组中每个元素单独一行输出，我们用内建的`printf`命令：

  ```bash
  printf "+ %s\n" ${fruits[*]}
  # 等价于。没有引号
  printf "+ %s\n" Apple Desert fig Plum

  # + Apple
  # + Desert
  # + fig
  # + Plum
  ```

- 为什么`Desert`和`fig`各占了一行？尝试用引号包起来：

  ```bash
  printf "+ %s\n" "${fruits[*]}"
  # 等价于，在最外城加一个引号
  printf "+ %s\n" "Apple Desert fig Plum"

  # + Apple Desert fig Plum
  ```

  - 现在所有的元素都跑去了一行 —— 这不是我们想要的！

- 为了解决这个痛点，`${fruits[@]}`闪亮登场：

  ```bash
  printf "+ %s\n" "${fruits[@]}"
  # 等价于
  printf "+ %s\n" Apple "Desert fig" Plum

  # + Apple
  # + Desert fig
  # + Plum
  ```
  - 在引号内，`${fruits[@]}` **将数组中的每个元素扩展为一个单独的参数**
  - 数组元素中的空格得以保留。
  - 使用arr[@]的时候，不加引号shellchecker会报错

## 10.3. 数组切片

- 除此之外，可以通过 _切片_ 运算符来取出数组中的某一片元素：

  ```bash
  echo ${fruits[@]:0:2} # Apple Desert fig
  ```

  - `${fruits[@]}`扩展为整个数组
  - `:0:2`取出了数组中从0开始，长度为2的元素。

## 10.4. 向数组中添加元素

- 向数组中添加元素也非常简单。复合赋值在这里显得格外有用。我们可以这样做：

  ```bash
  fruits=(Orange "${fruits[@]}" Banana Cherry)
  echo ${fruits[@]} # Orange Apple Desert fig Plum Banana Cherry
  ```

- 上面的例子中，`${fruits[@]}`扩展为整个数组，并被置换到复合赋值语句中，接着，对数组`fruits`的赋值覆盖了它原来的值。

## 10.5. 从数组中删除元素

- 用`unset`命令来从数组中删除一个元素：

  ```bash
  unset fruits[0]
  echo ${fruits[@]} # Apple Desert fig Plum Banana Cherry
  ```

## 10.6. 多行字符串处理

```bash
#! /bin/bash

# mul 为包含换行的文本
read -r -d '' mul <<-_exit
hello world
hello
world
hello world
hello world
_exit

# 使用 分词 切分为数组
IFS=$'\n'
arr1=($mul)
# 会扩展为 arr1=('hello world' 'hello' 'world' 'hello world' 'hello world')
echo ${#arr1[@]}
IFS=$' \n\t'

# 使用 read 切分为数组. `-d ''` 使读到newline时不自动停止
IFS=$'\n' read -r -d '' -a arr2 <<< "$mul"
echo ${#arr2[@]}

# 使用while read
while read line;
do
  echo $line
done < <(echo "${mul}")
```

# 11. 字典

## 11.1. 定义

```bash
declare -A sounds # Declares sound as a Dictionary object (aka associative array).
sounds[dog]="bark"
sounds[cow]="moo"
sounds[bird]="tweet"
sounds[wolf]="howl"
```

## 11.2. 基本使用

```bash
echo ${sounds[dog]} # Dog's sound
echo ${sounds[@]}   # All values
echo ${!sounds[@]}  # All keys
echo ${#sounds[@]}  # Number of elements
unset sounds[dog]   # Delete dog
```

## 11.3. 遍历

### 11.3.1. 遍历所有value

```bash
for val in "${sounds[@]}"; do
  echo $val
done
```

### 11.3.2. 遍历所有key

```bash
for key in "${!sounds[@]}"; do
  echo $key
done
```

# 12. 流，管道以及序列

- Bash有很强大的工具来处理程序之间的协同工作
- 使用流，我们能将一个程序的输出发送到另一个程序或文件，因此，我们能方便地记录日志或做一些其它我们想做的事。
- 管道给了我们创建传送带的机会，控制程序的执行成为可能。
- 学习如何使用这些强大的、高级的工具是非常非常重要的。

## 12.1. 流

- Bash接收输入，并以字符序列或 **字符流** 的形式产生输出。这些流能被重定向到文件或另一个流中。
  > 有三个文件描述符：

  | 代码 |  描述符  | 描述         |
  | :--: | :------: | :----------- |
  | `0`  | `stdin`  | 标准输入     |
  | `1`  | `stdout` | 标准输出     |
  | `2`  | `stderr` | 标准错误输出 |

- 重定向让我们可以控制一个命令的输入来自哪里，输出结果到什么地方
  > 这些运算符在控制流的重定向时会被用到：

  | Operator | Description                                                   |
  | :------: | :------------------------------------------------------------ |
  |   `>`    | 重定向输出                                                    |
  |   `>>`   | 以追加的方式重定向输出                                        |
  |   `&>`   | 重定向输出和错误输出                                          |
  |  `&>>`   | 以追加的形式重定向输出和错误输出                              |
  |   `<`    | 重定向输入                                                    |
  |  `2>&1`  | 错误输出重定向到标准输出                                      |
  |   `<<`   | [Here 文档](http://tldp.org/LDP/abs/html/here-docs.html) 语法 |
  |  `<<-`   | 也是here文档，但是会忽略leading tab                           |
  |  `<<<`   | [Here 字符串](http://www.tldp.org/LDP/abs/html/x17837.html)   |

  > 细分：

  |     Operator      | Description                                      |
  | :---------------: | :----------------------------------------------- |
  | `command > file`  | 将输出以覆盖的方式重定向到 file                  |
  | `command < file`  | 将输入重定向到 file                              |
  | `command >> file` | 将输出以追加的方式重定向到 file                  |
  |    `n > file`     | 将文件描述符 n 重定向到 file                     |
  |    `n >> file`    | 将文件描述符 n 以追加的方式重定向到 file         |
  |     `n >& m`      | 将输出 m 和 n 合并                               |
  |     `n <& m`      | 将输入 m 和 n 合并                               |
  |     `&>file`      | 将标准输出和标准错误输出都重定向 file            |
  |     `<< tag`      | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |

  > 注：/dev/null是一个"黑洞"，任何向他写入的内容都被丢弃，也无法从此文件读取到任何内容

- 以下是一些使用重定向的例子：

  ```bash
  # ls的结果将会被写到list.txt中
  ls -l > list.txt

  # 将输出附加到list.txt中
  ls -a >> list.txt

  # 所有的错误信息会被写到errors.txt中
  grep da * 2> errors.txt

  # 从errors.txt中读取输入
  less < errors.txt
  ```
  ```bash
  # 先输入后输出
  # cat从test.sh 获得输入数据，然后输出给文件catfile
	cat > catfile < test.sh

  # 向cat输入文件结束符，cat输出到catfile，这时不用按ctrl+d即可退出
	cat > catfile << eof
  ```

- 使用流访问网络

  ```bash
  cd /proc/$$/fd
  exec 9<> /dev/tcp/github.com/80 # 设置socket (一切皆文件)
  echo -e "GET / HTTP/1.0 \n" 1>& 9 # 将输出重定向到文件标识符9（socket）
  # -e 识别转义字符
  cat 0<& 9  # 将输入重定向到0
  ```

## 12.2. 管道

- 说明
  - 我们不仅能将流重定向到文件中，还能重定向到其它程序中
  - **管道** 允许我们把一个程序的输出当做另一个程序的输入。

- 示例：
  - 在下面的例子中，`command1`把它的输出发送给了`command2`，然后输出被传递到`command3`：

    ```bash
    command1 | command2 | command3
    ```
  - 这样的结构被称作 **管道**。

- 在实际操作中，这可以用来在多个程序间依次处理数据。
  - 在下面的例子中
  - `ls -l`的输出被发送给了`grep`
  - 来打印出扩展名是`.md`的文件
  - 它的输出最终发送给了`less`：

    ```bash
    ls -l | grep .md$ | less
    ```

- 返回值：
  - 管道的返回值通常是管道中最后一个命令的返回值。
  - shell会等到管道中所有的命令都结束后，才会返回一个值。
  - 如果你想让管道中任意一个命令失败后，管道就宣告失败，那么需要用下面的命令设置pipefail选项：

    ```bash
    set -o pipefail
    ```

- 在使用管道时，较多命令支持使用 `-` 在管道前后来表示前一个指令的 `stdin` 和后一个指令的 `stdout`。

  ```bash
  tar -cvf - /home | tar -xvf - -C /tmp/homeback
  # curl 中有 @filename的语法，如果要读取stdin，就是 @-
  curl --location --request POST 'url'  --data @-
  ```

## 12.3. 命令序列

> 命令序列是由`;`，`&`，`&&`或者`||`运算符分隔的一个或多个管道序列。

- 如果一个命令以`&`结尾，shell将会在一个子shell中异步执行这个命令。 **换句话说，这个命令将会在后台执行** 。
- 以`;`分隔的命令将会依次执行：一个接着一个。shell会等待直到每个命令执行完。

  ```bash
  # command2 会在 command1 之后执行
  command1 ; command2

  # 等同于这种写法
  command1
  command2
  ```

- 以`&&`和`||`分隔的命令分别叫做 _与_ 和 _或_ 序列。
  - _与序列_ 看起来是这样的：

    ```bash
    # 当且仅当command1执行成功（返回0值）时，command2才会执行
    command1 && command2
    ```

  - _或序列_ 是下面这种形式：

    ```bash
    # 当且仅当command1执行失败（返回错误码）时，command2才会执行
    command1 || command2
    ```

  - _与_ 或 _或_ 序列的返回值是序列中最后一个执行的命令的返回值。

# 13. 条件表达式与条件语句

跟其它程序设计语言一样，Bash中的条件语句让我们可以决定一个操作是否被执行。结果取决于一个包在`[[ ]]`里的表达式。

条件表达式可以包含`&&`和`||`运算符，分别对应 _与_ 和 _或_ 。除此之外还有很多有用的[表达式](#基元和组合表达式)。

共有两个不同的条件表达式：`if`和`case`。

## 13.1. 基元和组合表达式

- 说明
  - 由`[[ ]]`（`sh`中是`[ ]`）包起来的表达式被称作 **检测命令** 或 **基元**
  - 这些表达式帮助我们检测一个条件的结果
  - `[ `是一个内建命令，一定要有空格。除了要用` ]`来闭合外，和test命令相同
  - 在下面的表里，为了兼容`sh`，我们用的是`[ ]`
  - 这里可以找到有关[bash中单双中括号区别](http://serverfault.com/a/52050)的答案。
    - 当”[ ]”中使用”-n”或者”-z”这些选项判断变量是否为空时，必须在变量的外侧加上双引号，才更加保险，
    - 使用”[[ ]]”时则不用考虑这样的

- **跟文件系统相关：**

  |         基元          | 含义                                                 |
  | :-------------------: | :--------------------------------------------------- |
  |     `[ -e FILE ]`     | 如果`FILE`存在 (**e**xists)，为真                    |
  |     `[ -f FILE ]`     | 如果`FILE`存在且为一个普通文件（**f**ile），为真     |
  |     `[ -d FILE ]`     | 如果`FILE`存在且为一个目录（**d**irectory），为真    |
  |     `[ -s FILE ]`     | 如果`FILE`存在且非空（**s**ize 大于 0），为真        |
  |     `[ -r FILE ]`     | 如果`FILE`存在且有读权限（**r**eadable），为真       |
  |     `[ -w FILE ]`     | 如果`FILE`存在且有写权限（**w**ritable），为真       |
  |     `[ -x FILE ]`     | 如果`FILE`存在且有可执行权限（e**x**ecutable），为真 |
  |     `[ -L FILE ]`     | 如果`FILE`存在且为一个符号链接（**l**ink），为真     |
  | `[ FILE1 -nt FILE2 ]` | `FILE1`比`FILE2`新（**n**ewer **t**han）             |
  | `[ FILE1 -ot FILE2 ]` | `FILE1`比`FILE2`旧（**o**lder **t**han）             |

- **跟字符串相关：**

|         基元          | 含义                                        |
| :-------------------: | :------------------------------------------ |
|     `[ -z STR ]`      | `STR`为空（长度为 0，**z**ero）             |
|     `[ -n STR ]`      | `STR`非空（长度非 0，**n**on-zero）         |
|  `[ STR1 == STR2 ]`   | `STR1`和`STR2`相等                          |
|  `[ STR1 != STR2 ]`   | `STR1`和`STR2`不等                          |
|  `[ STR1 =~ STR2 ]`   | 前者包含后者(子字符串)                      |
| `[[ STR1 =~ regex ]]` | regex可以从STR1中找到匹配字符串,只有[[ 支持 |
|   `[ STR1 < STR2 ]`   | ASCII 小于                                  |
|   `[ STR1 > STR2 ]`   | ASCII 大于                                  |

- **算数二元运算符：**

  |        基元         | 含义                                                  |
  | :-----------------: | :---------------------------------------------------- |
  | `[ ARG1 -eq ARG2 ]` | `ARG1`和`ARG2`相等（**eq**ual）                       |
  | `[ ARG1 -ne ARG2 ]` | `ARG1`和`ARG2`不等（**n**ot **e**qual）               |
  | `[ ARG1 -lt ARG2 ]` | `ARG1`小于`ARG2`（**l**ess **t**han）                 |
  | `[ ARG1 -le ARG2 ]` | `ARG1`小于等于`ARG2`（**l**ess than or **e**qual）    |
  | `[ ARG1 -gt ARG2 ]` | `ARG1`大于`ARG2`（**g**reater **t**han）              |
  | `[ ARG1 -ge ARG2 ]` | `ARG1`大于等于`ARG2`（**g**reater than or **e**qual） |

- 条件语句可以跟 **组合表达式** 配合使用：

  |      Operation       | Effect                                                  |
  | :------------------: | :------------------------------------------------------ |
  |     `[ ! EXPR ]`     | 如果`EXPR`为假，为真                                    |
  |     `[ (EXPR) ]`     | 返回`EXPR`的值                                          |
  | `[ EXPR1 -a EXPR2 ]` | 逻辑 _与_， 如果`EXPR1`和（**a**nd）`EXPR2`都为真，为真 |
  | `[ EXPR1 -o EXPR2 ]` | 逻辑 _或_， 如果`EXPR1`或（**o**r）`EXPR2`为真，为真    |

- 当然，还有很多有用的基元，在[Bash的man页面](http://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)能很容易找到它们。

## 13.2. 使用`if`

- `if`在使用上跟其它语言相同。
  - 如果中括号里的表达式为真，那么`then`和`fi`之间的代码会被执行
  - `fi`标志着条件代码块的结束。

  ```bash
  # 写成一行
  if [[ 1 -eq 1 ]]; then echo "true"; fi

  # 写成多行
  if [[ 1 -eq 1 ]]; then
    echo "true"
  fi
  ```

- 同样，我们可以使用`if..else`语句，例如：

  ```bash
  # 写成一行
  if [[ 2 -ne 1 ]]; then echo "true"; else echo "false"; fi

  # 写成多行
  if [[ 2 -ne 1 ]]; then
    echo "true"
  else
    echo "false"
  fi
  ```

- 有些时候，`if..else`不能满足我们的要求。别忘了`if..elif..else`，使用起来也很方便。

  ```bash
  if [[ `uname` == "Adam" ]]; then
    echo "Do not eat an apple!"
  elif [[ `uname` == "Eva" ]]; then
    echo "Do not take an apple!"
  else
    echo "Apples are delicious!"
  fi
  ```

## 13.3. 使用`case`

- 如果你需要面对很多情况，分别要采取不同的措施，那么使用`case`会比嵌套的`if`更有用。
  > 使用`case`来解决复杂的条件判断，看起来像下面这样：

  ```bash
  case "$extension" in
    "jpg"|"jpeg")
      echo "It's image with jpeg extension."
    ;;
    "png")
      echo "It's image with png extension."
    ;;
    "gif")
      echo "Oh, it's a giphy!"
    ;;
    *)
      echo "Woops! It's not image!"
    ;;
  esac
  ```

  - 每种情况都是匹配了某个模式的表达式。
  - `|`用来分割多个模式，`)`用来结束一个模式序列。
  - 第一个匹配上的模式对应的命令将会被执行。
  - `*`代表任何不匹配以上给定模式的模式。
  - 命令块儿之间要用`;;`分隔。

- `shift` 命令：
  - 会对参数进行左移，即删除`$#`中的第一个参数
  - 经常与case一起使用遍历参数

  ```bash
  # 示例
  while [ $# -gt 0 ]
  do
    case $1 in
      -h|--help) print_usage; exit;;
      # For options with required arguments, an additional shift is needed.
      -i|--ip) IP="$2" ; shift;;
      -p|--port) PORT="$2" ; shift;;
      -f|--file) FILE="$2" ; shift;;
      -t|--type) TYPE="$2" ; shift;;
      (--) shift; break;;
      (-*) echo "$0: invalid option $1" 1>&2; exit 1;;
      (*) break;;
    esac
    shift
  done
  ```

# 14. 循环

循环其实不足为奇。跟其它程序设计语言一样，bash中的循环也是只要控制条件为真就一直迭代执行的代码块。

Bash中有四种循环：`for`，`while`，`until`和`select`。

## 14.1. `for`循环

- `for`与它在C语言中的循环

  ```bash
  # 不需要括号括起来 elem
  for arg in elem1 elem2 ... elemN
  do
    # 语句
  done
  ```

  - 在每次循环的过程中，`arg`依次被赋值为从`elem1`到`elemN`
  - 这些值还可以是通配符或者[大括号扩展](#大括号扩展)。

- `for .. in ..` 和扩展一起使用

  ```bash
  # 除了 * ，其他类型的扩展也支持
  for file in `path/*`
  do
    echo "${file}"
  done

  for file in ~/{Document,.vim}; do  echo ${file}; done
  # /home/wsain/Document
  # /home/wsain/.vim

  for file in aaa{Document,.vim}bbb; do  echo ${file}; done
  # aaaDocumentbbb
  # aaa.vimbbb
  ```
  ```bash
  for file in $(ls -a)
  do
    echo "${file}"
  done
  ```

- 当然，我们还可以把`for`循环写在一行，但这要求`do`之前要有一个分号，就像下面这样：

  ```bash
  for i in {1..5}; do echo $i; done
  ```

- 如果你觉得`for..in..do`对你来说有点奇怪，那么你也可以像C语言那样使用`for`，比如：

  ```bash
  for (( i = 0; i < 10; i++ )); do
    echo $i
  done
  ```

- 当我们想对一个目录下的所有文件做同样的操作时，`for`就很方便了
  > 举个例子，如果我们想把所有的`.bash`文件移动到`script`文件夹中
  > 并给它们可执行权限，我们的脚本可以这样写：

  ```bash
  #!/bin/bash

  for FILE in $HOME/*.bash; do
    mv "$FILE" "${HOME}/scripts"
    chmod +x "${HOME}/scripts/${FILE}"
  done
  ```

## 14.2. `while`循环

- `while`循环检测一个条件，只要这个条件为 _真_，就执行一段命令
  - 被检测的条件跟`if..then`中使用的[基元](#基元和组合表达式)并无二异
  - 因此一个`while`循环看起来会是这样：

  ```bash
  while [[ condition ]]
  do
    # 语句
  done
  ```

- 跟`for`循环一样，如果我们把`do`和被检测的条件写到一行，那么必须要在`do`之前加一个分号。

  ```bash
  #!/bin/bash

  # 0到9之间每个数的平方
  x=0
  while [[ $x -lt 10 ]]; do # x小于10
    echo $(( x * x ))
    x=$(( x + 1 )) # x加1
  done
  ```

## 14.3. `until`循环

- `until`循环跟`while`循环正好相反
  - 它跟`while`一样也需要检测一个测试条件
  - 但不同的是，只要该条件为 _假_ 就一直执行循环：

  ```bash
  until [[ condition ]]; do
    # 语句
  done
  ```

## 14.4. `select`循环

- `select`循环帮助我们组织一个用户菜单。它的语法几乎跟`for`循环一致：

  ```bash
  select answer in elem1 elem2 ... elemN
  do
    # 语句
  done
  ```

- `select`会打印`elem1..elemN`以及它们的序列号到屏幕上，之后会提示用户输入。
  - 通常看到的是`$?`（`PS3`变量）
  - 用户的选择结果会被保存到`answer`中
  - 如果`answer`是一个在`1..N`之间的数字，那么`语句`会被执行，紧接着会进行下一次迭代
  - 如果不想这样的话我们可以使用`break`语句。

- 一个可能的实例可能会是这样：

  ```bash
  #!/bin/bash

  PS3="Choose the package manager: "
  select ITEM in bower npm gem pip
  do
    echo -n "Enter the package name: " && read PACKAGE
    case $ITEM in
      bower) bower install $PACKAGE ;;
      npm)   npm   install $PACKAGE ;;
      gem)   gem   install $PACKAGE ;;
      pip)   pip   install $PACKAGE ;;
    esac
    break # 避免无限循环
  done
  ```

  - 这个例子，先询问用户他想使用什么包管理器。接着，又询问了想安装什么包，最后执行安装操作。
  - 运行这个脚本，会得到如下输出：

    ```
    $ ./my_script
    1) bower
    2) npm
    3) gem
    4) pip
    Choose the package manager: 2
    Enter the package name: bash-handbook
    <installing bash-handbook>
    ```

## 14.5. 实际情景

### 14.5.1. range

```bash
for i in {1..5}; do # 大括号扩展
    echo "Welcome $i"
done

for i in {5..50..5}; do # 带步长
    echo "Welcome $i"
done
```

### 14.5.2. reading line

```bash
cat file.txt | while read line; do
  echo $line
done
```

```bash
while read -r var;
do
    echo $var | awk '{print $4}'
done< <(echo "a b c d")
```

```bash
# https://unix.stackexchange.com/questions/592511/while-loop-done-command-instead-of-done-file
while read <&3 -r var;
do
    echo $var
done 3< <(echo "a b c d")
```

```bash
# 注意：下面这种写法可以循环遍历lines

while read line;
do
    echo "${line}"
done <<< "${lines}"

# 但下面这种只会循环读第一行
while read line  <<< "${lines}" ;
do
    echo "${line}"
done

# 可以理解成 while 是一个命令，会fork + exec 后面的子命令
# 主要是看重定向输入到while命令，还是到子命令
```

### 14.5.3. list file

```bash
for file in "path"/*; do
  if [ -d "$file" ]; then # $file 是全路径
    # 为文件夹时
  else
    # 为文件时
  fi
done
```

## 14.6. 循环控制

- 我们会遇到想提前结束一个循环或跳过某次循环执行的情况
  - 这些可以使用shell内建的`break`和`continue`语句来实现
  - 它们可以在任何循环中使用。

- 使用
  - `break`语句用来提前结束当前循环。我们之前已经见过它了。
  - `continue`语句用来跳过某次迭代。我们可以这么来用它：

  ```bash
  for (( i = 0; i < 10; i++ )); do
    if [[ $(( i % 2 )) -eq 0 ]]; then continue; fi
    echo $i
  done
  ```

# 15. 函数

## 15.1. 基本说明

- 在脚本中，可以定义并调用函数。
  - 跟其它程序设计语言类似，函数是一个代码块，但有所不同。
  - bash中，函数是一个 **命令序列** ，这个命令序列组织在某个名字下面，即 _函数名_ 。
  - 调用函数跟其它语言一样，写下函数名字，函数就会被 _调用_ 。
  - 必须在调用前声明函数。

  ```bash
  my_func () {
    # 语句
  }

  my_func # 调用 my_func
  ```

- 示例：下面这个函数接收一个名字参数，返回`0`，表示成功执行。

  ```bash
  # 带参数的函数
  greeting () {
    if [[ -n $1 ]]; then
      echo "Hello, $1!"
    else
      echo "Hello, unknown!"
    fi
    return 0
  }

  # greeting Denys  # Hello, Denys!
  # greeting        # Hello, unknown!
  ```

## 15.2. 参数与返回

- **参数和返回值** ：函数可以接收参数并返回结果。
  - 参数，在函数内部
    - 跟 非交互式 下的脚本参数处理方式相同，使用 **位置参数**
  - 返回值可以使用`return`命令，_返回_ 。
    - 之前已经介绍过 **返回值**
    - 函数的返回值和脚本、命令的返回值要求相同，必须为0~255
    - 不带任何参数的`return`会返回最后一个执行的命令的返回值
    - 上面的例子，`return 0`会返回一个成功表示执行的值，`0`。

- 字符串返回。通过函数调用+获取stdout值：

  ```bash
  return_str() {
    echo "value"
  }
  v=$(return_str)
  echo "${v}"
  ```

- 函数式编程：
  - bash脚本中的数据类型本质为字符串
  - 所以可以做到传递函数的效果。

  ```bash
  function get_execute_time() {
    t1=$(date +%s)
    $1
    t2=$(date +%s)
    spend_time=$((t2 - t1))
    echo "$1花费时间: ${spend_time}"
  }
  function fun1() {
    sleep 3
  }

  # 下面调用都可以执行
  get_execute_time fun1
  get_execute_time "sleep 3"
  ```

# 16. 其他

## 16.1. Debugging

### 16.1.1. bash参数设置

- shell提供了用于debugging脚本的工具
  - 如果我们想以debug模式运行某脚本，可以在其shebang中使用一个特殊的选项：

  ```bash
  #!/bin/bash options
  ```

- options是一些可以改变shell行为的选项。下表是一些可能对你有用的选项：

  | Short | Name        | Description                                                |
  | :---: | :---------- | :--------------------------------------------------------- |
  | `-f`  | noglob      | 禁止文件名展开（globbing）                                 |
  | `-i`  | interactive | 让脚本以 _交互_ 模式运行                                   |
  | `-n`  | noexec      | 读取命令，但不执行（语法检查）                             |
  | `-t`  | —           | 执行完第一条命令后退出                                     |
  | `-v`  | verbose     | 在执行每条命令前，向`stderr`输出该命令                     |
  | `-x`  | xtrace      | 在执行每条命令前，向`stderr`输出该命令以及该命令的扩展参数 |
  | `-e`  |             | 脚本只要发生错误，就终止执行。                             |
  | `-u`  |             | 执行脚本时，如果遇到不存在的变量，Bash 默认忽略它。        |

- 举个例子，如果我们在脚本中指定了`-x`例如：

  ```bash
  #!/bin/bash -x

  for (( i = 0; i < 3; i++ )); do
    echo $i
  done
  ```

  - 这会向`stdout`打印出变量的值和一些其它有用的信息：

  ```
  $ ./my_script
  + (( i = 0 ))
  + (( i < 3 ))
  + echo 0
  0
  + (( i++  ))
  + (( i < 3 ))
  + echo 1
  1
  + (( i++  ))
  + (( i < 3 ))
  + echo 2
  2
  + (( i++  ))
  + (( i < 3 ))
  ```

- 有时我们需要debug脚本的一部分。
  - 这种情况下，使用`set`命令会很方便。这个命令可以启用或禁用选项
  - 使用`-`启用选项，`+`禁用选项：

  ```bash
  #!/bin/bash

  echo "xtrace is turned off"
  set -x
  echo "xtrace is enabled"
  set +x
  echo "xtrace is turned off again"
  ```

### 16.1.2. 环境变量

- LINENO: 变量`LINENO`返回它在脚本里面的行号。

  ```bash
  #!/bin/bash

  echo "This is line $LINENO"
  ```

  ```bash
  $ ./test.sh
  This is line 3
  ```

- FUNCNAME
  - 变量`FUNCNAME`返回一个数组，内容是当前的函数调用堆栈。
  - 该数组的0号成员是当前调用的函数，1号成员是调用当前函数的函数，以此类推。

  ```bash
  #!/bin/bash

  function func1()
  {
    echo "func1: FUNCNAME0 is ${FUNCNAME[0]}"
    echo "func1: FUNCNAME1 is ${FUNCNAME[1]}"
    echo "func1: FUNCNAME2 is ${FUNCNAME[2]}"
    func2
  }

  function func2()
  {
    echo "func2: FUNCNAME0 is ${FUNCNAME[0]}"
    echo "func2: FUNCNAME1 is ${FUNCNAME[1]}"
    echo "func2: FUNCNAME2 is ${FUNCNAME[2]}"
  }

  func1
  ```

  ```bash
  $ ./test.sh
  func1: FUNCNAME0 is func1
  func1: FUNCNAME1 is main
  func1: FUNCNAME2 is
  func2: FUNCNAME0 is func2
  func2: FUNCNAME1 is func1
  func2: FUNCNAME2 is main
  ```

- BASH_SOURCE:
  - 变量`BASH_SOURCE`返回一个数组，内容是当前的脚本调用堆栈。
  - 该数组的0号成员是当前执行的脚本，1号成员是调用当前脚本的脚本，以此类推，
  - 跟变量`FUNCNAME`是一一对应关系。

  > 下面有两个子脚本`lib1.sh`和`lib2.sh`。

  ```bash
  # lib1.sh
  function func1()
  {
    echo "func1: BASH_SOURCE0 is ${BASH_SOURCE[0]}"
    echo "func1: BASH_SOURCE1 is ${BASH_SOURCE[1]}"
    echo "func1: BASH_SOURCE2 is ${BASH_SOURCE[2]}"
    func2
  }
  ```

  ```bash
  # lib2.sh
  function func2()
  {
    echo "func2: BASH_SOURCE0 is ${BASH_SOURCE[0]}"
    echo "func2: BASH_SOURCE1 is ${BASH_SOURCE[1]}"
    echo "func2: BASH_SOURCE2 is ${BASH_SOURCE[2]}"
  }
  ```

  > 然后，主脚本`main.sh`调用上面两个子脚本。

  ```bash
  #!/bin/bash
  # main.sh

  source lib1.sh
  source lib2.sh

  func1
  ```

  > 执行主脚本`main.sh`，会得到下面的结果。

  ```bash
  $ ./main.sh
  func1: BASH_SOURCE0 is lib1.sh
  func1: BASH_SOURCE1 is ./main.sh
  func1: BASH_SOURCE2 is
  func2: BASH_SOURCE0 is lib2.sh
  func2: BASH_SOURCE1 is lib1.sh
  func2: BASH_SOURCE2 is ./main.sh
  ```

- BASH_LINENO
  - 变量`BASH_LINENO`返回一个数组，内容是每一轮调用对应的行号。
  - `${BASH_LINENO[$i]}`跟`${FUNCNAME[$i]}`是一一对应关系，表示`${FUNCNAME[$i]}`在调用它的脚本文件`${BASH_SOURCE[$i+1]}`里面的行号。
  - 下面有两个子脚本`lib1.sh`和`lib2.sh`。

  ```bash
  # lib1.sh
  function func1()
  {
    echo "func1: BASH_LINENO is ${BASH_LINENO[0]}"
    echo "func1: FUNCNAME is ${FUNCNAME[0]}"
    echo "func1: BASH_SOURCE is ${BASH_SOURCE[1]}"

    func2
  }
  ```

  ```bash
  # lib2.sh
  function func2()
  {
    echo "func2: BASH_LINENO is ${BASH_LINENO[0]}"
    echo "func2: FUNCNAME is ${FUNCNAME[0]}"
    echo "func2: BASH_SOURCE is ${BASH_SOURCE[1]}"
  }
  ```

  > 然后，主脚本`main.sh`调用上面两个子脚本。

  ```bash
  #!/bin/bash
  # main.sh

  source lib1.sh
  source lib2.sh

  func1
  ```

  > 执行主脚本`main.sh`，会得到下面的结果。

  ```bash
  $ ./main.sh
  func1: BASH_LINENO is 7
  func1: FUNCNAME is func1
  func1: BASH_SOURCE is main.sh
  func2: BASH_LINENO is 8
  func2: FUNCNAME is func2
  func2: BASH_SOURCE is lib1.sh
  ```

  - 上面例子中，
    - 函数`func1`是在`main.sh`的第7行调用
    - 函数`func2`是在`lib1.sh`的第8行调用的。

## 16.2. read

- 基本使用

  ```bash
  read var
  read -t 60 -p "please input your name in 1 min: " user_name
  read -s -p "please input your password:" password
  ```

- 读取数据并分隔为数组

  ```bash
  IFS=',' read -a arr1 <<< "a,b,c,d"
  echo ${#arr1[@]}
  ```

- 赋值多个变量
  ```bash
  IFS=':' read user group password <<< "user1:group1:xxxxx"
  echo "$user $group $password"
  ```

- 读取多行数据
  ```bash
  read -r -d '' multi_lines <<- _exit
  aaa bbb
  ccc ddd
  eeeeeee
  ggg ggg
  _exit

  echo "$multi_lines"
  ```

## 16.3. bash中的括号

> see `man bash` Compound Commands section

- `[[`
- `[`
- `(`
- `((`
- `${ ...; }`

## 16.4. 双括号语法

- 语法：

  ```bash
  ((表达式1,表达式2...))
  ```
- 特点：
	- 在双括号结构中，所有表达式可以像c语言一样，如：a++,b--等。
	- 在双括号结构中，所有变量可以不加入：`“$”`符号前缀。
	- 双括号可以进行逻辑运算，四则运算
	- 双括号结构 扩展了for，while,if条件测试运算
	- 支持多个表达式运算，各个表达式之间用“,”分开
	- 双括号结构之间支持多个表达式，然后加减乘除等c语言常用运算符都支持。
  - 如果双括号带：`$`，将获得表达式值，赋值给左边变量
	- 计算时 `(())` 语法比let、expr更有效率。

- 例

  ```bash
  a=1
  b=2
  c=3
  a=$((a+1,b++,c++))
  echo $a
  #结果为2,3,4
  ```

TODO: bash笔记

## 16.5. 目录堆栈

dirs

## 16.6. set,shopt命令

[set 命令，shopt 命令](https://wangdoc.com/bash/set)

[set, shopt](https://www.gnu.org/software/bash/manual/html_node/Modifying-Shell-Behavior.html)

## 16.7. mktmp, trap命令

## 16.8. getoption命令

## 16.9. 命令提示符

## 16.10. bash 补全原理

`man bash`:

- Programmable Completion
- SHELL BUILTIN COMMANDS
  - complete
  - compgen
  - compopt

<https://jasonkayzk.github.io/2020/12/06/Bash命令自动补全的原理/>

# 17. 后记

因此如果你想了解更多，请运行`man bash`，从那里开始

# 18. 想了解更多？

下面是一些其它有关Bash的资料：

- Bash的man页面。在Bash可以运行的众多环境中，通过运行`man bash`可以借助帮助系统`man`来显示Bash的帮助信息。有关`man`命令的更多信息，请看托管在[The Linux Information Project](http://www.linfo.org/)上的网页["The man Command"](http://www.linfo.org/man.html)。
- ["Bourne-Again SHell manual"](https://www.gnu.org/software/bash/manual/)，有很多可选的格式，包括HTML，Info，Tex，PDF，以及Textinfo。托管在<https://www.gnu.org/>上。截止到2016/01，它基于的是Bash的4.3版本，最后更新日期是2015/02/02。

# 19. 其它资源

- 学习资源
  - [x] **[bash-handbook-zh-CN](https://github.com/liushuaikobe/bash-handbook-zh-CN)**
  - [ ] **[网道 Bash 脚本教程](https://wangdoc.com/bash/)**
    - [ ] [Bash启动环境](https://wangdoc.com/bash/startup)
  - [ ] **[rstacruz/cheatsheets-bash.md](https://github.com/rstacruz/cheatsheets/blob/master/bash.md)**
    > 异常全的bash命令。包括一些字符串处理
  - [ ] [GNU Bash Reference Manual](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html)
    - [译文](https://xy2401.com/local-docs/gnu/manual.zh/bash.html)
  - [ ] [Bash的history命令，实现history实时更新](https://rockhong.github.io/history-in-bash.html)

- 辅助
  - [shellcheck](https://github.com/koalaman/shellcheck)
    > 一个shell脚本的静态分析工具，既可以在网页[www.shellcheck.net](http://www.shellcheck.net/)上使用它，又可以在命令行中使用，安装教程在[koalaman/shellcheck](https://github.com/koalaman/shellcheck)的github仓库页面上
  - [awesome-bash](https://github.com/awesome-lists/awesome-bash)
    > 是一个组织有序的有关Bash脚本以及相关资源的列表
  - [awesome-shell](https://github.com/alebcay/awesome-shell)
    > 另一个组织有序的shell资源列表
  - [bash-it](https://github.com/Bash-it/bash-it)
    > 为你日常使用，开发以及维护shell脚本和自定义命令提供了一个可靠的框架
    >
    > 有时间可以搞搞，跟`oh-my-zsh`差不多
  - [dotfiles.github.io](http://dotfiles.github.io/)
    > 上面有bash和其它shell的各种dotfiles集合以及shell框架的链接
  - [learnyoubash](https://github.com/denysdovhan/learnyoubash)
    > 帮助你编写你的第一个bash脚本

- 其他参考资料:
  - [Why doesn't Bash `(())` work inside `[[]]`?](https://stackoverflow.com/questions/61889505/why-doesnt-bash-work-inside)
  - [Use { ..; } instead of (..) to avoid subshell overhead](https://www.shellcheck.net/wiki/SC2235)
  - [What is the difference between the Bash operators [[ vs [ vs ( vs ((?](https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs)
  - Process Substitution
    - [gnu Process Substitution](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html#Process-Substitution)
    - [What does "< <(command args)" mean in the shell?](https://stackoverflow.com/questions/2443085/what-does-command-args-mean-in-the-shell)
