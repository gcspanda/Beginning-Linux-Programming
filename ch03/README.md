# 文件操作

## 3.1 Linux文件结构

### 3.1.2 文件和设备

`/dev/console`：系统控制台

`/dev/tty`：如果一个进程有控制终端的话，那么这个文件就是特殊控制终端的别名。由系统自动运行的进程和脚本没有控制终端，所以不能打开`/dev/tty`

`/dev/null`：空设备，所有向这个设备的输出都将被丢弃，通常把不需要的输入输出定向到这个文件。

## 3.2 系统调用和设备驱动程序

用于访问设备驱动程序的底层函数（系统调用）

- `open`：打开文件或设备
- `read`：从打开的文件或设备里读数据
- `write`：向文件或设备写数据
- `close`：关闭文件或设备
- `ioctl`：把控制信息传递给设备驱动陈鼓型

## 3.3 库函数

## 3.4 底层文件访问

- 0：标准输入
- 1：标准输出
- 2：标准错误

### 3.4.1 write系统调用

作用：将缓冲区buf的前nbytes个字节写入与文件描述符fildes关联的文件中。返回实际写入的字节数。返回0表示没有写入任何数据，如果返回-1就表示调用出错，错误代码保存在全局变量errno里。

```c++
#include <unistd.h>
size_t write(int fildes, const void *buf, size_t nbytes);
```

### 3.4.2 read系统调用

作用：从与文件描述符fildes相关联的文件里读入nbytes个字节的数据，并把他们放到数据区buf中。返回的实际字节数可能小于请求的字节数。返回0，就表示未读入任何数据，已经到达文件尾，如果返回-1则表示调用出错。

```c++
#include <unistd.h>
size_t read(int fildes, void *buf, size_t nbytes);
```

### 3.4.3 open系统调用

作用：创建新的文件描述符

```c++
#include <fcntul.h>
#include <sys/types.h>
#include <sys/stat.h>

int open(const char *path, int oflags);
int open(const char *path, int oflags, mode_t mode);
```

|   模式   |      说明      |
| :------: | :------------: |
| O_RDONLY | 以只读方式打开 |
| O_WRONLY | 以只写方式打开 |
|  O_RDWR  | 以读写方式打开 |



|  oflags  |                      说明                      |
| :------: | :--------------------------------------------: |
| O_APPEND |            把写入数据追加在文件末尾            |
| O_TRUNC  |        把文件长度设置为零，丢弃已有内容        |
| O_CREATE | 如果需要，就按参数mode中给出的访问模式创建文件 |
|  O_EXCL  |    与O_CREATE一起使用，确保调用者创建出文件    |

### 3.4.4 访问权限的初始值

1. umask：当文件被创建时，为文件的访问权限设定一个掩码。

2. close：终止描述符fildes与其对应文件之间的关联。文件描述符被释放并能够重新使用。调用成功时返回0，出错时返回-1。

```c++
#include <unistd.h>
int close(int fildes)
```

3. ioctl：提供一个用于 控制设备及其描述符行为和配置底层服务的接口。终端、文件描述符、套接字都可以有为它们定义的ioctl。

```c++
#include <unistd.h>
int ioctl(int fildes, int cmd, ...);
```

### 3.4.5 其他与文件管理有关的系统调用

1. lseek：对文件描述符fildes的读写指针进行设置。

```c++
#include <unistd.h>
#include <sys/types/h>

off_t lseek(int fildes, off_t offset, int whence);
```

offset用来指定位置，whence定义该偏移值的用法

2. fstat、stat和lstat：fstat系统调用返回与打开的文件描述符相关的文件的状态信息，该信息将会写到一个buf结构中，buf的地址以参数形式传递给fstat。

```c++
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>

int fstat(int fildes, struct stat *buf);
int stat(const char *path, struct stat *buf);
int lstat(const char *path, struct stat *buf);
```

3. dup和dup2：dup系统调用提供了一种复制文件描述符的方法，使我们能够通过两个或者更多个不同的描述符来访问同一个文件。这可以用于在文件的不同位置对数据进行读写。dup系统调用复制文件描述符fildes，返回一个新的描述符。dup2系统调用则是通过明确指定目标描述符来把一个文件描述符复制为另外一个。

```c++
#include <unistd.h>

int dup(int fildes);
int dup2(int fildes, int fildes2);
```

## 3.5 标准I/O库

### 3.5.1 fopen函数

打开由filename参数指定的文件，并把它与一个文件流关联起来。mode参数指定文件的打开方式。

```c++
#include <stdio.h>
FILE *fopen(const char *filename, const char *mode);
```

成功时返回一个非空的FILE *指针，失败时返回NULL值，NULL值在头文件stdio.h里定义。

### 3.5.2 fread函数

用于从一个文件流里读取数据。数据从文件流stream读到由ptr指向的数据缓冲区里。fread和fwrite都是对数据记录进行操作，size参数指定每个数据记录的长度，计数器nitems给出要传输的记录个数。它的返回值是成功读到数据缓冲区里的记录个数（而不是字节数）。当到达文件尾时，它的返回值可能会小于nitems，甚至可以是零。

```c++
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);
```

### 3.5.3 fwrite函数

```c++
#include <stdio.h>
size_t fread(const void *ptr, size_t size, size_t nitems, FILE *stream);
```

PS：不推荐将fread和fwrite用于结构化数据。部分原因在于用fwrite写的文件在不同的计算机体系结构中之间可能不具备可移植性。

### 3.5.4 fclose函数

关闭指定的文件流stream，使所有尚未写出的数据都写出。

```c++
#include <stdio.h>
int fclose(FILE *stream);
```

### 3.5.5 fflush函数

把文件流里的所有未写出数据立刻写出。使用这个函数还可以来确保在程序继续执行之前重要的数据都已经被写到磁盘上。有时在调试程序时，你还可以用它来确认程序是正在写数据而不是被挂起了。

PS：调用fclose函数隐含执行了一次flush操作。

```c++
#include <stdio.h>
int fflush(FILE *stream)
```

### 3.5.6 fseek函数

fseek函数是与lseek系统调用对应的文件流函数。它在文件流里为下一次读写操作指定位置。offset和whence参数的含义和取值与lseek系统调用完全一样。但lseek返回的是一个off_t数值，fseek返回的是一个整数：0表示成功，-1表示失败并设置errno指出错误。

```c++
#include <stdio.h>
int fseek(FILE *stream, long int offset, int whence);
```

### 3.5.7 fgetc、getc和getchar函数

fgetc函数从文件流里取出下一个字节并把它作为一个字符返回。当它到达文件尾或出现错误时，它返回EOF。必须通过ferror或feof来区分这两种情况。

```c++
#include <stdio.h>

int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar()
```

getc函数的作用和fgetc一样，但它有可能被实现为一个宏，如果是这样，stream参数就可能被计算不止一次，所以它不能有副作用（例如，它不能影响变量）。此外，你也不能保证能够使用getc的地址作为一个函数指针。

getchar函数的作用相当于getc(stdin)，它从标准输入里读取下一个字符。

### 3.5.8 fputc、putc和putchar函数

fputc函数把一个字符写到一个输出文件流中。它返回写入的值，如果失败，则返回EOF。

```c++
#include <stdio.h>

int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);
```

类似于fgetc和getc之间的关系，putc函数的作用相当于fputc，但它可能被实现为一个宏。

PS：putchar和getchar都是把字符当作int类型而不是char类型来使用，这就运去文件尾（EOF）标识取值为-1，这是一个超出字符数字编码范围的值。

### 3.5.9 fgets和gets函数

fgets函数从输入文件流stream里读取一个字符串。

```c++
#include <stdio.h>

char *fgets(char *s, int n, FILE *stream);
char *gets(char *s);
```

fgets把读到的字符写到s指向的字符串里，直到出现下面某种情况：遇到换行符，已经输入了n-1个字符，或者到达文件尾。它会把遇到的换行符也传递到接收的字符串里，再加上一个表示结尾的空字节\0。一次调用最多只能传输n-1个字符。

成功完成时，fgets返回一个指向字符串s的指针。如果文件流已经到达文件尾，fgets会设置这个文件流的EOF标识并返回空指针。如果出现读错误，fgets返回一个空指针并设置errno以指出错误的类型。

gets函数和fgets类似，只不过从标准输入读取数据并丢弃遇到的换行符，在接收字符串的尾部加上一个null字节。

## 3.6 格式化输入和输出

### 3.6.1 printf、fprintf和sprintf函数

```c++
#include <stdio.h>

int printf(const char *format, ...);
int sprintf(char *s, const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
```

printf函数把自己的输出送到标准输出。fprintf函数把自己的输出送到一个指定的文件流。sprintf函数把自己输出和一个结尾空字符写到作为参数传递过来的字符串s里。

### 3.6.2 scanf、fscanf和sscanf函数

scanf系列函数工作方式与printf系列函数相似，区别是前者的作用是从一个文件流里读取数据，并把数据值放到以指针参数形式传递过来的地址处的变量中。

```c++
#include <stdio.h>

int scanf(const char *format, ...);
int fsacnt(FILE *stream, const char *format, ...);
int sscanf(const char *s, const char *format, ...);
```

PS：我们使用`%[]`控制符读取由一个字符集合中的字符 构成的字符串。格式字符串`%[A-Z]`将读取一个由大写字母构成的字符串。如果字符集中的第一个字符是`^`，就表示将读取一个由不属于该字符集合中的字符构成的字符串。因此要读取一个带空格的字符串，并且在遇到第一个逗号时停止，你可以用`%[^,]`

### 3.6.3 其他流函数

- fgetpos：获得文件流的当前（读写）位置
- fsetpos：设置文件流的当前（读写）位置
- ftell：返回文件流当前（读写）位置的偏移值
- rewind：重置文件流里的读写位置
- freopen：重新使用一个文件流
- setvbuf：设置文件流的缓冲机制
- remove：相当于unlink函数，但如果它的path参数是一个目录的话，其作用就相当于rmdir函数

### 3.6.4 文件流错误

为了表明错误，许多stdio库函数会返回一个超出范围的值，比如空指针或EOF常数。此时，错误由外部变量errno指出：

```c++
#include <errno.h>

extern int errno;
```

也可以通过检查文件流的状态来确定是否发生了错误，或者是否到达了文件尾。

```c++
#include <stdio.h>

int ferror(FILE *stream);
int feof(FILE *stream);
void clearerr(FILE *stream);
```

ferror函数测试一个文件流的错误标识，如果该标识被设置就返回一个非零值，否则返回零。

feof函数测试一个文件流的文件尾标识，如果该标识被设置就返回非零值，否则返回零。

clearerr函数的作用是清除由stream指向的文件流的文件尾标识和错误标识。没有返回值也没有定义任何错误。

### 3.6.5 文件流和文件描述符

每个文件流都和一个底层文件描述符相关联。你可以把底层的输入输出操作与高层的文件流操作混合使用，但是一般来说，这并不是一个明智的做法，因为数据缓冲的后果难以预料。

```c++
#include <stdio.h>

int fileno(FILE *stream);
FILE *fdopen(int fildes, const char *mode);
```

可以通过调用fileno函数来确定文件流使用的是哪个底层文件描述符。它返回指定文件流使用的文件描述符，如失败就返回-1。

fdopen：为一个已经打开的文件描述符提供stdio缓冲区。返回一个新的文件流，失败时返回NULL。

## 3.7 文件和目录的维护

### 3.7.1 chmod系统调用

改变文件或目录的访问权限。

```c++
#include <sys/stat.h>
int chmod(const char *path, mode_t mode);
```

### 3.7.2 chown系统调用

超级用户可以用其来改变一个文件的属主。

```c++
#include <sys/types.h>
#include <unistd.h>

int chown(const char *path, uid_t, owner, gid_t group);
```

### 3.7.3 unlink、link和symlink系统调用

unlink系统调用删除一个文件的目录项并减少它的连接数，成功返回0，失败返回-1。

```c++
#include <unistd.h>

int unlink(const char *path);
int link(const char *path1, const char *path2);
int symlink(const char *path1, const char *path2);
```

如果一个 文件的连接数减少到零，并且没有进程打开它，这个文件就会被删除。

### 3.7.4 mkdir和rmdir系统调用

mkdir：建立目录

```c++
#include <sys/types.h>
#include <sys/stat.h>

int mkdir(const char *path, mode_t mode);
```

rmdir：删除目录

```c++
#include <unistd.h>
int rmdir(const char *path);
```

### 3.7.5 chdir系统调用和getcwd函数

chdir：切换目录

```c++
#include <unistd.h>
int chdir(const char *path);
```

getcwd：确定当前工作目录。将当前目录的名字写到给定的缓冲区buf里。如果目录名的长度超出了参数size给出的缓冲区长度，则返回NULL，如果成功则返回指针buf。

```c++
#include <unistd.h>
char *getcwd(char *buf, size_t size);
```

## 3.8 扫描目录

被称为目录流的指向这个结构的指针`DIR *`被用来完成各种目录操作，其使用方法与用来操作普通文件的文件流`FILE *`非常相似。用户不应该直接改动DIR结构中的数据字段。

### 3.8.1 opendir函数

opendir：打开一个目录并建立一个目录流，成功则返回一个指向DIR结构的指针，失败则返回一个空指针，该指针用于读取目录数据项。

```c++
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name);
```

### 3.8.2 readdir函数

返回一个指针，该指针指向的结构里保存着目录流dirp中下一个目录项的有关资料。如果在调用此函数的时候有其他进程在该目录里创建或删除 文件，则不能保证能够列出该目录里的所有文件和子目录。失败返回NULL。

```c++
#include <sys/types.h>
#include <dirent.h>

struct dirent *readdir(DIR *dirp);
```

### 3.8.3 telldir函数

该函数的返回值记录着一个目录流里的当前位置。

```c++
#include <sys/types.h>
#include <dirent.h>

long int telldir(DIR *dirp);
```

### 3.8.4 seekdir函数

作用：设置目录流dirp的目录项指针。loc的值用来设置指针位置，它应该通过前一个telldir调用获得。

```c++
#include <sys/types.h>
#include <dirent.h>

void seekdir(DIR *dirp, long int loc);
```

### 3.8.5 closedir函数

关闭一个目录流并释放与之关联的资源，成功返回0，失败返回-1。

```c++
#include <sys/types.h>
#include <dirent.h>

int closedir(DIR *dirp);
```

## 3.9 错误处理

错误代码的取值和含义都列在头文件`errno.h`里，

- EPERM：操纵不允许
- ENOENT：文件或目录不存在
- EINTR：系统调用被中断
- EIO：I/O错误
- EBUSY：设备或资源忙
- EEXIST：文件存在
- EINVAL：无效参数
- EMFILE：打开的文件过多
- ENODEV：设备不存在
- EISDIR：是一个目录
- ENOTDIR：不是一个目录

### 3.9.1 strerror函数

将错误代码映射为一个字符串，该字符串对发生的错误类型进行说明。

```c++
#include <string.h>
char *strerror(int errnum);
```

### 3.9.2 perror函数

将errno变量中报告的当前错误映射到一个字符串，并把它输出到标准错误输出流，该字符串的前面先加上字符串s（如果不为空）中给出 的信息，再加上一个冒号和一个空格。

```c++
#include <stdio.h>
void perror(const char *s);
```

## 3.10 /proc文件系统

`/proc`目录中包含许多特殊文件用来对驱动程序和内核信息进行更高层的访问。

## 3.11 高级主题：fcntl和mmap

本节很少被用到，但是在解决一些棘手问题时，它们可以提供比较简单的解决方案。

