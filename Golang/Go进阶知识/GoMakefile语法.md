# Go Makefile 语法

Makefile文件由三个部分组成，分别是**Makefile规则**、**Makefile语法**和**Makefile命令**（这些命令可以是Linux命令，也可以是可执行的脚本文件）。在这一讲里，我会介绍下Makefile规则和Makefile语法里的一些核心语法知识。在介绍这些语法知识之前，我们先来看下如何使用Makefile脚本。

## 一、**Makefile的使用方法**

在实际使用过程中，我们一般是先编写一个Makefile文件，指定整个项目的编译规则，然后通过Linux make命令来解析该Makefile文件，实现项目编译、管理的自动化。

默认情况下，make命令会在当前目录下，按照GNUmakefile、makefile、Makefile文件的顺序查找Makefile文件，一旦找到，就开始读取这个文件并执行。

大多数的make都支持“makefile”和“Makefile”这两种文件名，但**我建议使用“Makefile”**。因为这个文件名第一个字符大写，会很明显，容易辨别。make也支持 `-f` 和 `--file` 参数来指定其他文件名，比如 `make -f golang.mk` 或者 `make --file golang.mk` 。

## 二、**Makefile规则介绍**

规则是Makefile中的重要概念，它一般由目标、依赖和命令组成，用来指定源文件编译的先后顺序。Makefile之所以受欢迎，核心原因就是Makefile规则，因为Makefile规则可以自动判断是否需要重新编译某个目标，从而确保目标仅在需要时编译。这一讲我们主要来看Makefile规则里的规则语法、伪目标和order-only依赖。

### 1、**规则语法**

Makefile的规则语法，主要包括target、prerequisites和command，示例如下：

```plain
target ...: prerequisites ...
    command
          ...
          ...
```

* \*\*target，\*\*可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。target可使用通配符，当有多个目标时，目标之间用空格分隔。
* \*\*prerequisites，\*\*代表生成该target所需要的依赖项。当有多个依赖项时，依赖项之间用空格分隔。
* **command**，代表该target要执行的命令（可以是任意的shell命令）。
  * 在执行command之前，默认会先打印出该命令，然后再输出命令的结果；如果不想打印出命令，可在各个command前加上`@`。
  * command可以为多条，也可以分行写，但每行都要以tab键开始。另外，如果后一条命令依赖前一条命令，则这两条命令需要写在同一行，并用分号进行分隔。
  * 如果要忽略命令的出错，需要在各个command之前加上减号`-`。

**只要targets不存在，或prerequisites中有一个以上的文件比targets文件新，那么command所定义的命令就会被执行，从而产生我们需要的文件，或执行我们期望的操作。**

我们直接通过一个例子来理解下Makefile的规则吧。

* 第一步，先编写一个hello.c文件。

```plain
#include <stdio.h>
int main(){
  printf("Hello World!\n");
  return 0;
}
```

* 第二步，在当前目录下，编写Makefile文件。

```plain
hello: hello.o
        gcc -o hello hello.o

hello.o: hello.c
        gcc -c hello.c

clean:
        rm hello.o
```

第三步，执行make，产生可执行文件。

```plain
$ make
gcc -c hello.c
gcc -o hello hello.o
$ ls
hello  hello.c  hello.o  Makefile
```

上面的示例Makefile文件有两个target，分别是hello和hello.o，每个target都指定了构建command。当执行make命令时，发现hello、hello.o文件不存在，就会执行command命令生成target。

* 第四步，不更新任何文件，再次执行make。

```plain
$ make
make: 'hello' is up to date.
```

当target存在，并且prerequisites都不比target新时，不会执行对应的command。

* 第五步，更新hello.c，并再次执行make。

```plain
$ touch hello.c
$ make
gcc -c hello.c
gcc -o hello hello.o
```

当target存在，但 prerequisites 比 target 新时，会重新执行对应的command。

* 第六步，清理编译中间文件。

Makefile一般都会有一个clean伪目标，用来清理编译中间产物，或者对源码目录做一些定制化的清理：

```plain
$ make clean
rm hello.o
```

### 2、伪目标

接下来我们介绍下Makefile中的伪目标。Makefile的管理能力基本上都是通过伪目标来实现的。

在上面的Makefile示例中，我们定义了一个clean目标，这其实是一个伪目标，也就是说我们不会为该目标生成任何文件。因为伪目标不是文件，make 无法生成它的依赖关系，也无法决定是否要执行它。

通常情况下，我们需要显式地标识这个目标为伪目标。在Makefile中可以使用`.PHONY`来标识一个目标为伪目标：

```plain
.PHONY: clean
clean:
    rm hello.o
```

伪目标可以有依赖文件，也可以作为“默认目标”，例如：

```plain
.PHONY: all
all: lint test build
```

因为伪目标总是会被执行，所以其依赖总是会被决议。通过这种方式，可以达到**同时执行所有依赖项**的目的。

### 3、**order-only依赖**

在上面介绍的规则中，只要prerequisites中有任何文件发生改变，就会重新构造target。但是有时候，我们希望\*\*只有当prerequisites中的部分文件改变时，才重新构造target。\*\*这时，你可以通过order-only prerequisites实现。

order-only prerequisites的形式如下：

```plain
targets : normal-prerequisites | order-only-prerequisites
    command
    ...
    ...
```

在上面的规则中，只有第一次构造targets时，才会使用order-only-prerequisites。后面即使order-only-prerequisites发生改变，也不会重新构造targets。

只有normal-prerequisites中的文件发生改变时，才会重新构造targets。这里，符号“ | ”后面的prerequisites就是order-only-prerequisites。

到这里，我们就介绍了Makefile的规则。接下来，我们再来看下Makefile中的一些核心语法知识。


> 更新: 2024-10-03 22:08:50  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/nec3bp4qknc4kstg>