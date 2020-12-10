# 数据管理

- 动态内存管理：可以做什么以及`Linux`不允许做什么
- 文件锁定：协调锁、共享文件的锁定区域和避免死锁
- `dbm`数据库：一个大多数`Linux`系统都提供的、基本的、不急于`SQL`的数据库函数库

## 7.1 内存管理

`malloc`函数用来分配内存

```c
#include <stdlib.h>
void *malloc(size_t size);
```

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define A_MEGABYTE (1024 * 1024)

int main()
{
    char *some_memory;
    int megabyte = A_MEGABYTE;
    int exit_code = EXIT_FAILURE;

    some_memory = (char *)malloc(megabyte); // 分配1MB内存
    if (some_memory != NULL)
    {
        sprintf(some_memory, "Hello World\n");
        printf("%s", some_memory);
        exit_code = EXIT_SUCCESS;
    }
    exit(exit_code);
}
```

请求全部的物理内存

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

#define A_MEGABYTE (1024 * 1024)
#define PHY_MEM_MEGS 1024 /* Adjust this number as required */

int main()
{
    char *some_memory;
    size_t size_to_allocate = A_MEGABYTE;
    int megs_obtatined = 0;

    while (megs_obtatined < (PHY_MEM_MEGS * 2))
    {
        some_memory = (char *)malloc(size_to_allocate);
        if (some_memory != NULL)
        {
            megs_obtatined++;
            sprintf(some_memory, "Hello World");
            printf("%s - now allocated %d Megabytes\n", some_memory, megs_obtatined);
        }
        else
        {
            exit(EXIT_FAILURE);
        }
    }
    exit(EXIT_SUCCESS);
}
```

### 滥用内存

### 空指针

### 释放内存

```c
#include <stdlib.h>
void free(void *ptr_to_memory);
```

### 其他内存分配函数

```c
#include <stdlib.h>
void *calloc(size_t number_of_elements, size_t element_size);
void *realloc(void *existing_memory, size_t new_size);
```



## 7.2 文件锁定

- 最简单的方法是以原子操作的方式创建锁文件。
- 第二种方法是允许程序锁定文件的一部分，从而可以独享 这一部分内容的访问。

`Linux`通常会在`/var/spool`目录下创建一个锁文件，注意，锁文件仅仅是充当一个指示器的角色，程序间需要通过相互协作来使用它们。用术语来说，锁文件只是**建议锁**而不是**强制锁**

如果一个程序在执行时只需独占某个资源一段很短的时间（术语来讲叫做**临界区**），它就需要在进入临界区之前使用`open`系统调用创建锁文件，然后退出临界区时用`unlink`系统调用删除 该锁文件。

相关头文件`fcntl.h`

创建锁文件程序：

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>

int main()
{
    int file_desc;
    int save_errno;

    file_desc = open("LCK.test", O_RDWR | O_CREAT | O_EXCL, 0444);
    if (file_desc == -1)
    {
        save_errno = errno;
        printf("Open failed with error %d\n", save_errno);
    }
    else
    {
        printf("Open succeeded\n");
    }
    exit(EXIT_SUCCESS);
}
```

协调性锁文件

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>

const char *lock_file = "LCK.test";

int main()
{
    int file_desc;
    int tries = 10;

    while (tries--)
    {
        file_desc = open(lock_file, O_RDWR | O_CREAT | O_EXCL, 0444);
        if (file_desc == -1)
        {
            printf("%d - Lock already present\n", getpid());
            sleep(3);
        }
        else
        {
            printf("%d - I have exclusive access\n", getpid());
            sleep(1);
            (void)close(file_desc);
            (void)unlink(lock_file);
            sleep(2);
        }
    }
    exit(EXIT_SUCCESS);
}
```



