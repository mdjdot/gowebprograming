# HTTP协议简介
	• HTTP 超文本传输协议属于应用层的面向对象的协议
	• 客户端发送给服务器的成为请求报文，服务器发送给客户端的成为响应报文

# HTTP协议的会话方式
	• 浏览器和web服务器的连接过程是短暂的，每次连接只处理一个请求和响应。对每一个页面的访问，浏览器与web服务器都要建立一次单独的连接
	• 浏览器发哦web服务器之间的所有通讯都是完全独立分开的请求和响应对
	客户端	--建立连接-->	服务器
	客户端	--发出请求信息-->	服务器
	客户端	--回送响应信息-->	服务器
	客户端	--关闭连接-->	服务器
	
# HTTP1.0、HTTP1.1、HTTP2.0
	• HTTP2.0只用于https网址
	• HTTPS是SSL（Secure Socket Layer，安全套接字层）上的HTTP，实际就是在SSL/TLS（Transport Layer Security，传输层安全协议）连接的上层进行HTTP通信
	• HTTP1.0版本浏览器请求带图片的网页，会由于下载图片而与服务器之间开启一个新的连接
	• HTTP1.1版本浏览器可以拿到当前请求对应的全部资源再断开连接，提高效率

# 报文
|||
|-|-|
|报文首部|服务端或客户端处理的请求或相应的内容及属性|
|空行（CR+LF）| CR（Carriage Return）回车符（16进制0xd）LF（Line Feed）换行符（16进制0x0a）|
|报文主题|应被发送的数据|
  

|请求报文|
|-|
|请求首行（请求行）；|
|请求头信息（请求头）；|
|空行；|
|请求体；|

get请求没有请求体，post请求有请求体
```
GET https://www.baidu.com/ HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36
Sec-Fetch-User: ?1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Referer: https://www.baidu.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: BIDUPSID=4B97FCCAC082269CFD0F7FDF123FCB0E; PSTM=1577685031; BAIDUID=4B97FCCAC082269CD5A65CBF57E4AD9E:FG=1; BD_UPN=12314753; delPer=0; BD_CK_SAM=1; PSINO=1; H_PS_PSSID=1455_21085_30210_30473_30283_26350_28703; BD_HOME=0; H_PS_645EC=6e8cJeSBvPXBlwH%2B1UAX4u51PaaCOrmQN717W7DZl%2Bl%2F7mcZ9coKEOBbYmI; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; pgv_pvi=9829453824; pgv_si=s9822876672; WWW_ST=1578322241821
```

|||
|-|-|
|GET /Hello/index.jsp HTTP/1.1 ：|GET 请 求 ， 请 求 服 务 器 路 径 为Hello/index.jsp，协议为 1.1；|
|Host:localhost：|请求的主机名为 localhost；|
|User-Agent:|Mozilla/4.0 (compatible; MSIE 8.0…：与浏览器和 OS 相关的信息。有些网站会显示用户的系统版本和浏览器版本信息，这都是通过获取 User Agent 头信息而来的；| 
|Accept: */*：|告诉服务器，当前客户端可以接收的文档类型，*/*，就表示什么都可以接收；|
|Accept-Language:|zh-CN：当前客户端支持的语言，可以在浏览器的工具选项中找到语言相关信息；|
|Accept-Encoding:|gzip, deflate：支持的压缩格式。数据在网络上传递时，可能服务器会把数据压缩后再发送；|
|Connection: keep-alive：|客户端支持的链接方式，保持一段时间链接，默认为 3000ms；|
|Cookie:|JSESSIONID=369766FDF6220F7803433C0B2DE36D98：因为不是第一次访问这个地址，所以会在请求中把上一次服务器响应中发送过来的Cookie 在请求中一并发送过去。|


|响应报文|
|-|
|响应首行（响应行）；|
|响应头信息（响应头）；|
|空行；|
|响应体；|


```
HTTP/1.1 200 OK
Bdpagetype: 1
Bdqid: 0xc40abe18002b19b4
Cache-Control: private
Connection: keep-alive
Content-Type: text/html
Cxy_all: baidu+42775a4ce4bb6ecb1e5943a604571db3
Date: Mon, 06 Jan 2020 14:50:42 GMT
Expires: Mon, 06 Jan 2020 14:50:29 GMT
Server: BWS/1.1
Set-Cookie: delPer=0; path=/; domain=.baidu.com
Set-Cookie: BDSVRTM=0; path=/
Set-Cookie: BD_HOME=0; path=/
Set-Cookie: H_PS_PSSID=1455_21085_30210_30473_30283_26350_28703; path=/; domain=.baidu.com
Strict-Transport-Security: max-age=172800
Traceid: 1578322242280473677814126312191492299188
Vary: Accept-Encoding
X-Ua-Compatible: IE=Edge,chrome=1
Content-Length: 158729

<!DOCTYPE html>
<!--STATUS OK-->
```

```
<html>
<head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
	<meta content="always" name="referrer">
    <meta name="theme-color" content="#2932e1">    
    <title>百度一下，你就知道</title>
<body>

</body> 
</html>
```

```
   HTTP/1.1 200 OK：响应协议为 HTTP1.1，状态码为 200，表示请求成功； 
 Server: Apache-Coyote/1.1：服务器的版本信息； 
 Content-Type: text/html;charset=UTF-8：响应体使用的编码为 UTF-8； 
 Content-Length: 274：响应体为 274 字节； 
 Date: Tue, 07 Apr 2015 10:08:26 GMT：响应的时间，这可能会有 8 小时的时区差；
```

# 响应状态码
	• HTTP1.1协议中定义了5类状态码，
	• 状态码由三位数字组成，第一位表明类型
	1xx	提示信息	表示请求已被成功接收，继续处理
	2xx	成功	表示请求已被成功接收，理解，接受
	3xx	重定向	要完成请求必须进行进一步处理
	4xx	客户端错误	请求有语法错误或请求无法实现
	5xx	服务器端错误	服务器未能实现合法的请求
	
	200	请求成功
	404	请求的资源未找到
	500	请求资源找到了，但服务器内部出现了错误
	302	重定向
