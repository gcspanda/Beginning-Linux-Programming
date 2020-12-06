# 终端

## 5.1 对终端进行读写

如果想知道标准输出是否被重定向，只需要检查底层文件描述符是否关联到一个终端。系统调用isatty就是用来完成这一任务的。只需要将有效的文件描述符传递给它，即可判断出是否连接到一个终端。如果fd连接到一个终端，则系统调用返回1，否则返回0。

isatty只能对文件描述符进行操作，如果在程序中使用的是文件流，可以与fileno函数结合使用。

```c++
#include <unistd.h>

int isatty(int fd);
```

## 5.2 与终端进行对话



## 5.3 终端驱动程序和通用终端接口



## 5.4 termios结构

termios是在POSIX规范中定义的标准接口，类似于系统V中的termio接口。头文件`termios.h`

可以被调整来影响终端的值按照不同的模式被分成如下几组：

- 输入模式
- 输出模式
- 控制模式
- 本地模式
- 特殊控制模式

```c++
#include <termios.h>
struct termios
{
    tcflag_t c_iflag;
    tcflag_t c_oflag;
    tcflag_t c_cflag;
    tcflag_t c_lflag;
    cc_t	 c_cc[NCCS];
};
```

结构成员的名称与上面列出的5种参数类型相对应。

可以调用函数tcgetattr来初始化一个终端对应的termios结构

```c++
#include <termios.h>

int tcgetattr(int fd, struct termios *termios_p);
```

这个函数调用把当前终端接口变量的值写入termios_p参数指向的结构。如果这些值其后被修改了，你可通过调用函数tcsetattr来重新配置终端接口

```c++
#include <termios.h>

int tcsetattr(int fd, int actions, const struct termios *termios_p);
```

参数actions控制修改方式，共有3种修改方式

- TCSANOW：立即对值进行修改
- TCSADRAIN：等当前的输出完成后再对值进行修改。
- TCSAFLUSH：等当前的输出完成后再对值进行修改，但丢弃还未从read调用返回的当前可用的任何输入。

### 5.4.1 输入模式

输入模式控制输入数据（终端驱动程序从串行口或键盘接收到的字符）在被传递给程序之前的处理方式。通过termios结构中的c_iflag成员的标志对他们进行控制。

用户一般无需频繁修改输入模式，因为通常默认值就是最合适的。

### 5.4.2 输出模式

通过termios结构中的c_oflag成员的标志对他们进行控制。



留个坑





