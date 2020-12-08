# Linux环境

## 4.1 程序参数

程序`args.c`对其参数进行检查

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int arg;

    for (arg = 0; arg < argc; arg++)
    {
        if (argv[arg][0] == '-')
            printf("option: %s\n", argv[arg] + 1);
        else
            printf("argument %d: %s\n", arg, argv[arg]);
    }
    exit(0);
}
```

```shell
# ./args -i -lr 'hi there' -f fred.c
argument 0: ./args
option: i
option: lr
argument 3: hi there
option: f
argument 5: fred.c
```

### getopt

```c
#include <unistd.h>
int getopt(int argc, char *const argv[], const char *optstring);
extern char *optarg;
extern int optind, opterr, optopt;
```

`getopt`函数将传递给程序的`main`函数的`argc`和`argv`作为参数，同时接受一个选项指定符字符串`optstring`，该字符串告诉`getopt`哪些选项可用，以及它们是否有关联值。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int opt;
    while ((opt = getopt(argc, argv, ":if:lr")) != -1)	// 字母后面带:表示后面还有参数
    {
        switch (opt)
        {
        case 'i':
        case 'l':
        case 'r':
            printf("option: %c\n", opt);
            break;
        case 'f':
            printf("filename: %s\n", optarg);
            break;
        case ':':
            printf("option needs a value\n");
            break;
        case '?':	// 遇到不符合optstring指定的其他选项 则将全域变量optopt设为"?"
            printf("unknown option: %c\n", optopt);
            break;
        }
    }
    for (; optind < argc; optind++)
        printf("argument: %s\n", argv[optind]);

    exit(0);
}
```

```shell
# ./argopt -i -lr 'hi there' -f fred.c -q
option: i
option: l
option: r
filename: fred.c
unknown option: q
argument: hi there
```

### getopt_long

`GNU C`函数库包含`getopt`的另一个版本，称作`getopt_long`，接受以双划线开始的长参数。

 

## 4.2 环境变量

`C`语言程序可以使用`putenv`和`getenv`函数来访问环境变量

```c
#include <stdlib.h>
char *getenv(const char *name);
char *putenv(const char *string);
```

环境由一组格式为”名字=值“的字符串组成。

- `getenv`函数
  - 以给定的名字搜索环境中的一个字符串，并返回与该名字相关的值。如果请求的变量不存在，它就返回`null`。
  - `getenv`返回的字符串是存储在`getenv`提供的静态空间中，所以如果想进一步使用它，就必须把它复制到另一个字符串中，以免它被后续的`getenv`调用所覆盖
- `putenv`
  - 以一个格式为”名字=值“的字符串作为参数，并将该字符串加到当前环境中。失败返回-1
  - 若失败，错误变量`errno`将被设置为`ENOMEM`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[])
{
    char *var, *value;
    if (argc == 1 || argc > 3)
    {
        fprintf(stderr, "usage: environ var [value]\n");
        exit(1);
    }
    /* 调用getenv从环境中取出变量的值 */
    var = argv[1];
    value = getenv(var);
    if (value)
        printf("Variable %s has value %s\n", var, value);
    else
        printf("Variable %s has no value\n", var);

    /**
     * 检查程序调用时是否有第二个参数 如果有则通过构造一个格式为
     * “名字=值”的字符串并调用putenv来设置变量的值
    */
    if (argc == 3)
    {
        char *string;
        value = argv[2];
        string = malloc(strlen(var) + strlen(value) + 2);
        if (!string)
        {
            fprintf(stderr, "out of memory\n");
            exit(1);
        }
        strcpy(string, var);
        strcat(string, "=");
        strcat(string, value);
        printf("Calling putenv with: %s\n", string);
        if (putenv(string) != 0)
        {
            fprintf(stderr, "putenv failed\n");
            free(string);
            exit(1);
        }
        value = getenv(var);
        if (value)
            printf("New value of %s is %s\n", var, value);
        else
            printf("New value of %s is null??\n", var);
    }
    exit(0);
}
```

```shell
# ./environ HOME
Variable HOME has value /root
# ./environ FRED
Variable FRED has no value
# ./environ FRED hello
Variable FRED has no value
Calling putenv with: FRED=hello
New value of FRED is hello
# ./environ FRED
Variable FRED has no value
```



### environ变量

程序可以通过`environ`变量直接访问这个字符串数组

```c
#include <stdio.h>
#include <stdlib.h>

extern char **environ;	// 以null结尾的字符串数组

int main()
{
    char **env = environ;

    while (*env)
    {
        printf("%s\n", *env);
        env++;
    }
    exit(0);
}
```



## 4.3 时间和日期

时间通过一个预定义的类型`time_t`来处理，与处理时间值的函数一起定义在头文件`time.h`。

可以通过`time`函数得到底层时间值，返回的是从纪元开始至今的秒数。如果`tloc`不是一个空指针，`time`函数还会把返回值写入到`tloc`指针指向的位置。

```c
#include <time.h>
time_t time(time_t *tloc);
```

```c
#include <time.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
// 打印时间 每两秒打印一次
int main()
{
    int i;
    time_t the_time;

    for (i = 1; i <= 10; i++)
    {
        the_time = time((time_t *)0);
        printf("The time is %ld\n", the_time);
        sleep(2);
    }
    exit(0);
}
```



`difftime`函数用于计算两个`time_t`值之间的秒数并以`double`类型返回它

`gmtime`函数把底层时间值分解为一个结构

```c
#include <time.h>
struct tm *gmtime(const time_t timeval);
```

| tm成员      | 说明               |
| ----------- | ------------------ |
| int tm_sec  | 秒，0-61           |
| int tm_min  | 分，0-59           |
| int tm_hour | 小时，0-23         |
| int tm_mday | 月份中的日期，1-31 |
| 未完待续    |                    |

`tm_sec`的范围允许闰秒或双闰秒

利用`tm`结构和`gmtime`函数打印出当前时间和日期。

```c
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    struct tm *tm_ptr;
    time_t the_time;

    (void)time(&the_time);  // 获取底层的时间值
    tm_ptr = gmtime(&the_time); // 将该值转换为一个包含有用的时间和日期值得结构

    printf("Raw time is %ld\n", the_time);
    printf("gmtime gives:\n");
    printf("date: %02d/%02d/%02d\n",
           tm_ptr->tm_year, tm_ptr->tm_mon, tm_ptr->tm_mday);
    printf("time:  %02d:%02d:%02d\n",
           tm_ptr->tm_hour, tm_ptr->tm_min, tm_ptr->tm_sec);

    exit(0);
}
```

`localtime`函数获取当地时间

```c
#include <time.h>
struct tm *localtime(const time_t *timeval);
```

`mktime`函数：将已分解为`tm`结构再再转换为原始的`time_t`时间值

```c
#include <time.h>
time_t mktime(struct tm *timeptr);
```

可以使用`asctime`函数和`ctime`函数更友好的表示时间和日期

```c
#include <time.h>
char *asctime(const struct tm *timeptr);
char *ctime(const time_t *timeval);
```

`ctime`函数用法

```c
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    time_t timeval;
    (void)time(&timeval);
    printf("The date is: %s", ctime(&timeval));
    exit(0);
}
```

`strftime`函数提供对时间和日期的更多控制

```c
#include <time.h>
size_t strftime(char *s, size_t maxsize, const char *format, struct tm *timeptr);
```

`strptime`函数用于读取日期，该函数以一个代表日期和时间的字符串为参数，并创建表示同一日期和时间的`tm`结构，`format`参数同上

```c
#include <time.h>
char *strptime(const char *buf, const char *format, struct tm *timeptr);
```

`strftime`函数和`strptime`函数

```

```

