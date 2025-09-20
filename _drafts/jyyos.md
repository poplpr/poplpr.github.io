readelf 展示 elf 格式文件文件头（File Header） 信息的指令。

在进程管理中，`fork()` 是在当前位置新增一个子进程，而 `execve(path, argv, envp)` 则是将一个新进程从头开始。

gdb 中的 si 是使用 s 执行一条汇编语句，同理 ni 也是使用 n 执行一条汇编语句。

gdb 的 info proc mappings 可以查看寄存器信息

gdb 中使用 `source <file>` 可以导入脚本，jyy 课上展示了使用 python 文件导入脚本。

### mmap

在状态机状态上增加/删除/修改一段可访问的内存，也可以把硬盘里的内容映射成内存。

- MAP_ANONYMOUS: 匿名 (申请) 内存
- fd: 把文件 “搬到” 进程地址空间中 (例子：加载器)
- 更多的行为请参考手册 (复杂性暴增)

```cpp
// 映射
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
int munmap(void *addr, size_t length);

// 修改映射权限
int mprotect(void *addr, size_t length, int prot);
```

我们可以用 pmap 命令查看进程的地址空间 

strace 是 Linux 系统下强大的诊断和调试工具，用于追踪程序执行期间的系统调用(system calls)和接收到的信号(signals)。系统调用是应用程序与操作系统内核交互的接口，通过 strace 我们可以深入了解程序的底层行为。

xxd 是一个 Linux 下的命令行工具，用于将文件或数据转换为十六进制格式显示，类似于十六进制查看器。

`time <command>` 输出一个命令运行时长。

linux 下可以通过修改 `/proc/<pid>/mem` 来修改进程的地址空间。`ptrace` 指令也可以。

