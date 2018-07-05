



# HTTP方法

客户端请求的 请求报文 的 请求行，包含HTTP方法


请求方法| 作用| 说明 
---|---|---
==GET== | 获取资源 |
 HEAD | 获取报文首部 | 不返回报文实体主要用于URL的有效性、更新时间
==POST== | 传输实体主体 | 用来传输数据
PUT | 上传文件 | 不带任何验证、任何人都可以上传文件，存在安全问题，完全替换
PATCH | 对资源进行部分更改 | PATCH允许部分修改
DELETE | 删除文件 | 和PUT相反，并且同样不带验证机制
OPTIONS | 查询URL支持的方法，会返回ALLOW: GET,POST,HEAD,OPTIONS

# HTTP状态码

状态码 | 类别 | 原因
---|---|---
1XX | Informational(信息性状态码) | 接收请求正在处理
2XX | Success（成功状态码）| 请求正常处理完毕
3XX | Redirection（重定向） | 需要进行重定向，附加操作处理请求
4XX | Client Error （客户端错误状态码）| 服务端无法或者拒绝处理请求
5XX | Server Error （服务器错误状态码）| 服务器处理出错


## 1XX 信息

- 100 Continue ： 表明目前未知都正常、客户端可以继续发送请求、或者忽略这个响应

## 2XX 成功

- 200 OK
- 204 No Content ： 请求成功处理、但是响应的报文不包含实体部分，一般用于客户端发送消息，服务端不需要返回数据时候使用
- 206 Partial Content ： 表示客户端进行范围请求，响应报文由Content-Range

## 3XX 重定向

- 301 Moved Permanently： 永久性重定向
- 302 Found ：临时性重定向
- 303 See Other ： 302相同，但是客户端必须使用GET方法
- 虽然协议规定301、302 状态下不允许把POST 方法改为GET，但是大多数都会把301、302、032 改为GET
- 304 Not Modified：
- 307 Temporary Redirect：临时性重定向、要求不把POST改成GET

## 4XX 客户端错误

- 400 Bad Request： 
- 401 Unauthorized: 需要验证信息或者验证失败
- 403 Forbidden：请求拒绝
- 404 NOT Found

## 5XX 服务器错误

- 500 ：服务器处理错误
- 501 ：服务器处于超负荷或者正在停机维护，无法处理请求


# HTTP首部

4种类型的首部：通用首部字段、请求首部字段、响应首部字段、实体首部字段

## 通用首部字段


首部字段明  | 说明
---|---
Cache-Control | 控制缓存的行为
Date | 创建报文的日期时间

## 请求首部字段

首部字段明  | 说明
---|---
Accept | 用户可处理的媒体类型
Accept-Charset | 优先字符集
Accept-Encoding | 优先内容编码
Accept-Launguage | 优先的语言
Host | 请求所在的服务器

## 响应首部字段

首部字段明  | 说明
---|---
Location | 重定向的URI

## 实体首部字段

首部字段明  | 说明
---|--- 
Content-Encoding | 实体主体使用的编码方式
Content-Length | 实体主体的大小
Content-MD5 | 实体主体的报文摘要
Content-Type | 实体的媒体类型

# 具体应用

## Cookie

HTTP 是无状态的，HTTP/1.1 服务器发送给浏览器，HTTP/1.1后引入Cookie
来保存状态信息

Cookie 是服务器发送给客户端保存在本地的小数据块，在下一次向同一个服务器的时候会带上Cookie。用于告知服务端两个请求是否来自一个服务器，并保持用户的登录状态。

### 创建过程

服务器的响应报文发送包含Set-Cookie的首部字段，客户端得到响应报文后把Cookie保存到浏览器中


```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[page content]

```

再次向同一个服务器发送请求的时候，会带上Cookie



```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry

```

### 分类

- 绘话期Cookie ： 浏览器关闭后会自动删除
- 持久性Cookie ： 指定一个特定时间（Expires）或者有效期（max-age）


```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;

```

### Secure 和 HttpOnly

- Secure： Cookie只应该被HTTPS协议加密后发送给服务器，但无法保障。
 
- HttpOnly：Cookie不能被JavaScript调用。跨域脚本XSS攻击。常用Document。cookie API 窃取用户的 Cookie信息

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly

```

## Session

存储在服务端，可以存储在服务器的文件、数据库、内存
常见的存储在内存中例如Redis。

使用Session维护登录的过程

1. 登录验证、用户信息存储在Redis中对应ID成为Session ID
2. 返回的响应报文中包含Set-Cookie包含Session ID，收到报文后将Cookie植入浏览器中。
3. 下次访问服务器Cookie会带上SessionID

应该经常更换Session ID，或者验证保障安全

Cookie 只能存储ASCII 码字符串

# 缓存

## 作用

-  缓解服务器压力
-  降低客户端获取资源的延迟

## 实现方法

- 让代理服务器进行缓存
- 让客户端浏览器缓存

## Cache-Control


### 禁止缓存 no-store

```http
Cache-Control: no-store
```

### 私有缓存和公共缓存

- Private： 单独用户使用，一般存储在浏览器中

```http
Cache-Control: private

```
- Public： 多个用户使用，一般存在代理服务器中

```http
Cache-Control: public
```

### 缓存过期时间 

- max-age 资源缓存的时间小于该缓存就可以接收该缓存

```http
Cache-Control: max-age=31536000
```



- Expires 资源过期时间，HTTP/1.1中会优先处理 Cache-Control：max-age指令
1.0中会忽略

```http
Expires: Wed, 04 Jul 2012 08:26:05 GMT
```

### 缓存验证

- ETag： 资源的唯一标识
- If-None-Match： 会将ETag值放入If-None-Match中，如果一致返回304 NOT Modified

- Last-Modified 响应报文中标示资源的最后修改时间，精确到秒

### 短连接和长连接

- 短连接，进行一次通信后会断开连接
- 长连接，一次TCP连接后，可以多次通信


默认是HTTP/1.1长连接 Connection : Keep-Alive

//提前断开
```http
Connection : close；
```

### 流水线

默认请求都是按顺序发出，下一个请求响应后才会被发出。流水线可以请求一起发出，不需要等待响应

### 内容编码

- Accept-Encoding  常用的有gzip、compress、deflate、identity


### 范围请求

- Range 请求报文中的首部字段请求范围

```http
GET /z4d4kWk.jpg HTTP/1.1
Host: i.imgur.com
Range: bytes=0-1023
```

- Accept-Ranges 响应报文中，能响应返回bytes，不能使用none

### 多部分对象集合

一份报文中多种实体同时发送，==boundary== 值进行分隔，每个部分都有首部字段

```
Content-Type: multipart/form-data; boundary=AaB03x

--AaB03x
Content-Disposition: form-data; name="submit-name"

Larry
--AaB03x
Content-Disposition: form-data; name="files"; filename="file1.txt"
Content-Type: text/plain

... contents of file1.txt ...
--AaB03x--
```
# 通信数据转发

## 代理
代理服务器接收客户端的请求，转发给其他的服务器
作用：

- 缓存
- 负载均衡
- 网络访问控制
- 访问日志记录

正向代理和反向代理

## 网关

网关服务器将http协议转为其他的协议进行通信、从而请求其他非http服务器的服务

## 隧道
使用SSL加密，为客户端提供一条安全的通信道路



# GET 和 POST的区别

## 作用

GET 用于获取资源，POST用于传输主体

## 参数

GET 查询参数出现在URL中，而POST参数存储在实体主体中

```http
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1

```

```http
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2

```
## 幂等性

GET是幂等的，POST是非幂等的


