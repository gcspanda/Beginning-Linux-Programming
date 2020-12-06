# 开发工具

- make命令和makefile文件
- 使用RCS和CVS系统对源代码进行控制
- 编写手册页
- 使用path和tar命令来发布软件
- 开发环境

## 9.2 make命令和makefile文件

常被用于控制源代码的编译，而且还用于手册页的编写以及将应用程序安装到目标目录。

### 9.2.1 makefile的语法



### 9.2.2 make命令的选项和参数

常用的3个选项：

- `-k`：让make命令在发现错误时仍然继续执行。可以用这个命令在一次操作中发现所有未编译成功的源文件。
- `-n`：让make命令输出将要执行的操作步骤，而不是真正执行这些操作。
- `-f <filename>`：它的作用是告诉make命令将哪个文件作为makefile文件。如果没有使用这个选项，标准版本的make命令会首先在当前目录下查找名为makefile的文件，如果不存在则会查找命名为Makefile的文件。

1.依赖关系

写法规则：先写目标名称，然后紧跟一个冒号，接着是空格或制表符tab，最后是用空格或制表符tab隔开的问及那列表（这些文件用于创建目标文件）

```makefile
myapp: main.o 2.o 3.o
main.o: main.c a .h
2.o: 2.c a.h b.h
3.o: 3.c b.h c.h
```

如果想一次创建多个文件，可以 利用伪目标all。如果没有指定一个all目标，则make命令只创建在makefile中找到的第一个目标。

```makefile
all: myapp myapp.1
```

2.规则

规则开头必须是制表符，而且如果makefile文件中的某行以空格结尾，可能会引起make命令执行失败。

```makefile
myapp: main.o 2.o 3.o
	gcc -o myapp main.o 2.o 3.o
main.o: main.c a.h
	gcc -c main.c
2.o: 2.c a.h b.h
	gcc -c 2.c
3.o: 3.c b.h c.h
	gcc -c 3.c
```

### 9.2.3 文件中的注释

makefile文件中的注释以`#`号开头，一直延续到这一行的结束。

### 9.2.4 makefile文件中的宏

对于管理包含非常多源文件的大型项目来说，makefile显得过于庞大并缺乏弹性。因此，可以使用宏以一种更通用的格式来书写它们。

通过语句`MACRONAME=value`在makefile中定义宏，引用宏的方法是使用`$(MACRONAME)`或`${MACRONMAE}`。如果想把一个宏的值设置为空，你可以令等号后面为空。

```makefile
all: myapp

# Which compiler
CC = gcc

# Where are include files kept
INCLUDE = .

# Options for development
CFLAGS = -g -Wall -ansi

# Options for release
# CFLAGS = -O -Wall -ansi

myapp: main.o 2.o 3.o
	$(CC) -o myapp main.o 2.o 3.o

main.o: main.c a.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c

2.o: 2.c a.h b.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c

3.o: 3.c b.h c.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c
```

make命令将`$(CC)`、 `$(CFLAGS)`和`$(INCLUDE)` 替换为相应的宏定义。

常用的宏

|  宏  |                         定义                         |
| :--: | :--------------------------------------------------: |
|  $?  | 当前目标所依赖的文件列表中比当前目标文件还要新的文件 |
|  $@  |                    当前目标的名字                    |
|  $<  |                  当前依赖文件的名字                  |
|  $*  |           不包括后缀名的当前依赖文件的名字           |

两个特殊字符：

- `-`：告诉make命令忽略所有错误。
- `@`：告诉make在执行某条命令前不要将该命令显示在标准输出上。

### 9.2.5 多个目标

增加clean选项来删除不需要的目标文件，增加一个install选项来将编译成功的应用程序安装到另一个目录下。

rm指令以减号`-`开头，减号的目的是让make命令忽略rm命令的执行结果（例如目标文件不存在导致rm命令返回错误）。

行`clean: `的后面是空的，因此该目标总被认为是过时的，所以在执行make命令时，如果指定目标clean，则该目标所对应的规则将总被执行。

目标install依赖于myapp，所以make命令知道它必须首先创建myapp，然后才能执行制作该目标所需的其他命令。用于制作install目标的规则由几个shell脚本命令组成。由于make命令在执行规则时会调用一个shell，并且会针对每个规则使用一个新shell，所以必须在上面每行代码的结尾加上一个反斜杠`\`，让所有shell脚本命令在逻辑上处于同一行，并作为一个整体传递给一个shell执行。这个命令以符号`@`开头，表示make在执行这些规则之前不会在标准输出上显示命令本身。

```makefile
all: myapp

# Which compiler
CC = gcc

# Where are include files kept
INSTDIR = /usr/local/bin

# Where are include files kept
INCLUDE = .

# Options for development
CFLAGS = -g -Wall -ansi

# Options for release
# CFLAGS = -O -Wall -ansi

myapp: main.o 2.o 3.o
	$(CC) -o myapp main.o 2.o 3.o

main.o: main.c a.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c

2.o: 2.c a.h b.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c

3.o: 3.c b.h c.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c


clean:
	-rm main.o 2.o 3.o

install: myapp
	@if [ -d $(INSTDIR) ]; \
		then \
		cp myapp $(INSTDIR);\
		chmod a+x $(INSTDIR)/myapp;\
		chmod og-w $(INSTDIR)/myapp;\
		echo "Installed in $(INSTDIR)";\
	else \
		echo "Sorry, $(INSTDIR) does not exist";\
	fi
```

### 9.2.6 内置规则



### 9.2.7 后缀和模式规则



### 9.2.8 用make管理函数库

函数库实际上就是文件，通常以`.a`为后缀名，在该文件中包含了一组目标文件。

用于管理函数库的语法是`lib(file.o)`，它的含义是目标文件file.o是存储在函数库lib.a中的。make命令通过一个内置规则来管理函数库，该规则的常见形式如下所示：

```makefile
.c.a:
	$(CC) -c $(CFLAGS) $<
	$(AR) $(ARFLAGS) $@ $*.o
```

宏`$(AR)`和`$(ARFLAGS)`的默认取值通常分别是命令ar和选项rv。这个相当简洁的语法告诉make，想从`.c`文件中得到`.a`库文件，它必须应用上面两条规则。

- 第一条规则告诉它必须编译源文件以生成目标文件。
- 第二条规则告诉它用ar命令将新的目标文件添加到函数库中。

因此，如果有一个名为fud的函数库，其中包含目标文件bas.o，则第一条规则中的`$<`将被替换为bas.c，而第二条规则中的`$@`和`$*`将被分别替换为库文件fud.a和名字bas。

### 9.2.9 高级主题：makefile文件和子目录



### 9.2.10 GNU make和gcc



## 9.3 源代码控制

如果你做的不是一个简单的项目，特别是项目开发者不止一个时，为避免文件修改冲突并跟踪对源文件所做出的修改，对源文件改动方面的管理就变得非常重要。

- SCCS：源代码控制系统
- RCS：版本控制系统
- CVS：并发版本控制系统（比SCCS和RCS更高级的工具）
- Subversion（目的是最终替换CVS）

