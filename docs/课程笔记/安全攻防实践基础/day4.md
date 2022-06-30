# 第 4 天：PWN 基础导论

（PWN：一个拟声词，表示攻破了设备或者系统。“砰”。）

## GCC 的插件 GEF

GEF 增强了 GDB。

- si：单步调试，进入函数。
- ni：单步调试，但不进入函数。
- s
- n
- vmmap：虚拟内存映射情况。
- disass：找出某个函数的汇编代码。

> 让 gcc 编译一个 32 位程序出来：`-m32`。
>
> 让 gcc 编译成静态链接：`-static`。
>
> gdb attach：让 gdb 直接挂载到一个正在运行的程序上。

## 系统调用

查看系统调用：`strace ./*`。

## PWN 是什么？

逆向 -(reverse)-> 找洞 -(exploit part)-> 利用

reverse：给逆向打手吧……

exploit part：充满技术和经验的过程

### 洞？

“洞”就是程序员**不期待**程序中出现的事情。或者说，bug。

```cpp
int a, b, c[10];
cin >> a >> b;
c[10] = a / b;
```

然而，编译器并不总是那么聪明……

```cpp
c[getchar()] = a / b;
```

> 低情商：我写了一个大 bug！
>
> 高情商：我写了一个好 pwn 题！

### Bug 的成因

没学明白：

```cpp
long x = 0x12345678;
printf("%s", x);
```

天天越界：

```cpp
int val[16];
for(int i = 0; i <= 16; i++) { /*...*/ }
```

并且，还有各种 corner case，非常复杂的大型应用（浏览器），更新迭代过程中的“顾己失彼”……

### 当 Bug 变成 Vulnerability（漏洞）

小的安全 bug：

- local Denial of Service
- local information leak

大的安全 bug：

- local privilege escalation
- remote Denial of Service
- remote infomation leak
- remote code execution

### PWN 在 CTF

出题人会在 target 程序中引入 bug，让选手通过它找到 flag。

> 但是，如果有出题人没意识到的 bug……

## 栈溢出

### 栈布局

```cpp
void func(int a, int b)
{
	int x, y;
	x = a + b;
	y = a - b;
}
```

理论上的栈布局：

```plain
高地址
---
Value of b					\ 参数
Value of a					/ arguments
Return Address
Previous Frame Pointer
Val of x						\ 局部变量
Val of y						/ local variables
↓↓↓
低地址
```

这叫做一个函数的栈帧（stack frame），或者 activation frame、activation record。

### 栈指针 Stack pointer

32 位（x86)：esp。

64 位（x64)：rsp。

arm32，aarch64：sp。

### 帧指针 Frame pointer

32 位：ebp。

64 位：rbp。

arm32，aarch64：fp。

一般用于访问参数和变量。

比如对于上面那个：x 在 %ebp - 8，a 在 %ebp + 8，b 在 %ebp + 12。

> leave = move $ebp, $esp; pop $ebp

### cdecl 调用约定

32 位：参数从“右”往“左”丢到栈里面。

64 位：寄存器 rdi、rsi、rdx、rcx、r8、r9 会优先承担传参功能，返回值放在 rax。

ARM：寄存器 r0 – r3 (= a1 – a4) 会优先承担传参功能，返回值放在 r0。

AArch64：寄存器 r0 – r7 优先承担传参功能，返回值放在 r0。

### 函数的调用链

```cpp
b() {...}
a() { b(); }
main { a(); }
```

```plain
低地址
↑↑↑
未分配
b() 的栈帧
a() 的栈帧
main() 的栈帧
---
高地址
```

### 爆栈 Stack Buffer Overflow

CTF 中常用的栈溢出情形：

- gets()
- 整型符号问题

#### 符号问题的例子

```cpp
int inputSize;
char buffer[32];
scanf("%d", &inputSize);
if(inputSize >= 32)
{
	puts("size too large");
	return 1;
}
fgets(buffer, inputSize, stdin);
```

然而，inputSize 是一个有符号数……

### 爆栈的危害

栈上面有什么？

- 临时变量
- 帧指针
- 返回地址
- 传参（64 位的话，也许没有）

那这个栈溢出漏洞威力有多少？

- 能溢出多少字节？
- 溢出部分的可控性？

## 交互：pwntools

pwntools 和可执行程序之间进行交互，并提供一些很方便的功能（比如，给它们不可见字符）：

```python
>>> from pwn import *
>>> p = process(...)
>>> payload = b"\xff" * 128
>>> p.sendline(payload)
>>> p.recv(128)
>>> p.interactive()
```

### 远程交互

```python
>>> p = remote("10.214.160.13", 11002)
>>> p.recv() # p.recvline()
>>> ...
>>> p.send(...)
```

## 今天的作业

!!! warning "警告"
	抄袭行为是严厉禁止的。

### Overflow 例子

#### 第 1 波

??? done "提示"
    ```shell
    gef➤  p &buffer
    $3 = (char (*)[32]) 0x7fffffffde10
    gef➤  p &rootuser
    $4 = (int *) 0x7fffffffde3c
    ```

#### 第 2 波

??? done "提示"
    返回地址是在什么时候插入的呢？插在了哪里呢？

#### 第 3 波

??? done "提示"
    ```shell
    gef➤  p &backdoor
    $1 = (void (*)()) 0x401176 <backdoor>
    gef➤  p &main
    $2 = (int (*)(int, char **)) 0x4011b9 <main>
    ```

#### 第 4 波

??? done "提示"
    buffer[32] 的第 buffer[16] 到 buffer[23] 会填充到函数 bof 的栈帧的 $rbp。