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

