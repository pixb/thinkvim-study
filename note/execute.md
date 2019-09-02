# Execute命令

`execute`命令用来把一个字符串当作Vimscript命令执行。在前面的章节我们曾经跟它打过交道， 不过随着对Vimscript中的字符串有更深入的了解，现在我们将再次认识它。

## `execute`基本用法

执行下面的命令：

```
:execute "echom 'Hello, world!'"
```

Vim把`echom 'Hello, world!'`当作一个命令，而且尽职地在把它输出的同时将消息记录下来。 Execute是一个非常强大的工具，因为它允许你用任意字符串来创造命令。

让我们试试一个更实用的例子。先在Vim里打开一个文件作为准备工作，接着使用`:edit foo.txt`在同一个窗口创建新的缓冲区。 现在执行下面的命令：

```
:execute "rightbelow vsplit " . bufname("#")
```

Vim将在第二个文件的右边打开第一个文件的竖直分割窗口(vertical split)。为什么会这样？

首先，Vim将`"rightbelow vsplit"`和`bufname('#')`调用的结果连接在一起，创建一个字符串作为命令。

我们过一段时间才会讲到相应的函数，现在姑且认为它返回前一个缓冲区的路径名。 你可以用`echom`来确认这一点。

待`bufname`执行完毕，Vim将结果连接成`"rightbelow vsplit bar.txt"`。 `execute`命令将此作为Vimscript命令执行，在新的分割里打开该文件。

## Execute危险吗？

在大多数编程语言中使用诸如"eval"来构造可执行的字符串是会受到谴责的(如果不会是更严重的后果)。 因为两个原因，Vimscript中的`execute`命令能免于操这份心。

首先，大多数Vimscript代码仅仅接受唯一的来源——用户的输入。 假设有用户想输入一个古怪的字符串来执行邪恶的命令，无所谓，反正这是他们自己的计算机！ 然而在其他语言里，程序通常得接受来自不可信的用户的输入。Vim是一个特殊的环境， 在此无需担心一般的安全性问题。

第二个原因是因为Vimscript有时候处理问题的方式过于晦涩难懂且稀奇古怪。 这时`execute`会是完成任务的最简单，最直白的方法。 在大多数其他语言中，使用"eval"不会省下你多少击键的生命，但在Vimscript里这样做可以化繁为简。

## 练习

浏览`:help execute`来明了哪些命令你可以用`execute`实现而哪些不可以。 但当涉猎，因为我们很快将重新审视这个问题。

阅读`:help leftabove`，`:help rightbelow`，`:help :split`和`:help :vsplit`(注意最后两个条目中额外的分号)。

在你的`~/.vimrc`中加入能在选定的分割(竖直或水平，上/下/左/右方位)中打开前一个缓冲区的映射。