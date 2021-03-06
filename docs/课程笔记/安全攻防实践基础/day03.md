# 第 3 天：逆向工程

## 为什么要逆向工程？

- 不开源
- 寻找商业机密
- 找到漏洞
- 破解，修改
- 分析协议
- 好奇心……

## 开始

阅读汇编代码如同大海捞针……并且还有驱动、库文件等乱七八糟的干扰项。

想要逆向？它们还有防御机制。

程序的被动防护：加壳、混淆……

程序的主动防护：检测逆向工具、反调试手段……

## 怎么变成一个好的逆向打手呢？

### 工具

#### 静态分析工具

- 反编译器
- Hex Editor（010Editor）

#### 动态分析工具

- Debugger

## IDA

（我超，我学汇编的时候怎么就没有这么强大的工具！？）

### 快捷键

在程序图和汇编代码之间切换：++space++。

按汇编代码生成一份（pseudocode）C 伪代码：++f5++。

伪代码并不一定是完全正确的，可能有各种各样的错误。比如把函数的传参分析错。

给伪代码的字段重命名：++n++。

修改伪代码函数的定义：++y++。

查找所有对于这个变量 / 函数的引用：++x++。

> `dq` 长度为 8 字节。就是 quadra (4) 个 word。

更多子功能：`Views` -> `Open Subviews`。

查找 strings：++shift+f12++。

修改汇编代码：`Edit` -> `Patch Program` -> `Assemble`。

保存：`Edit` -> `Patch Program` -> `Apply Patches...`。

给 struct 添加成员变量：++d++。在 IDA 没有正确识别出 struct 的时候，得手动添加。

> LODWORD，HIDWORD 等：给个 8 字节的变量的低 4 个字节、高 4 个字节赋值。

把解析设成 undefined：++u++。

让 IDA 解释成代码段：++c++。即 code。

让 IDA 把代码段当成一个函数来分析：++p++。

跳转到地址：++g++。

### 动态调试

软件断点：在代码段里面的断点。

硬件断点：可以检测某个变量的变化的断点。

### 使用 API 给的接口进行修改

比如，当源代码进行了一个混淆，就可以写个 python 把那些内容混淆回来。

## x96dbg

= x32dbg + x64dbg

## 今天的作业

!!! warning "警告"
	抄袭行为是严厉禁止的。

### Reverse1

??? done "提示"
	IDA 和 IDA64 不是同一个东西。

### attachment

??? done "提示"
	加密是可逆的。

	另外，flag 格式确实是 `flag{...}`。你得到的东西和它有什么关系呢？

### Reverse2

??? done "提示"
	IDA 的 string 功能好像还不够好。

	那个 MessageBox 有两个传参分别是窗口标题和窗口内容。与窗口标题和窗口内容在一块儿的，一般是什么呢？

### Start

!!! bug "请帮帮我！"
	main 函数已经找到 3 个了，其中一个是钓鱼的，一个有 bug，剩下一个里面有非常可怕的加密过程。

你也不能指望一个第一天才会写 Hello World 的人去打 NOIP 吧？

题解没有就算了，连个 Hint 都不给，做你妈呢。
