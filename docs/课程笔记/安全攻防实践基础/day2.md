# 第 2 天：二进制基础

来自：《程序员的自我修养》

## 编译

主流编译器：gcc，Windows Compiler (cl.exe)，clang，……

对于 gcc：

hello.c + \*.h -(cpp)-> hello.i -(gcc)-> hello.s -(as)-> hello.o

hello.o + \*.a + \*.so -(ld)-> ./a.out

## ELF 文件

`objdump -d *.o`

ELF 文件由一大片汇编（二进制）代码和一些其他杂七杂八的东西组成。

File offset：这一段在 .elf 文件中的偏移。

.data 段：已经初始化了的全局静态变量和局部静态变量。

.rodata 段：只读的数据（比如字符串常量）。

.bss 段：未初始化的全局静态变量和局部静态变量。

（C 语言编译器会把全局符号分成 strong 和 weak 两类：函数和初始化过的全局符号为 strong，没有初始化的全局符号是 weak。.bss 段存没初始化的那一部分。）

.comment 段：编译器的信息。

## 静态链接

### 空间与地址分配

- 扫描所有的输入目标文件，获得它们的各个段的各种属性。
- 收集所有目标文件中的符号定义和符号，引用到一个全局符号表。
- 合并所有段，并建立映射关系。

### 符号解析和重定位

- 以上一步收集的信息，读取输入文件中，段的数据和重定位信息，进行符号解析和重定位。

#### 重定位表

#### 符号解析

每个目标文件都可能定义了一些符号，也可能引用了定义在其他目标文件的符号。

重定位的时候，链接器会在全局符号表找到相应的符号进行重定位。

## 虚拟地址空间

虚拟地址空间（Virtual Addres Space）是操作系统给每一个进程提供的一坨**私有的**“内存”。这些“内存”指向的数据是当前进程需要的所有数据信息，但是它们不一定都在真实的内存中。内存管理单元（Memory Management Unit，MMU）负责这“内存”和内存的转换。

在内存不够用的时候，MMU 会把当前内存的某些东西扔进磁盘中。等需要它们的时候再拿出来。这样就可以更加好地运行多个程序了。

## 装载

- 可执行文件：
	- 是静态的
	- 预先编译好的指令和数据的集合
	- 在磁盘里面
- 进程：
	- 是动态的
	- 程序运行时候的一个过程
	- 在内存里面

每个程序跑起来后，会得到自己的一个独立虚拟地址空间。32 位下的空间为 `0x0` 到 `0xffff ffff`。（所以 32 位下的指针大小是 4 个字节。）

### 装载方式

#### 全部装入

- 家里有矿啊？
- 所需内存大于物理内存……？

#### 动态装载

不要的模块丢到磁盘中。

##### Overlay

Overlay（基本上被淘汰）：程序员自己控制程序的装载和卸载。（这很费程序员。）

通过各种模块互相不同时存在的特性，可以建一棵树来对这些模块的地址进行尽量可以复用的安排。

##### 分页（Paging）

分页（Paging）：把内存和所有磁盘中的数据和指令按照“页”为单位进行划分。之后装载和操作就都是一页一页来的。页的大小可以是 4 KiB，8 KiB，2 MiB，4 MiB 等。

### 进程的建立

- 创建一个独立的虚拟地址空间。
- 读取 ELF 文件头，建立虚拟空间和可执行文件的映射关系。
- 把 CPU 指令设置成可执行文件的入口地址，位于文件头。

#### 页错误

上面三步执行完后，可执行文件还没有被装载到物理内存中。CPU 会发现这个问题，并触发页错误。

此时，操作系统通过第二步中的数据结构，在物理内存中分配页面，之后让进程继续执行。

### 页的划分

以 ELF 文件中的段进行划分：空间浪费严重。

根据段的权限进行划分：可读可执行的；可读可写的；只读的；……

### 堆和栈

#### 栈初始化

进程在刚开始启动的时候，需要知道自己运行的环境。

- 环境变量
- 进程的运行参数

这些东西是被压到栈里面的。

#### 堆初始化

如果每次都用系统调用的话，那么这样的开销是很大的，会影响性能。

所以运行库先提前向操作系统申请一块较大的堆空间，程序需要的时候再从这里面拿空间给它。

程序的运行库管理堆的分配，glibc 下有复杂的堆分配算法。

#### brk

`int brk(void *end_data_segment)`

堆的地址是从 start_brk 到 brk 的，brk 就是 program break 的意思。

brk() 通过把 brk 的地址往上移来增加堆的大小。

ASLR（地址空间配置随机加载）关闭时，start_brk 指向数据段的末尾。

ASLR 开启时，start_brk 和数据段末尾之间有一段随机的 brk offset。

#### mmap

`void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset)`

mmap 申请一块新内存，并由此创建一个私有而匿名的映射段。

`start`：起始位置。如果置 0 就是让系统自己决定。

`length`：长度。

`prot, start`：空间的权限和映射类型。

## 动态链接

静态链接的缺点：

- 浪费内存！
- 程序的更新和发布会很麻烦。

而动态链接这么做：

- 把程序的模块分割开来，形成独立的文件。
- 让链接在运行时再运行。

Linux 下：Dynamic Shared Objects (.so)

Windows 下：Dynamic Linking Library (.dll)

然而由于链接过程发生在装载的时候，每次装载的时候都要链接一下，这会影响性能。不过大家有方法对它进行优化。

### 模块

Lib.c -(Compiler)-> Lib.o

Lib.o + C Runtime Library -(Linker)-> Lib.so

Program1.c -(Compiler)-> Program1.o

Program1.o + (Lib.so -> Stub) -(Linker)-> Program1

### 地址冲突

#### 装载时重定位（Rebasing）

把模块里面的所有重定位项暴力修正一下。比如本来应该是 0x100，却被分到了 0x1000，那就给所有重定位项加一个 0x900 就好了。

#### 地址无关代码

- 模块内外的函数调用、跳转
- 模块内外的数据访问

### 模块间的数据 / 函数访问：全局偏移表（GOT 表）

其他模块的全局变量地址是和模块的装载地址有关的。为此，ELF 建立了一个指向这些变量的指针数组，即 GOT 表。

比如在 .data 段里面有个 `int b = 100` 在 0x20002000，在 .text 段里面有个 `void ext()` 在 0x20001000，那么 GOT 表里面就存了 b 和 ext() 的真实地址。

其他家伙使用的时候，就会先 jmp 到 GOT 表里面的对应位置。

#### 延迟绑定（Lazy Binding，PLT）

动态链接下，函数第一次被用到的时候才给它绑定一下。

为了实现它，调用函数的时候，在原函数和 GOT 表之间还有一个 PLT 表。

```asm
bar@plt:
	jmp *(bar@GOT)
	push n
	push moduleID
	jmp _dl_runtime_resolve
```

初始化前，*(bar@GOT) 暂时是它下一条指令的位置。也就是这个 jmp 指令和一个空指令一样。

最后调用的 _dl_runtime_resolve 才会把 bar 函数真正的地址写到 bar@GOT 里面。

#### 数据段地址无关性

```cpp
static int a;
static int *p = &a;
```

a 的地址会随着装载的时候的地址而改变，而 p 指向的绝对地址也会一起改变。

此时编译器和链接器又会产生一个重定位表。

## 从 _start 开始

_start -> __libc_start_main -> exit -> _exit

main 函数就在 __libc_start_main 里面。

大概关系如下：

```cpp
void _start()
{
	%ebp = 0;
	int argc = (pop from stack);
	char **argv = (top of stack);
	__libc_start_main(main, argc, argv,
		__libc_csu_init, __libc_csu_fini, edx, (top of stack));
}

int __libc_start_main(
	int (*main) (int, char **, char **),
	int argc,
	char * __unbounded *__unbounded ubp_av,
	__typeof (main) init,
	void (*fini) (void),
	void (*rtld_fini) (void),
	void * __unbounded stack_end) {/*...*/}

void exit (int status)
{
    while (__exit_funcs != NULL)
	{
        // __exit_funcs是存储由__cxa_atexit和atexit注册的函数链表
        // 执行注册函数
        __exit_funcs = __exit_funcs->next;
    }
    /*...*/
    _exit(status);
}
```

## 今天的作业

!!! warning "警告"
	抄袭行为是严厉禁止的。

这才第二天哪，哪吃得下这种屎。

### 1. 在 linux 上编译、链接、装载程序

\*.c -(gcc -E)-> expanded source -(gcc -s)-> \*.s -(as)-> \*.o

然而 \*.o 文件里面还没把标准库里面的那些东西链接进去。用 gcc -o 链接。

### 2. 3. 手撕 ELF

给的 struct 里面的顺序就是 ELF 文件中那些东西排列的顺序。

### 挑战部分

#### Makefile

.PHONY 有什么用？假设你有个方法是 make love，然后如果你的工作目录下出现了一个叫 love 的文件，那当你想要 make love 的时候，make 就会因为 love 已经存在而不 make love。

这个时候，如果你写一个 .PHONY:love 在前面，那么 make 就不管有没有 love 都 make love。因为 .PHONY 就是 phonytargets 的意思。你可以理解成 .PHONY:love 就是告诉 make，love 总是不存在。
