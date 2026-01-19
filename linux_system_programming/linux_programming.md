---
typora-copy-images-to: ./${filename}.assets
---


# man

**安装**

```bash
sudo pacman -Sy man-pages
```
这种方式安装的是与当前系统glibc版本向匹配的man-pages。

也可以直接在网页查询Ubuntu各个版本的

{% link ubuntu man, https://manpages.ubuntu.com/manpages/jammy/en/,  https://assets.ubuntu.com/v1/49a1a858-favicon-32x32.png%}

**内容**

| Section | 主题分类                                           | 内容举例                                    | 典型用途                  |
| :-----: | :--------------------------------------------- | :-------------------------------------- | :-------------------- |
|  **1**  | 用户命令（User Commands）                            | `ls(1)`, `grep(1)`, `man(1)`            | 普通 shell 命令、可执行程序     |
|  **2**  | 系统调用（System Calls）                             | `open(2)`, `read(2)`, `fork(2)`         | 内核提供的系统调用接口（C 语言层）    |
|  **3**  | C 库函数（Library Functions）                       | `printf(3)`, `malloc(3)`, `strcpy(3)`   | 标准 C 库（libc）与其他库函数    |
|  **4**  | 设备与特殊文件（Special Files）                         | `null(4)`, `tty(4)`                     | `/dev` 下的设备文件接口       |
|  **5**  | 文件格式与配置（File Formats and Conventions）          | `passwd(5)`, `fstab(5)`                 | 配置文件格式说明              |
|  **6**  | 游戏（Games）                                      | `fortune(6)`                            | 历史遗留部分，现在较少使用         |
|  **7**  | 概念与协议（Miscellaneous / Conventions / Protocols） | `signal(7)`, `socket(7)`, `regex(7)`    | 语义性或系统性文档（协议、宏、标准等）   |
|  **8**  | 系统管理命令（System Administration Commands）         | `mount(8)`, `ifconfig(8)`, `systemd(8)` | 只有 root 或管理员使用的命令     |
|  **9**  | 内核开发接口（Kernel Developer Docs, 非标准）             | `request_irq(9)`（在内核源码自带）               | 内核编程 API（仅在内核源码文档中出现） |

**使用**

```bash
man 2 open
```



# Linux下I/O操作

## 文件I/O

### open()

| 函数              | int open(const char *pathname, int flags)<br />int open(const char *pathname, int flags, mode_t mode) |
| ----------------- | ------------------------------------------------------------ |
| 头文件            | **#include <sys/types.h>**<br />**#include <sys/stat.h>**<br />**#include <fcntl.h>** |
| 参数 **pathname** | 路径和文件名                                                 |
| 参数 **flags**    | 文件打开方式，可用多个标志位按位或设置                       |
| 参数 **mode**     | 权限掩码，对不同用户和组设置可执行，读，写权限，使用八进制数表示，此参数可不写 |
| 返回值            | open () 执行成功会**返回 int 型文件描述符**，出错时返回 - 1。 |
| 作用              | 通过系统调用，可以打开文件，并返回文件描述符                 |

参数 flags 可选标志：

- **O_CREAT**
  要打开的文件名不存在时自动创建改文件。

- **O_EXCL**
  要和 O_CREAT 一起使用才能生效，如果文件存在则 open()调用失败。

- **O_RDONLY**
  只读模式打开文件。

- **O_WRONLY**
  只写模式打开文件。

- **O_RDWR**
  可读可写模式打开文件。

- **O_APPEND**
  以追加模式打开文件。

- **O_NONBLOCK** 

  以非阻塞模式打开

### close()

| 函数        | int close(int fd)        |
| ----------- | ------------------------ |
| 头文件      | **#include <unistd.h>**  |
| 参数 **fd** | 文件描述符               |
| 返回值      | 成功返回 0；错误返回 - 1 |

### read()

| 函数           | ssize_t read(int fd, void *buf, size_t count)                |
| -------------- | ------------------------------------------------------------ |
| 头文件         | **#include <unistd.h>**                                      |
| 参数 **fd**    | 要读的文件描述符                                             |
| 参数 **buf**   | 缓冲区，存放读到的内容                                       |
| 参数 **count** | 每次读取的字节数                                             |
| 返回值         | 返回值大于 0，表示读取到的字节数；<br />等于 0 在阻塞模式下表示到达文件末尾或没有数据可读（EOF），并调用阻塞；<br />等于 - 1 表示出错，在非阻塞模式下表示没有数据可读。 |

### write()

| 函数           | ssize_t write(int fd, const void *buf, size_t count);        |
| -------------- | ------------------------------------------------------------ |
| 头文件         | **#include <unistd.h>**                                      |
| 参数 **fd**    | 文件描述符                                                   |
| 参数 **buf**   | 缓存区，存放将要写入的数据                                   |
| 参数 **count** | 每次写入的字节数（原 “个数” 表述更准确为字节数）             |
| 功能           | 从 buf 缓冲区读取 count 个字节写入由 fd 标识的文件           |
| 返回值         | 大于或等于 0 表示执行成功，返回写入的字节数；<br />返回 - 1 代表出错 |

注意open时要有写权限

### lseek()

所有打开的文件都有一个**当前文件偏移量（current file offset）**，以下简称为 cfo。cfo 通常是一个**非负整数**，用于表明文件开始处到文件当前位置的字节数。读写操作通常开始于 cfo，并且使 cfo 增大，增量为读写的字节数。文件被打开时，cfo 会被初始化为 0，除非使用了 O_APPEND 。 使用 lseek 函数可以改变文件的 cfo 。

| 项目                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| 函数定义              | **off_t lseek(int fd, off_t offset, int whence);**           |
| 头文件                | **#include <sys/types.h>**<br />**#include <unistd.h>**      |
| 参数 **fd**           | 文件描述符                                                   |
| 参数 **off_t offset** | 偏移量，**单位为字节**，**正负分别表示向前、向后移动**       |
| 参数 **whence**       | 位置基点，可选 **SEEK_SET**（文件开头）、**SEEK_CUR**（当前指针位置）、**SEEK_END**（文件末尾） |
| 功能                  | 移动文件读写指针；获取文件长度；拓展文件空间                 |
| 返回值                | 成功**返回当前位移(SEEK_CUR)**，失败返回 - 1                 |



**例子**：

- 把文件位置指针设置为 100(开头+100个字节)

  ```c
  lseek(fd,100,SEEK_SET);
  ```

- 把文件位置设置成文件末尾
  ```c
  lseek(fd,0,SEEK_END);
  ```

- 确定当前的文件位置
  ```c
  lseek(fd,0,SEEK_CUR);
  ```

### access()

```c
       #include <unistd.h>

       int access(const char *path, int amode);
```

| 项目                | 内容                                                         |
| ------------------- | ------------------------------------------------------------ |
| **头文件**          | `#include <unistd.h>`                                        |
| **函数原型**        | `int access(const char *path, int amode);`                   |
| **作用**            | 检查指定路径 `path` 对调用进程的**实际用户ID（real UID）和组ID（real GID）** 是否可访问（例如可读、可写、可执行或存在）。 |
| **参数**            | **`path`**：要检查的文件路径。<br />**`amode`**：要检查的访问模式，是以下常量之一或它们的按位或： <br />`F_OK`：文件是否存在<br />`R_OK`：是否可读<br />`W_OK`：是否可写<br />`X_OK`：是否可执行 |
| **返回值**          | 成功返回 `0`；失败返回 `-1` 并设置 `errno`。                 |
| **常见 errno**      | `EACCES`：权限不足<br />`ENOENT`：文件不存在<br />`ENOTDIR`：路径中某部分不是目录<br />`EROFS`：写入只读文件系统<br />`ELOOP`：符号链接过多<br />`ENAMETOOLONG`：路径太长 |
| **行为说明**        | - 使用**真实用户ID和组ID**来检查权限，而不是有效ID。- 如果 `amode` 是多个标志的组合，会分别检查。- 仅检查访问权限，不实际打开文件。 |
| **注意事项 / 限制** | ⚠️ 不推荐使用，因为存在 **TOCTTOU（time-of-check-to-time-of-use）竞态条件**：检查权限后文件可能已被修改。<br />👉 更安全做法：直接尝试操作（如 `open()`）并捕获错误。 |
| **相关函数**        | `faccessat()`（更安全，可指定目录fd）`chmod()`、`fstat()`    |



## 目录I/O

### mkdir()



| 项目              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 函数              | **int mkdir(const char *pathname, mode_t mode)**;            |
| 头文件            | **#include <sys/stat.h><br />#include <sys/types.h>**        |
| 参数 **pathname** | 要创建目录的路径和名称                                       |
| 参数 **mode**     | **权限掩码**，以八进制数设置用户、组的读、写、执行权限，该参数可省略 |
| 返回值            | 成功返回 0，出错返回 - 1                                     |
| 功能              | 创建一个目录                                                 |

### opendir()/closedir()

| 函数          | opendir                                                |
| ------------- | ------------------------------------------------------ |
| 函数定义      | **DIR *opendir(const char *name);**                    |
| 头文件        | **#include <sys/types.h>**<br/>**#include <dirent.h>** |
| 参数 **name** | 目录的路径名                                           |
| 返回值        | 成功**返回目录流（`DIR*` 类型）**，失败返回 `NULL`     |
| 功能          | 打开指定目录，获取用于遍历目录的目录流                 |

| 函数          | closedir                                               |
| ------------- | ------------------------------------------------------ |
| 函数定义      | **int closedir(DIR *dirp)**                            |
| 头文件        | **#include <sys/types.h>**<br/>**#include <dirent.h>** |
| 参数 **dirp** | 要关闭的目录流指针                                     |
| 功能          | 关闭目录流，释放相关资源                               |

### readdir()

```bash
man 3 readdir
```



| 项目               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| 函数               | **struct dirent *readdir(DIR *dirp)**;<br/> **int readdir_r(DIR *dirp, struct dirent *entry, struct dirent \*\*result);** |
| 头文件             | **#include <dirent.h>**                                      |
| 参数 **DIR *dirp** | 要读取的目录流指针                                           |
| 返回值             | 成功返回读取到的目录项指针（`struct dirent*` 类型），失败返回 `NULL` |
| 功能               | 读取目录中的目录项，用于遍历目录内容                         |

 In the glibc implementation, the dirent structure is defined as follows:

```c
           struct dirent {
               ino_t          d_ino;       /* Inode number */
               off_t          d_off;       /* Not an offset; see below */
               unsigned short d_reclen;    /* Length of this record */
               unsigned char  d_type;      /* Type of file; not supported
                                              by all filesystem types */
               char           d_name[256]; /* Null-terminated filename */
           };
```

注意读取返回值dirent是链表形式连接的，意味着每次读取会读取目录中的一个目录项，需要循环读取才能遍历所有的目录项。

## 库

**库是一种可执行的二进制文件，是编译好的代码**。

使用库可以提高开发效率。在 Linux 下有静态库和动态库。

- 静态库在程序编译的时候会被链接到目标代码里面。所以程序在运行的时候不再需要静态库了。因此编译出来的体积就比较大。**以 lib 开头，以.a 结尾**。

- 动态库（动态库也叫共享库）在程序编译的时候不会被链接到目标代码里面，而是在程序运行的时候被载入的。所以程序在运行的时候需要动态库了。因此编译出来的体积就比较小。**以 lib 开头，以.so 结尾**。

### 静态库

- 静态库的制作步骤：

  1. 编写或准备库的源代码

  2. 将源码.c 文件编译生成.o 文件

  3. **使用 ar 命令创建静态库**

  4. 测试库文件
  

例子：

```bash
# -c只编译但不链接，生成 .o 对象文件
gcc -c mylib.c -o mylib.o

ar cr libmylib.a mylib.o
```

ar命令的含义：

| 部分         | 解释                                          |
| ------------ | --------------------------------------------- |
| `ar`         | “archiver” 工具，用来打包 `.o` 文件           |
| `c`          | create：创建一个新的归档文件（不管是否存在）  |
| `r`          | replace：把目标文件加入归档（如已存在则替换） |
| `libmylib.a` | 输出的静态库文件名                            |
| `mylib.o`    | 要加入库的目标文件                            |



### 动态库

- 动态库制作步骤：
  1. 编写或准备库的源代码
  2. 将源码.c 文件编译生成.o 文件
  3. **使用 gcc 命令创建动态库**
  4. 测试库文件



例子：

```bash
gcc -c -fpic mylib.c -o mylib.o
gcc -shared -o libmylib.so mylib.o
```

第一步

| 选项         | 含义                                                    |
| ------------ | ------------------------------------------------------- |
| `-c`         | 只编译，不链接（生成 `.o`）                             |
| `-fpic`      | 生成 **位置无关代码**（Position Independent Code, PIC） |
| `mylib.c`    | 源文件                                                  |
| `-o mylib.o` | 输出文件名                                              |

第二步

| 选项             | 含义                                         |
| ---------------- | -------------------------------------------- |
| `-shared`        | 生成共享对象（`.so` 文件），而不是可执行文件 |
| `-o libmylib.so` | 指定输出文件                                 |
| `mylib.o`        | 输入的目标文件                               |



**系统会默认去/lib，/usr/lib 目录下去查找动态函数库**，如果我们使用的库不在里面，就会提示错误。

- 第一种方法：
  将生成的动态库拷贝到/lib 或者/usr/lib 里面去，因为系统会默认去这俩个路径下寻找。

- 第二种方法：
  把我们的动态库所在的路径加到环境变量里面去，比如我们动态库所在的路径为/home/test，我们就可以这样添加，但是这种方法只在当前设置的窗口有效。

  ```bash
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/test/
  ```

- 第三种方法：
  **修改 ubuntu 下的配置文件/etc/ld.so.conf**，我们在这个配置文件里面加入动态库所在的位置，然后**使用命令 ldconfig 更新目录**。



# 进程与进程通信

每个进程都有一个唯一的标识符，既进程 ID，简称 **pid**

进程间的通信的方法：

1. **管道通信**：有名管道，无名管道
2. **信号通信**：信号的发送，信号的接受，信号的处理
3. **IPC 通信**：共享内存，消息队列，信号灯
4. **Socket 通信**

## 进程基础

### getpid()

| 项目   | getppid 函数说明                                   |
| ------ | -------------------------------------------------- |
| 头文件 | **#include <sys/types.h><br/>#include <unistd.h>** |
| 函数   | **pid_t getppid(void);**                           |
| 返回值 | 父进程的 PID 号                                    |
| 功能   | 获取当前进程的父进程 PID                           |

### fork()

| 项目   | fork 函数说明                                                |
| ------ | ------------------------------------------------------------ |
| 头文件 | **#include <unistd.h>**                                      |
| 函数   | **pid_t fork(void);**                                        |
| 返回值 | 调用成功时，**父进程返回子进程 PID，子进程返回 0**；<br />失败返回 -1 |
| 功能   | 系统调用，创建一个与父进程几乎完全相同的子进程，是实现进程并发的基础手段之一 |



### exec

| 项目              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 函数              | **int execve(const char *filename, char *const argv[], char *const envp[]);** |
| 头文件            | **#include <unistd.h>**                                      |
| 参数 **filename** | 新程序的路径名，指定要载入进程空间的程序位置                 |
| 参数 **argv []**  | 命令行参数数组，`argv[0]` 为命令名，后续元素为参数           |
| 参数 **envp []**  | 新程序的环境变量数组                                         |
| 返回值            | 成功时不会返回；若调用失败，返回 - 1，可通过 `errno` 检查错误原因 |

以下函数都是根据execve的实现

```c
int execl(const char *path, const char *arg, .../* (char
*) NULL */);

int execlp(const char *file, const char *arg, .../* (char
*) NULL */);

int execle(const char *path, const char *arg, .../*, (char *) NULL, char * const envp[] */);

int execv(const char *path, char *const argv[])
    
int execvp(const char *file, char *const argv[]);

int execvpe(const char *file, char *const argv[],char *const envp[]);
```

| 函数原型 | 核心差异（路径 / 参数 / 环境变量）                           | 头文件                            | 返回值                |
| -------- | ------------------------------------------------------------ | --------------------------------- | --------------------- |
| `int execl(const char *path, const char *arg, .../* (char *) NULL */);` | 绝对路径、可变参数、继承环境      | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |
| `int execlp(const char *file, const char *arg, .../* (char *) NULL */);` | 自动搜 PATH、可变参数、继承环境   | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |
| `int execle(const char *path, const char *arg, .../*, (char *) NULL, char * const envp[] */);` | 绝对路径、可变参数、自定义环境    | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |
| `int execv(const char *path, char *const argv[]);`           | 绝对路径、参数数组、继承环境      | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |
| `int execvp(const char *file, char *const argv[]);`          | 自动搜 PATH、参数数组、继承环境   | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |
| `int execvpe(const char *file, char *const argv[], char *const envp[]);` | 自动搜 PATH、参数数组、自定义环境 | `#include <unistd.h>` | 成功不返回，失败返回 - 1 |

1. **路径规则**：带`p`的函数（execlp、execvp、execvpe）可传入文件名，自动在系统`PATH`环境变量指定的目录中搜索程序；不带`p`的函数（execl、execle、execv）需传入程序的绝对路径（如`/bin/ls`）。
2. **参数传递**：带`l`的函数（execl、execlp、execle）用可变参数（`...`）传递命令行参数，最后必须以`(char*)NULL`结尾；带`v`的函数（execv、execvp、execvpe）用字符串数组`argv`传递参数，数组末尾需为`NULL`。
3. **环境变量**：带`e`的函数（execle、execvpe）需手动传入环境变量数组`envp`，自定义新程序的运行环境；不带`e`的函数直接继承当前进程的环境变量。

例子：

```c
if (pid == 0)
{
	printf("This is child,child pid is %d\n", getpid(), getppid());
	execl("/home/test/hello","hello",NULL);
	exit(1);//exit(1)不会执行（除非 execl() 调用失败）
}
```



### **`ps` 命令**

**功能：** 列出系统中当前运行的进程及其状态。

常用参数

| 参数  | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| `aux` | 显示所有用户的所有进程（a：显示终端以外的进程，u：显示用户信息，x：显示没有控制终端的进程） |
| `-ef` | 类似 aux，显示全格式信息                                     |
| `-e`  | 显示所有进程                                                 |
| `-f`  | 完整格式显示（Full format）                                  |

例子：

```bash
$ ps aux
```

输出包括字段：

| 字段    | 含义               |
| ------- | ------------------ |
| USER    | 进程所属用户       |
| PID     | 进程 ID            |
| %CPU    | CPU 使用率         |
| %MEM    | 内存使用率         |
| VSZ     | 虚拟内存大小（KB） |
| RSS     | 常驻内存大小（KB） |
| STAT    | 进程状态           |
| COMMAND | 命令名及参数       |



**进程状态STAT**

| 字符  | 描述                                            |
| ----- | ----------------------------------------------- |
| **D** | 不可中断的休眠状态（通常是等待 I/O）            |
| **R** | 正在运行或可运行状态（在 CPU 上执行或等待调度） |
| **S** | 休眠状态（可中断睡眠，等待事件）                |
| **T** | 暂停或追踪状态（如 `Ctrl+Z` 暂停或被调试）      |
| **Z** | 僵尸进程（子进程已结束，但父进程未回收）        |
| **W** | 内存不足，无法分页（很少见）                    |

额外修饰符

| 字符 | 描述                           |
| ---- | ------------------------------ |
| `<`  | 高优先级进程                   |
| `N`  | 低优先级进程                   |
| `L`  | 有内存锁定（Locked in memory） |

### **`kill`命令**

**功能：** 向进程发送信号，通常用来终止进程。

例子：

```bash
$ kill -l
$ kill -9 某个进程的pid
```



常用信号

| 信号           | 描述                                 |
| -------------- | ------------------------------------ |
| `SIGTERM` (15) | 终止进程，允许进程清理资源，默认信号 |
| `SIGKILL` (9)  | 强制终止进程，不可被捕获、阻塞或忽略 |
| `SIGSTOP` (19) | 暂停进程，可被 `SIGCONT` 恢复        |
| `SIGCONT`      | 继续执行被暂停的进程                 |

### 孤儿进程

**孤儿进程**：**父进程结束以后，子进程还未结束**，这个子进程就叫做孤儿进程。

**孤儿进程会被init进程领养**（ubuntu中为/sbin/upstart）,因为子进程的某些资源必须由父进程来释放

### 僵尸进程

**僵尸进程**：**子进程结束以后，父进程还在运行，但是父进程不去释放进程控制块**，这个子进程就叫做僵尸进程



### wait

wait()函数一般用在父进程中等待回收子进程的资源，而防止僵尸进程的产生。

| 项目       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 函数       | `pid_t wait(int *status)`                                    |
| 头文件     | `#include <sys/wait.h>`                                      |
| 返回值     | 成功返回回收的子进程 PID，失败返回 - 1                       |
| 功能与解读 | `wait` 是 Unix/Linux 系统中用于父进程等待子进程终止的系统调用。它会阻塞父进程，直到有子进程退出，然后回收该子进程的资源并返回其 PID。`status` 指针用于存储子进程的退出状态，可通过 `WIFEXITED`、`WEXITSTATUS` 等宏解析子进程是正常退出还是异常终止及具体退出码。该函数是实现进程同步、资源回收的关键工具，常用于父进程需要等待子进程完成任务后再继续执行的场景，例如多进程编程中父进程统一管理子进程的生命周期。 |

与 wait 函数的参数有关的俩个宏定义：
**WIFEXITED(status)**：如果**子进程正常退出，则该宏定义为真**
**WEXITSTATUS(status)**：如果**子进程正常退出，则该宏定义的值为子进程的退出值**。



### 守护进程

**守护进程(daemon)是一类在后台运行的特殊进程，用于执行特定的系统任务**。很多守护进程在系统引导的时候启动，并且一直运行直到系统关闭。另一些只在需要的时候才启动，完成任务后就自动结束。

守护进程没有控制终端，因此当某些情况发生时，不管是一般的报告性信息，还是需由管理员处理的紧急信息，都需要以某种方式输出。Syslog 函数就是输出这些信息的标准方法，它**把信息发送给 syslogd 守护进程**。



**守护进程的基本要求**：

1. 必须作为我们 init 进程的子进程（让子进程变成孤儿进程）
2. 不跟控制终端交互





**步骤：**

1. **使用 fork 函数创建一个新的进程，然后让父进程使用 exit 函数直接退出（必须要的）**
2. **调用 setsid 函数。（必须要的）**
3. 调用 chdir 函数，将当前的工作目录改成根目录，增强程序的健壮性。（不是必须要的）
4. 重设我们 umask 文件掩码，增强程序的健壮性和灵活性（不是必须要的）
5. 关闭文件描述符，节省资源（不是必须要的）
6. 执行我们需要执行的代码（必须要的）



| 步骤                    | 说明                                              | 必要性 |
| ----------------------- | ------------------------------------------------- | ------ |
| 1️⃣ `fork()` 并退出父进程 | 让子进程脱离原来的父进程控制，避免成为进程组长    | ✅ 必须 |
| 2️⃣ `setsid()`            | **创建新的会话（session），使进程脱离原控制终端** | ✅ 必须 |
| 3️⃣ `chdir("/")`          | 切换工作目录到根目录，防止占用挂载点              | ⚙️ 建议 |
| 4️⃣ `umask(0)`            | 清除文件创建掩码，保证文件权限可控                | ⚙️ 建议 |
| 5️⃣ 关闭文件描述符        | 关闭继承来的 stdin/stdout/stderr 等，防止误用终端 | ⚙️ 建议 |
| 6️⃣ 执行主逻辑            | 进入业务逻辑循环                                  | ✅ 必须 |

例子：

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>
int main(void)
{
	pid_t pid;
	// 步骤一：创建一个新的进程
	pid = fork();
	//父进程直接退出
	if (pid > 0){
		exit(0);
	}
	if (pid == 0){
		// 步骤二：调用 setsid 函数摆脱控制终端
		setsid();
		// 步骤三：更改工作目录
		chdir("/");
		// 步骤四：重新设置 umask 文件源码
		umask(0);
		// 步骤五：0 1 2 三个文件描述符
		close(1);
		close(2);
		close(3);
		// 步骤六：执行我们要执行的代码
    	while (1){
            
		}
	}
	return 0;
}
```

## 进程间的通信

进程间的通信应用也是很广泛的，比如后台进程和 GUI 界面数据传递，发送信号关机，Ctrl+C 终止正在运行的程序等。

### 无名管道

无名管道是最古老的进程通信方式，有如下两个特点：

1. **只能用于有关联的进程间数据交互**，如父子进程，兄弟进程，子孙进程，在目录中看不到文件节点，读写文件描述符存在一个 int 型数组中。
2. **只能单向传输数据**，即管道创建好后，一个进程只能进行读操作，另一个进程只能进行写操作，读出来字节顺序和写入的顺序一样。

| 项目            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| 函数            | `int pipe(int pipefd[2])`                                    |
| 头文件          | `#include <unistd.h>`                                        |
| 参数 pipefd [2] | 整型数组，`pipefd[0]` 为读端文件描述符，`pipefd[1]` 为写端文件描述符 |
| 返回值          | 成功返回 0，失败返回 - 1                                     |
| 功能与解读      | `pipe` 是 Unix/Linux 系统中用于创建无名管道的系统调用。无名管道是进程间通信的一种方式，仅适用于具有亲缘关系（如父子进程）的进程间双向通信。创建后，一个进程可通过写端（`pipefd[1]`）写入数据，另一个进程通过读端（`pipefd[0]`）读取数据，实现数据的单向传输（若需双向，需创建两个管道）。常用于父子进程间的命令传递、数据共享等场景，例如父进程通过管道向子进程发送指令，子进程执行后将结果通过管道返回父进程。 |



**步骤**：

1. **调用 pipe()创建无名管道**；
2. **fork()创建子进程，一个进程读，使用 read()，一个进程写，使用 write()**。

注意：**必须在fork函数之前创建无名管道！**

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>

int main(void)
{
	char buf[32] = {0};
	pid_t pid;
	// 定义一个变量来保存文件描述符
	// 因为一个读端，一个写端，所以数量为 2 个
	int fd[2];
	// 创建无名管道
	pipe(fd);
	printf("fd[0] is %d\n", fd[0]);
	printf("fd[2] is %d\n", fd[1]);
	// 创建进程
	pid = fork();
	if (pid < 0){
		printf("error\n");
	}
	if (pid > 0){
		int status;
		close(fd[0]);
		write(fd[1], "hello", 5);
		close(fd[1]);
		wait(&status);
		exit(0);
	}
	if (pid == 0){
		close(fd[1]);
		read(fd[0], buf, 32);
		printf("buf is %s\n", buf);
		close(fd[0]);
		exit(0);
	}
    return 0;
}
```

### 有名管道

| 项目              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 函数              | `int mkfifo(const char *pathname, mode_t mode)`              |
| 头文件            | `#include <sys/types.h>`<br />`#include <sys/stat.h>`        |
| 参数 **pathname** | 有名管道的路径和名称，用于标识要创建的管道文件               |
| 参数 **mode**     | 权限掩码，以八进制数设置用户、组的读、写权限（如`0666`）     |
| 返回值            | 成功返回 0，失败返回 - 1                                     |
| 功能与解读        | `mkfifo` 是 Unix/Linux 系统中用于创建有名管道的函数。有名管道是一种特殊的文件，可用于无亲缘关系的进程间通信。创建后，一个进程可像操作普通文件一样向其写入数据，另一个进程可从中读取数据，实现进程间的双向通信。常用于不同程序间的信息交互场景，例如服务程序与客户端程序通过有名管道传递指令和数据。使用时需注意权限设置，确保通信进程对管道文件有相应的读写权限。 |





**使用步骤**：

1. 使用 mkfifo()创建 fifo 文件描述符。
2. 打开管道文件描述符。
3. 通过读写文件描述符进行单向数据传输



**例子**

fifo_write.c

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main(int argc, char *argv[])
{
	int ret;
	char buf[32] = {0};
	int fd;
	if (argc < 2){
		printf("Usage:%s <fifo name> \n", argv[0]);
		return -1;
	}
	if (access(argv[1], F_OK) == 0){
		ret = mkfifo(argv[1], 0666);
		if (ret == -1){
			printf("mkfifo is error \n");
			return -2;
		}
		printf("mkfifo is ok \n");
	}
    fd = open(argv[1], O_WRONLY);
	while (1){
		sleep(1);
		write(fd, "hello", 5);
	}
	close(fd);
	return 0;
}
```



fifo_read.c

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
int main(int argc, char *argv[])
{
	char buf[32] = {0};
	int fd;
	if (argc < 2){
		printf("Usage:%s <fifo name> \n", argv[0]);
		return -1;
	}
	fd = open(argv[1], O_RDONLY);
	while (1){
		sleep(1);
		read(fd, buf, 32);
		printf("buf is %s\n", buf);
		memset(buf, 0, sizeof(buf));
	}
	close(fd);
	return 0;
}
```





也可以使用命令来创建

```bash
mkfifo fifo
ls
ls -al
```

管道类型文件大小为0不占用磁盘空间



### 信号通信

```bash
[zhaohang@cyberboy /]$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

| 信号名              | 说明                               | 默认操作                   |
| ------------------- | ---------------------------------- | -------------------------- |
| **SIGHUP**          | 终端挂起或控制终端关闭（用户登出） | 终止                       |
| **SIGINT**          | 终端中断（Ctrl+C）                 | 终止                       |
| **SIGQUIT**         | 终端退出（Ctrl+\）                 | 终止 + 栈转储（core dump） |
| **SIGILL**          | 非法指令                           | 终止 + 栈转储              |
| **SIGTRAP**         | 调试断点、单步中断                 | 终止 + 栈转储              |
| **SIGABRT**         | `abort()` 调用触发                 | 终止 + 栈转储              |
| **SIGBUS**          | 总线错误（访问未对齐地址）         | 终止 + 栈转储              |
| **SIGFPE**          | 算术异常（如除零）                 | 终止 + 栈转储              |
| **SIGKILL**         | 强制终止（无法捕获/忽略）          | 终止                       |
| **SIGUSR1**         | 用户自定义信号 1                   | 终止                       |
| **SIGSEGV**         | 段错误（无效内存访问）             | 终止 + 栈转储              |
| **SIGUSR2**         | 用户自定义信号 2                   | 终止                       |
| **SIGPIPE**         | 向无读端的管道写数据               | 终止                       |
| **SIGALRM**         | `alarm()` 到期                     | 终止                       |
| **SIGTERM**         | 终止信号（可捕获）                 | 终止                       |
| **SIGCHLD**         | 子进程退出或停止                   | 忽略                       |
| **SIGCONT**         | 继续运行被停止的进程               | 继续执行                   |
| **SIGSTOP**         | 停止进程（不可捕获）               | 停止                       |
| **SIGTSTP**         | 用户暂停（Ctrl+Z）                 | 停止                       |
| **SIGTTIN**         | 后台进程试图从终端读               | 停止                       |
| **SIGTTOU**         | 后台进程试图向终端写               | 停止                       |
| **SIGURG**          | 套接字紧急数据                     | 忽略                       |
| **SIGXCPU**         | CPU 时间超限                       | 终止 + 栈转储              |
| **SIGXFSZ**         | 文件大小超限                       | 终止 + 栈转储              |
| **SIGVTALRM**       | 虚拟计时器到期（`ITIMER_VIRTUAL`） | 终止                       |
| **SIGPROF**         | 统计计时器到期（`ITIMER_PROF`）    | 终止                       |
| **SIGWINCH**        | 终端窗口大小改变                   | 忽略                       |
| **SIGIO / SIGPOLL** | 异步 I/O 事件（设备可读写）        | 忽略                       |
| **SIGSYS**          | 非法系统调用                       | 终止 + 栈转储              |
| **SIGPWR**          | 电源故障（部分系统实现）           | 忽略或终止                 |
| **SIGSTKFLT**       | 协处理器栈错误（很少用）           | 终止                       |

#### 发送信号

##### kill()

| 项目       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 函数       | `int kill(pid_t pid, int sig)`                               |
| 头文件     | `#include <sys/types.h>`<br />`#include <signal.h>`          |
| 参数 pid   | - 大于 0：向 PID 为 `pid` 的进程发送信号<br />- 等于 0：向同一进程组的进程发送信号<br />- 等于 - 1：向除自身外所有进程 ID 大于 1 的进程发送信号<br />- 小于 - 1：向组 ID 等于 `pid` 绝对值的进程组内所有进程发送信号 |
| 参数 sig   | 要发送的信号；等于 0 时为空信号，常用于错误检查              |
| 返回值     | 成功返回 0，错误返回 - 1 并设置`errno`                       |
| 功能与解读 | `kill` 是 Unix/Linux 系统中用于向进程或进程组发送信号的函数。信号是进程间通信的一种轻量级方式，可用于实现进程的中断、终止、状态查询等操作。例如，发送 `SIGTERM` 信号可优雅终止进程，发送 `SIGKILL` 信号可强制终止进程。在进程管理、程序调试等场景中应用广泛，是实现进程间异步通知的关键工具。 |



##### raise()

| 项目       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 函数       | `int raise(int sig)`                                         |
| 头文件     | `#include <signal.h>`                                        |
| 参数 sig   | 要发送的信号                                                 |
| 功能与解读 | `raise` 函数用于向进程自身发送信号，等效于 `kill(getpid(), sig)`。信号是进程间（包括自身）异步通信的机制，该函数可用于实现进程的自我中断、自我终止等逻辑，例如程序在检测到内部错误时，通过 `raise(SIGABRT)` 主动触发异常终止，便于调试和资源清理。 |

##### alarm()

| 项目   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 函数   | `unsigned int alarm(unsigned int seconds)`                   |
| 头文件 | `#include <unistd.h>`                                        |
| 参数   | 设定的闹钟时间（单位：秒）                                   |
| 功能   | **设定时间到期后向进程发送 `SIGALRM` 信号，默认动作是终止进程** |
| 注意   | **每个进程仅能有一个活跃的 `alarm` 定时器，若需再次使用需重新注册** |

#### 接收信号

接收信号：如果要让我们**接收信号的进程可以接收到信号，那么这个进程就不能停止**。让进程不停止有三种方法：

- while
- sleep
- **pause**

##### pause()

| 项目       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 函数       | `int pause(void)`                                            |
| 头文件     | `#include <unistd.h>`                                        |
| 返回值     | 进程被信号中断后返回 - 1                                     |
| 功能与解读 | `pause` 函数用于将进程挂起，使其进入休眠状态，直到收到一个信号。常用于进程需要等待外部事件（如信号触发）才能继续执行的场景，例如实现简单的进程同步或等待特定信号来处理异步事件。需要注意的是，若信号的处理动作是终止进程或忽略该信号，`pause` 不会返回；只有当信号的处理函数执行完毕后，`pause` 才会返回 - 1，此时可通过 `errno` 进一步分析（通常 `errno` 为 `EINTR` 表示被信号中断）。 |

#### 信号处理

**信号是由操作系统来处理的**，**即信号的处理在内核态**。信号不一定会立即被处理，此时会储存在信号的信号表中

![信号的处理](https://cdn.jsdelivr.net/gh/even629/myPicGo/202511032010370.png)

由上图中可看出信号有三种处理方式：

1. 默认方式（通常是终止进程），
2. 忽略，不进行任何操作。
3. 捕捉并处理调用信号处理器（回调函数形式）



##### signal()



| 项目              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 函数              | `sighandler_t signal(int signum, sighandler_t handler);`（可简化为 `signal(参数1, 参数2);`） |
| 头文件            | `#include <unistd.h>`                                        |
| 参数 1（signum）  | 要处理的信号，系统信号可通过终端命令 `kill -l` 查看          |
| 参数 2（handler） | 信号处理方式：<br />- **忽略信号**：填写 `SIG_IGN`<br />- **系统默认处理**：填写 `SIG_DFL`<br />- **捕获信号并执行自定义函数**：需传入符合 `sighandler_t` 类型的函数指针（函数定义格式为 `void 函数名(int)`） |
| 返回值            | 调用成功返回最后一次注册该信号时的 `handler` 值；失败返回 `SIG_ERR` |
| 功能与解读        | `signal` 函数用于修改进程收到信号后的处理动作，是 Unix/Linux 系统中信号处理的基础工具。通过它可实现对信号的忽略、默认处理或自定义捕获逻辑，例如捕获 `SIGINT` 信号（终端中断信号）来实现程序的优雅退出。需要注意的是，`signal` 在不同系统中的行为存在差异，在追求可移植性的场景下，更推荐使用 `sigaction` 函数。 |

例子：

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
int main(void)
{
	signal(SIGINT,SIG_IGN);
	while(1){
		printf("wait signal\n");
		sleep(1);
	}
	return 0;
}
```

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void myfun(int sig)
{
	if(sig == SIGINT){
		printf("get sigint\n");
	}
}

int main(void)
{
	signal(SIGINT,myfun);
	while(1){
		sleep(1);
		printf("wait signal\n");
	}
	return 0;
}
```

### 共享内存

共享内存，顾名思义就是允许两个不相关的进程访问同一个逻辑内存，共享内存是两个正在运行的进程之间共享和传递数据的一种非常有效的方式。不同进程之间共享的内存通常为同一段物理内存。进程可以将同一段物理内存连接到他们自己的地址空间中，所有的进程都可以访问共享内存中的地址。如果某个进程向共享内存写入数据，所做的改动将立即影响到可以访问同一段共享内存的任何其他进程。



**特点**：

1. 速度快，因为共享内存不需要内核控制，所以没有系统调用。而且没有向内核拷贝数据的过程，所以效率和前面几个相比是最快的，可以用来进行批量数据的传输，比如图片。
2. 没有同步机制，需要借助 Linux 提供其他工具来进行同步，通常使用信号灯



**使用共享内存的步骤**：

1. 调用 shmget()创建共享内存段 id，
2. 调用 shmat()将 id 标识的共享内存段加到进程的虚拟地址空间，
3. 访问加入到进程的那部分映射后地址空间，可用 IO 操作读写。





#### shmget()

创建或获取共享内存

| 项目        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| 函数        | `int shmget(key_t key, size_t size, int shmflg)`             |
| 参数 key    | 由 `ftok` 生成的键标识，用于唯一标识系统中的 IPC 资源（共享内存） |
| 参数 size   | 申请共享内存的大小，操作系统内存申请最小单位为页（4k 字节），通常需申请页的整数倍以避免内存碎片 |
| 参数 shmflg | 控制标志：<br />- 创建新共享内存：需结合 `IPC_CREAT` 和 `IPC_EXCL`<br />- 使用已有共享内存：可使用 `IPC_CREAT` 或直接传 0 |
| 返回值      | 成功返回共享内存标识符；失败返回 - 1 并设置错误码            |
| 功能与解读  | `shmget` 是 Unix/Linux 系统中用于创建或获取共享内存的系统调用。共享内存是进程间通信中效率最高的方式，多个进程可通过映射同一块共享内存实现数据共享。`shmget` 负责初始化共享内存的创建或定位已有共享内存，为后续的 `shmat`（映射）、`shmdt`（解除映射）、`shmctl`（控制）操作提供基础。例如，在多进程协作处理大数据时，可通过共享内存实现进程间的高速数据传输。 |



例子：

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <string.h>
#include <stdlib.h>

int main(void)
{
	int shmid;
	shmid = shmget(IPC_PRIVATE, 1024, 0777);
	if (shmid < 0){
		printf("shmget is error\n");
		return -1;
	}
	printf("shmget is ok and shmid is %d\n", shmid);
	return 0;
}
```

使用IPC_PRIVATE创建的话，key值为0



通过命令查看Share Memory Segments：

```bash
[zhaohang@cyberboy /]$ ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
```



#### ftok()

生成 IPC（进程间通信）资源的唯一键标识

| 项目            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| 函数            | `key_t ftok(const char *pathname, int proj_id)`              |
| 头文件          | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`         |
| 参数 `pathname` | 文件的路径及名称，用于生成唯一键的基础标识                   |
| 参数`proj_id`   | 字符型标识（通常取一个字节的非零值），用于区分同一文件生成的不同键 |
| 返回值          | 成功返回 `key` 值，失败返回 - 1                              |
| 功能与解读      | `ftok` 函数用于生成 IPC（进程间通信）资源的唯一键标识，是消息队列、共享内存、信号量等 IPC 机制的基础。它通过文件的 inode 信息和 `proj_id` 组合生成唯一的 `key`，确保不同进程能通过该 `key` 识别并访问同一 IPC 资源。例如，在多进程使用共享内存时，需先通过 `ftok` 生成一致的 `key`，再调用 `shmget` 创建或获取共享内存。使用时需注意，若文件路径或 `proj_id` 发生变化，生成的 `key` 也会改变，可能导致进程间无法识别共享资源。 |



#### shmat()

将共享内存挂接到进程的地址空间

| 项目                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 函数                       | `void *shmat(int shmid, const void *shmaddr, int shmflg)`    |
| 头文件                     | `#include <sys/types.h>`<br />`#include <sys/shm.h>`         |
| 参数 `int shmid`           | 共享内存标识符，即 `shmget` 函数的返回值，用于标识要挂接的共享内存 |
| 参数 `const void *shmaddr` | 映射地址，**通常写 `NULL`，由系统自动完成共享内存到进程地址空间的映射** |
| 参数 `int shmflg`          | 访问权限标志，`0` 表示可读可写，`SHM_RDONLY` 表示只读        |
| 返回值                     | 成功返回共享内存映射到进程中的地址；失败返回 - 1（`(void*)-1`） |
| 功能与解读                 | `shmat` 函数用于将共享内存挂接到进程的地址空间，是共享内存使用流程中的关键步骤。进程通过该函数获得共享内存的虚拟地址后，即可像操作普通内存一样读写共享区域，实现进程间的高效数据共享。例如，在多进程协作任务中，一个进程写入共享内存的数据，可被其他挂接了同一共享内存的进程读取。使用时需注意，挂接成功后需通过 `shmdt` 解除映射，避免资源泄漏；同时要关注访问权限，确保进程对共享内存的操作符合 `shmflg` 的设置。 |



#### shmdt()

删除进程地址空间中对共享内存的映射，不会删除内核中的共享内存对象

| 项目                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 函数                       | `int shmdt(const void *shmaddr)`                             |
| 头文件                     | `#include <sys/types.h>`<br />`#include <sys/shm.h>`         |
| 参数 `const void *shmaddr` | 共享内存映射到进程后的地址（即 `shmat` 函数的返回值）        |
| 返回值                     | 成功返回 0，失败返回 - 1                                     |
| 功能                       | 解除进程与共享内存的地址映射（去关联共享内存）               |
| 注意                       | `shmdt` 仅删除进程地址空间中对共享内存的映射，不会删除内核中的共享内存对象。若需删除内核中的共享内存，需调用 `shmctl` 函数并指定 `IPC_RMID` 命令。 |



#### shmctl()

对共享内存进行控制操作，通过不同的 `cmd` 命令，可实现**共享内存属性的查询、修改以及对象的删除**

| 项目                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| 函数                        | `int shmctl(int shmid, int cmd, struct shmid_ds *buf)`       |
| 头文件                      | `#include <sys/ipc.h>`<br />`#include <sys/shm.h>`           |
| 参数 `int shmid`            | 共享内存标识符，用于指定要操作的共享内存资源                 |
| 参数 `int cmd`              | 操作命令：<br />- `IPC_STAT`：获取共享内存的属性<br />- `IPC_SET`：设置共享内存的属性<br />- `IPC_RMID`：删除共享内存对象 |
| 参数 `struct shmid_ds *buf` | 用于存储或设置共享内存属性的结构体指针，在 `IPC_STAT` 和 `IPC_SET` 时使用 |
| 功能与解读                  | `shmctl` 函数用于对共享内存进行控制操作，是共享内存生命周期管理的关键工具。通过不同的 `cmd` 命令，可实现共享内存属性的查询、修改以及对象的删除。例如，当进程不再需要共享内存时，调用 `shmctl(shmid, IPC_RMID, NULL)` 可删除内核中的共享内存对象，释放系统资源。该函数确保了共享内存资源的合理回收，避免了因资源未释放导致的系统开销或冲突。 |





例子：

父子进程通过共享内存通信。

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <string.h>
#include <stdlib.h>
int main(void)
{
	int shmid;
	key_t key;
	pid_t pid;
	char *s_addr, *p_addr;
	
    key = ftok("./a.c", 'a');
	shmid = shmget(key, 1024, 0777 | IPC_CREAT);
	if (shmid < 0){
		printf("shmget is error\n");
		return -1;
	}
	
    printf("shmget is ok and shmid is %d\n", shmid);
	pid = fork();
	if (pid > 0){
		p_addr = shmat(shmid, NULL, 0);
		strncpy(p_addr, "hello", 5);
		wait(NULL);
		exit(0);
    }
	if (pid == 0){
		sleep(2);
		s_addr = shmat(shmid, NULL, 0);
		printf("s_addr is %s\n", s_addr);
		exit(0);
    }
	return 0;
}
```

### 消息队列

三种ipcs

```bash
[zhaohang@cyberboy /]$ ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```



![IPC对象存储于内核空间](https://cdn.jsdelivr.net/gh/even629/myPicGo/202511032059237.png)

应用层使用 IPC 通信的步骤为

1. 获取 key 值，内核会将 key 值映射成 IPC 标识符，获取 key 值常用方法：

   （1）在 get 调用中将 IPC_PRIVATE 常量作为 key 值。
   （2）使用 ftok()生成 key。

2. 执行 IPC get 调用，通过 key 获取整数 IPC 标识符 id，每个 id 表示一个 IPC 对象。
   消息队列：msgget()
   共享内存：shmget()
   信号量：semget()

3. 通过 id 访问 IPC 对象。





**消息队列的特点**：

1. 发出的消息以链表形式存储，相当于一个列表，进程可以根据 id 向对应的“列表”增加和获取消息。
2. 进程接收数据时可以按照类型从队列中获取数据。



**消息队列的使用步骤**：

1. 创建 key；
2. msgget()通过 key 创建（或打开）消息队列对象 id；
3. 使用 msgsnd()/msgrcv()进行收发；
4. 通过 msgctl()删除 ipc 对象





通过 msgget()调用获取到 id 后即可使用消息队列访问 IPC 对象，消息队列常用 API 如下：



#### msgget()

用于获取或创建消息队列的唯一标识 ID

| 函数       | msgget                                                       |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int msgget(key_t key, int msgflg)`                          |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/msg.h>` |
| 参数       | - `key`：消息队列关联的键值<br />- `msgflg`：访问权限: - `IPC_CREAT`：若消息队列不存在则创建 - `IPC_EXCL`：与 `IPC_CREAT` 一起使用，若消息队列已存在则报错 - 权限位（如 `0666`）：指定创建的消息队列访问权限 |
| 返回值     | 成功返回消息队列 ID，失败返回 - 1                            |
| 功能与解读 | 用于获取或创建消息队列的唯一标识 ID，是消息队列 IPC 机制的初始化步骤。 |

#### msgsnd()

| 函数       | msgsnd                                                       |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg)` |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/msg.h>` |
| 参数       | - `msqid`：消息队列 ID<br />- `msgp`：指向消息的指针<br />- `msgsz`：消息字节数<br />- `msgflg`：发送模式（0 为阻塞，`IPC_NOWAIT` 为非阻塞） |
| 返回值     | 成功返回 0，失败返回 - 1                                     |
| 功能与解读 | 用于向消息队列发送数据，支持阻塞 / 非阻塞模式，是进程间消息传递的核心工具。 |

#### msgctl()

| 函数       | msgctl                                                       |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int msgctl(int msqid, int cmd, struct msqid_ds *buf)`       |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/msg.h>` |
| 参数       | - `msqid`：消息队列 ID<br />- `cmd`：操作命令（`IPC_STAT`/`IPC_SET`/`IPC_RMID`）<br />- `buf`：存储或设置属性的结构体指针 |
| 返回值     | 成功返回 0，失败返回 - 1                                     |
| 功能与解读 | 用于对消息队列进行属性查询、修改或删除操作，是消息队列生命周期管理的关键函数。 |

#### msgrcv()

| 项目          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| 函数          | `ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg)` |
| 参数 `msqid`  | 消息队列的 IPC 标识 ID，用于指定要接收消息的队列             |
| 参数 `msgp`   | 消息缓冲区指针，用于存储接收到的消息（消息需包含类型和数据字段） |
| 参数 `msgsz`  | 消息数据字段的大小（不包含消息类型的字节数）                 |
| 参数 `msgtyp` | 要接收的消息类型，用于筛选特定类型的消息                     |
| 参数 `msgflg` | 位掩码，可组合多种标志（如 `IPC_NOWAIT` 表示非阻塞接收）     |
| 返回值        | 成功返回接收到的消息数据字段大小；错误返回 - 1               |
| 功能与解读    | `msgrcv` 是 Unix/Linux 系统中用于从消息队列接收消息的函数。它支持按消息类型筛选接收，可实现进程间的异步消息交互与分类处理。例如，在多客户端向服务端发送不同类型请求的场景中，服务端可通过 `msgtyp` 区分请求类型并分别处理。接收模式可通过 `msgflg` 配置为阻塞或非阻塞，灵活适配不同的业务需求。该函数是消息队列 IPC 机制中实现消息消费的关键环节。 |



例子：

向消息队列里写：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>

struct msgbuf
{
	long mtype;
	char mtext[128];
};

int main(void)
{
	int msgid;
	key_t key;
	struct msgbuf msg;
	//获取 key 值
	key = ftok("./a.c", 'a');
	//获取到 id 后即可使用消息队列访问 IPC 对象
	msgid = msgget(key, 0666 | IPC_CREAT);
	if (msgid < 0){
		printf("msgget is error\n");
		return -1;
	}
	printf("msgget is ok and msgid is %d \n", msgid);
	msg.mtype = 1;
	strncpy(msg.mtext, "hello", 5);
	//发送数据
	msgsnd(msgid, &msg, strlen(msg.mtext), 0);
	return 0;
}
```

从头消息队列中读

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>

struct msgbuf
{
	long mtype;
    char mtext[128];
};

int main(void)
{
	int msgid;
	key_t key;
	struct msgbuf msg;
	key = ftok("./a.c", 'a');
	//获取到 id 后即可使用消息队列访问 IPC 对象
	msgid = msgget(key, 0666 | IPC_CREAT);
	if (msgid < 0){
		printf("msgget is error\n");
		return -1;
	}
	printf("msgget is ok and msgid is %d \n", msgid);
	//接收数据
	msgrcv(msgid, (void *)&msg, 128, 0, 0);
	printf("msg.mtype is %ld \n", msg.mtype);
	printf("msg.mtext is %s \n", msg.mtext);
	return 0;
}
```

### 信号量

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的



信号量只能进行两种操作等待和发送信号，即 **P(sv)**和 **V(sv)**,

- P(sv)：如果 sv 的值大于零，就给它减 1；如果它的值为零，就挂起该进程的执行

- V(sv)：如果有其他进程因等待 sv 而被挂起，就让它恢复运行，如果没有进程因等待 sv 而挂起，就给它加 1。

#### semget()

| 函数       | semget                                                       |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int semget(key_t key, int nsems, int semflg)`               |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/sem.h>` |
| 参数       | - `key`：信号量的键值<br />- `nsems`：信号量的数量<br />- `semflg`：标识（如创建 / 访问权限） |
| 返回值     | 成功返回信号量 ID，失败返回 - 1                              |
| 功能与解读 | 用于创建新信号量或获取已有信号量的 ID，是信号量 IPC 机制的初始化步骤。 |

#### semctl()

| 函数       | semctl                                                       |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int semctl(int semid, int semnum, int cmd, union semun arg)` |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/sem.h>` |
| 参数       | - `semid`：信号量 ID- `semnum`：信号量编号<br />- `cmd`：操作命令（`IPC_STAT`/`IPC_SET`/`IPC_RMID`/`SETVAL` 等）<br />- `arg`：`union semun` 结构体，用于存储或设置信号量属性 |
| 功能与解读 | 用于对信号量进行属性查询、修改、初始化或删除操作，是信号量生命周期管理的关键函数。 |

#### semop()

| 函数       | semop                                                        |
| ---------- | ------------------------------------------------------------ |
| 函数原型   | `int semop(int semid, struct sembuf *sops, size_t nsops)`    |
| 头文件     | `#include <sys/types.h>`<br />`#include <sys/ipc.h>`<br />`#include <sys/sem.h>` |
| 参数       | - `semid`：信号量 ID<br />- `sops`：`struct sembuf` 结构体数组，每个元素包含信号量编号、操作（`sem_op`：1 为 V 操作，-1 为 P 操作，0 为等待）、操作模式（`sem_flg`：0 为阻塞，`IPC_NOWAIT` 为非阻塞）<br />- `nsops`：要操作的信号量数量 |
| 功能与解读 | 用于在信号量上执行 P（申请资源）、V（释放资源）等操作，实现进程间的同步与互斥。 |



