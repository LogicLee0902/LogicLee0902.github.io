---
layout:     post
title:      "「The Missing Semester of Your CS Education」01 shell的使用"
subtitle:   OS pre 02
date:       2022-02-28
author:     Leo
header-img: "img/post-shell.png"
catalog: true
tags: ["shell@Tags@Tags", "操作系统@Courses@Series", "MIT@Schools@Series"]
lang: zh
header-style: text
katex: true
---

# shell

## shell简介

一种文字接口，允许执行程序，并获取某种半结构化的输出

这边介绍的 是bash(**B**ourne **A**gain **S**helll)

## shell的使用

打开界面的样子

```shell
logic_lee@Leo-MateBook:~$
```

@前的一般是用户名，@后的一般是主机名，`~`表示home，`$`表示当前非root用户，可以在此后面输入命令和需要的参数，而如果参数需要空格的话，一般有两种方法应对

1. 使用单引号 or 双引号将其包裹起来
2. 使用转义符号`\`

```shell
logic_lee@Leo-MateBook:~$ echo Hello\ World
Hello World
logic_lee@Leo-MateBook:~$ echo "Hello World"
Hello World
```

## shell导航

`pwd`可以获取当前工作目录，`.`表示当前目录，`..`上级目录，切换目录用`cd`（都很像）

一般来说，当我们运行一个程序时，如果我们没有指定路径，则该程序会在当前目录下执行。例如，我们常常会搜索文件，并在需要时创建文件。

为了查看指定目录下包含哪些文件，可以使用 `ls` 命令

```shell
logic_lee@Leo-MateBook:/$ ls -l /home
total 0
drwxr-xr-x 1 logic_lee logic_lee 4096 Feb 25 17:29 logic_lee
```

`-l`可以更加详尽的打印出文件或文件夹信息。接下来详细解说一下。

首先，第一个字符`d`表示`logic_lee`是一个目录。接下来的字符3个一组

（`rwx`）. 它们分别代表了文件所有者（`logic_lee`），用户组（`users`） 以及其他所有人具有的权限。其中 `-` 表示该用户不具备相应的权限。因此结合来看，为`rwx`, `r-x`, `r-x`

* 从上面的信息来看，只有文件所有者可以修改（`w`），`logic_lee` 文件夹 （例如，添加或删除文件夹中的文件）。

* 为了进入某个文件夹，用户需要具备该文件夹以及其父文件夹的“搜索”权限(以“可执行”：`x`表示)。

* 为了列出它的包含的内容，用户必须对该文件夹具备读权限（`r`）。

对于文件来说，权限的意义也是类似的。注意，`/bin` 目录下的程序在最后一组，即表示所有人的用户组中，均包含 `x` 权限，也就是说任何人都可以执行这些程序。

```shell
logic_lee@Leo-MateBook:/$ ls -l /bin
lrwxrwxrwx 1 root root 7 Feb 16 08:32 /bin -> usr/bin
```

还有几个命令常用，例如 `mv`（用于重命名或移动文件）、 `cp`（拷贝文件）以及 `mkdir`（新建文件夹）。

如果想要知道关于程序参数、输入输出的信息，亦或是想要了解它们的工作方式，用 `man` 这个程序。它会接受一个程序名作为参数，然后展现它的文档（用户手册）。注意，使用 `q` 可以退出该程序man

[![man.png](https://s4.ax1x.com/2022/02/27/bncxeJ.png)](https://imgtu.com/i/bncxeJ)

## 常用命令介绍

### `mv`

move file，用来为文件或目录改名，或将目录一如到其他文位置

#### 语法

---

```shell
mv [options] source dest
# change the name of source file to the dest file
mv [options] source... directory
# move the source to the directory
# the source can be the source_directory or the soure file
# and if the source is the source_directory and dest directory does'n exist, then change the name of the source directory to the dest
```

#### 参数说明

* -b: 如果存在目标，覆盖前，会创建一个备份
* -i：如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。
* -f:如果移动的源目录存在同名，不会询问，直接覆盖
* -n:不要覆盖任何已存在的文件或目录
* -u：当源文件比目标文件新，或目标文件不存在时再执行

#### 实例

[![buVs76.png](https://s4.ax1x.com/2022/02/28/buVs76.png)](https://imgtu.com/i/buVs76)

### `cp`

主要用于复制文件或目录

#### 语法

```shell
cp [options] source dest
```

#### 参数说明

- -a：此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容。其作用等于dpR参数组合。
- -d：复制时保留链接。这里所说的链接相当于 Windows 系统中的快捷方式。
- -f：覆盖已经存在的目标文件而不给出提示。
- -i：与 -f 选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答 y 时目标文件将被覆盖。
- -p：除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。
- -r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。
- -l：不复制文件，只是生成链接文件。

#### 实例

使用指令 **cp** 将当前目录 **test/** 下的所有文件复制到新目录 **newtest** 下，输入如下命令：

```shell
$ cp –r test/ newtest          
```

注意：用户使用该指令复制目录时，必须使用参数 **-r** 或者 **-R** 。

### `mkdir`

make directory，创建目录

#### 语法

```shell
mkdir [-p] dirName
```

#### 参数说明

* -p 确保目录名称存在，不存在就新建一个（就比如要建立某个文件夹下面的子文件夹时）

#### 实例

在工作目录下，建立一个名为 Leo 的子目录 :

```shell
mkdir Leo
```

在工作目录下的 Leo 目录中，建立一个名为 test 的子目录。

若 Leo 目录原本不存在，则建立一个。（注：本例若不加 -p 参数，且原本 Leo 目录不存在，则产生错误。）

```shell
mkdir -p Leo/test
```

### `sed`

用来对多个文件进行编辑（插入、删除、替换等）

#### 语法

```shell
sed 参数 动作 file
```

#### 参数说明

- -e<script>或--expression=<script> 以选项中指定的script来处理输入的文本文件。
- -f<script文件>或--file=<script文件> 以选项中指定的script文件来处理输入的文本文件。
- -h或--help 显示帮助。
- -n或--quiet或--silent 仅显示script处理后的结果。
- -i 直接修改源文件
- -V或--version 显示版本信息。

(-e+动作命令也可以用" " 或 ' '来省略-e, 单双引号的区别在于含义的替换，在后面会说)

#### 动作说明

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 's/old/new/g' 

其中不同的内容之间分隔符，不同内容比如就是a后面的字符串，s中的新旧字符串等，一般是/, !, :, |等

#### 实例

##### 以行为单位新增/删除

删除 `d`, 新增 `i`, `a`

```shell
#delete lines
sed 'n1,n2d'file
# n1: the begin of the line
# n2: the end of the line
# and the '$' means the last line of the file

# if only one line needed to be delete, then
sed 'nd'file

# Add a new line after the specified one
sed '2a new line' file

# Add a new line ahead of a line
sed '2i new line' file

# if want to add more the one line, use \ to seperate and > to continue in a new line
 sed '4a run \
> what' file

```

##### 以行为单的替换/显示

替换`c`, 显示`p`

```shell
# it's allowed that the total lines of the new line don't need to match the old one
sed '2,5c sin(sin(x)) \
> cos(sin(x))' file

# p to display
sed '3,5p' file
```

##### 数据搜索并替换、删除

删除`d`, 替换`s`

两个都可以用正则表达式搜哦，用`/`或者`;` `|`等隔开

##### 直接修改文件内容

加一个`-i`即可



## 程序间创建连接

在 shell 中，程序有两个主要的“流”：它们的输入流和输出流。 当程序尝试读取信息时，它们会从输入流中进行读取，当程序打印信息时，它们会将信息输出到输出流中。 通常，一个程序的输入输出流都是您的终端。也就是，您的键盘作为输入，显示器作为输出。 **但是，也可以重定向这些流。**

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```shell
logic_lee@Leo-MateBook:~$ echo hello > hello.txt
logic_lee@Leo-MateBook:~$ ls
hello.txt
logic_lee@Leo-MateBook:~$ cat hello.txt
hello
logic_lee@Leo-MateBook:~$ cat < hello.txt > world.txt
logic_lee@Leo-MateBook:~$ ls
hello.txt  world.txt
logic_lee@Leo-MateBook:~$ cat world.txt
hello
```

# 一个工具介绍

对于大多数的类 Unix 系统，有一类用户是非常特殊的，那就是：根用户（root user）。从上面的结果也可以看出来，根用户几乎不受任何限制，他可以创建、读取、更新和删除系统中的任何文件。通常不会用根用户登录系统，因为这样可能会因为某些错误的操作而破坏系统。 取而代之的是，会在需要的时候使用 `sudo` 命令。它的作用是可以 su（super user 或 root 的简写）的身份执行一些操作。 当遇到拒绝访问（permission denied）的错误时，通常是因为此时必须是根用户才能操作。然而，从安全的角度出发，需要再次确认是真的要执行此操作。

有一件事情是作为根用户才能做的，那就是向 `sysfs` 文件写入内容。系统被挂载在 `/sys` 下，`sysfs` 文件则暴露了一些内核（kernel）参数。 因此，不需要借助任何专用的工具，就可以轻松地在运行期间配置系统内核。

**注意 Windows 和 macOS 没有这个文件**

·

# 课后习题

---

没做最后一题5555，懒了

前面的题就无脑按着所说的做就好了，就注意下第5题用要用`>>`追加，用`>`会覆盖

说一下第9题，应该用`grep`命令，其可以在文件中找到所输入的字符串的那一行并输出, 如果没有写明文件名或者用`-`则就在标准输入找。所以利用`|`将`semester`的输出当作`grep`的输入就好了,再将输入定向到txt里面就好了。

```shell
./semester | grep last-modified >~/last-modified.txt
```

