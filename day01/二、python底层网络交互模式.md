[TOC]



python 的底层网络交互模块有哪些? 

#一、Socket

### Socket概念

Socket的中文翻译是套接字，它是TCP/IP网络环境下应用程序与底层通信驱动程序之间运行的开发接口，它可以将应用程序与具体的TCP/IP隔离开来，使得应用程序不需要了解TCP/IP的具体细节，就能够实现数据传输。
       在网络应用程序中，Socket通信是基于客户端/服务器结构。客户端是发送数据的一方。服务器时刻准备接受来自客户端的数据，对做出响应
### 实现基于tcp网络通信
- 服务器程序首先与客户机程序启动，步骤和调用函数：
1. 调用socket()函数创建一个流式套接字，返回套接字号s
2. 调用bind()将s绑定到已知地址，通常为本地ip
3. 调用listen()将s设为监听模式，准备接收来自客户端得连接请求
4. 调用accept()等待接受客户端连接请求
5. 如果接受到客户端请求，则accept()返回，得到新的套接字ns
6. 调用rev()接收来自客户端数据，调用send()向客户端发送数据
7. 与客户端通信结束，服务器可以调用shutdown()对方不再接收和发送数据，也可以由客户端程序断开连接，断开连接后，服务器进程调用closescoket()关闭套接字ns，此后服务器返回第四步
8. 如果要退出服务器程序，则调用closesocket()关闭最初得套接字s

| Socket编程的层次结构 |
| -------------------- |
| 应用层               |
| Socket开发接口       |
| 传输层               |
| TCP、UDP             |
| 网络层               |
| IP                   |
| 驱动                 |
| 物理层               |
- 客服端程序步骤以及调用函数：
1. 调用WSAStartup()函数加载Windows Sockets动态库，然后调用socket()函数创建一个流式套接字，返回套接字号s。
2. 调用connect()函数将套接字s连接到服务器。
3. 调用send()函数向服务器发送数据，调用recv()函数接收来自服务器的数据。
4. 与服务器的通信结束后，客户端程序可以调用close()函数关闭套接字。
- 相关函数：
1. socket()函数用于创建与指定的服务提供者绑定套接字，函数原型如下：
  socket=socket.socket(familly,type)

  参数说明如下：

  familly，指定协议的地址家族，可为AF_INET或AF_UNIX。AF_INET家族包括Internet地址，AF_UNIX家族用于同一台机器上的进程间通信。
  type，指定套接字的类型。
  | 套接字类型  | 说明                                                        |
  | ----------- | ----------------------------------------------------------- |
  | SOCK_STREAM | 提供顺序、可靠、双向和面向连接的字节流数据传输机制，使用TCP |
  | SOCK_DGRAM  | 支持无连接的数据报，使用UDP                                 |
  | SOCK_RAW    | 原始套接字，可以用于接收本机网卡上的数据帧或者数据包        |
2. bind（）函数
  bind()函数可以将本地地址与一个Socket绑定在一起，函数原型如下：
  socket.bind( address )
  参数address是一个双元素元组，格式是(host,port)。host代表主机，port代表端口号。
3. listen（）函数
  listen()函数可以将套接字设置为监听接入连接的状态，函数原型如下：

  listen(backlog);

  参数backlog指定等待连接队列的最大长度。
4. accept（）函数
  在服务器端调用listen()函数监听接入连接后，可以调用accept()函数来等待接受连接请求。accept()的函数原型如下：

  connection, address = socket.accept()

  调用accept()方法后，socket会进入waiting状态。客户请求连接时，accept()方法会建立连接并返回服务器。accept()方法返回一个含有两个元素的元组(connection,address)。第一个元素connection是新的socket对象，服务器必须通过它与客户通信；第二个元素 address是客户的Internet地址
5. recv（）函数
  调用recv()函数可以从已连接的Socket中接收数据。recv()的函数原型如下：

  buf = sock.recv(size)

  参数sock是接收数据的socket对象，参数size指定接收数据的缓冲区的大小。recv()的函数的返回接收的数据
6. send（）函数
  调用send()函数可以在已连接的Socket上发送数据。send()的函数原型如下：

  sock.recv(buf)

  参数sock是在已连接的Socket上发送数据。参数buf是也要已连接的Socket上发送数据。
7. close（）函数
  close ()函数用于关闭一个Socket，释放其所占用的所有资源。socket()的函数原型如下：

  s.closesocket();

  参数s表示要关闭的Socket。

### 使用socket通讯的简易服务器 

```python
import socket
if __name__ == "__main__":
    #创建socket对象s，基于internet地址和tcp协议
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    #绑定到本地的8001端口
    s.bind(("localhost",8001))
    #在本地8001端口监听，等待连接队列最大长度为5
    s.listen(5)
    print("等待连接")
    while True:
        #接收来自客户端的连接
        connection,address = s.accept()
        try:
            connection.settimeout(5)
            buf = connection.recv(1024).decode('utf-8')#接收客户端消息
            if buf=='1':
                connection.send(b'welcome to server')
            else:
                connection.send(b'please go out')
        except s.timeout:
            print('time out')

```

### 使用socket进行通信的简易客户端 

```python
mport socket
import time
if __name__ == '__main__':
    s =socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(('localhost',8001))
    #推迟执行
    time.sleep(2)
    s.send(b'1')
    print(s.recv(1024).decode('utf-8'))
```

### **基于TCP的Socket编程** 

```python
import socket
#创建UDP SOCKET
s = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
port = 8000 #服务器端口
host = '192.168.0.101'#服务器地址
while True:
    msg = input()# 接受用户输入
    if not msg:
       break
    # 发送数据
    s.sendto(msg.encode(),(host,port))
s.close()
```

```python
import socket
#创建基于UDP的socket对象用socket.SOCK_DGRAM
s = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
s.bind(('localhost',8000))

#循环调用recvfrom（）接收客户端发送来的数据

while True:

    data,addr = s.recvfrom(1024)
    if not data:
        print('client has exited!')
        break
   print('received:',data,'from',addr)
s.close()
```

# 二、urllib模块

### urllib介绍

我们首先了解一下 Urllib 库，它是 Python 内置的 HTTP 请求库，也就是说我们不需要额外安装即可使用，它包含四个模块：

- 第一个模块 request，它是最基本的 HTTP 请求模块，我们可以用它来模拟发送一请求，就像在浏览器里输入网址然后敲击回车一样，只需要给库方法传入 URL 还有额外的参数，就可以模拟实现这个过程了。
- 第二个 error 模块即异常处理模块，如果出现请求错误，我们可以捕获这些异常，然后进行重试或其他操作保证程序不会意外终止。
- 第三个 parse 模块是一个工具模块，提供了许多 URL 处理方法，比如拆分、解析、合并等等的方法。
- 第四个模块是 robotparser，主要是用来识别网站的 robots.txt 文件，然后判断哪些网站可以爬，哪些网站不可以爬的，其实用的比较少。
  在这里重点对前三个模块进行下讲解。

#### 一、发送请求

使用 Urllib 的 request 模块我们可以方便地实现 Request 的发送并得到 Response

- ##### 1、urlopen()

urllib.request 模块提供了最基本的构造 HTTP 请求的方法，利用它可以模拟浏览器的一个请求发起过程，同时它还带有处理authenticaton（授权验证），redirections（重定向)，cookies（浏览器Cookies）以及其它内容。
我们来感受一下它的强大之处，以 Python 官网为例，我们来把这个网页抓下来：

```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.read().decode('utf-8'))
```

接下来我们看下它返回的到底是什么，利用 type() 方法输出 Response 的类型。 

```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(type(response))
```

输出结果如下： 

```python
<class 'http.client.HTTPResponse'>
```

通过输出结果可以发现它是一个 HTTPResposne 类型的对象，它主要包含的方法有 read()、readinto()、getheader(name)、getheaders()、fileno() 等方法和 msg、version、status、reason、debuglevel、closed 等属性。
得到这个对象之后，我们把它赋值为 response 变量，然后就可以调用这些方法和属性，得到返回结果的一系列信息了。+

例如调用 read() 方法可以得到返回的网页内容，调用 status 属性就可以得到返回结果的状态码，如 200 代表请求成功，404 代表网页未找到等。

下面再来一个实例感受一下： 

```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```

```python
200
[('Server', 'nginx'), ('Content-Type', 'text/html; charset=utf-8'), ('X-Frame-Options', 'SAMEORIGIN'), ('X-Clacks-Overhead', 'GNU Terry Pratchett'), ('Content-Length', '47397'), ('Accept-Ranges', 'bytes'), ('Date', 'Mon, 01 Aug 2016 09:57:31 GMT'), ('Via', '1.1 varnish'), ('Age', '2473'), ('Connection', 'close'), ('X-Served-By', 'cache-lcy1125-LCY'), ('X-Cache', 'HIT'), ('X-Cache-Hits', '23'), ('Vary', 'Cookie'), ('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')]
nginx
```

可见，三个输出分别输出了响应的状态码，响应的头信息，以及通过调用 getheader() 方法并传递一个参数 Server 获取了 headers 中的 Server 值，结果是 nginx，意思就是服务器是 nginx 搭建的。 利用以上最基本的 urlopen() 方法，我们可以完成最基本的简单网页的 GET 请求抓取。 如果我们想给链接传递一些参数该怎么实现呢？我们首先看一下 urlopen() 函数的API： 

```python
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
```

可以发现除了第一个参数可以传递 URL 之外，我们还可以传递其它的内容，比如 data（附加数据）、timeout（超时时间）等等。 下面我们详细说明下这几个参数的用法: 

- ##### data参数

data 参数是可选的，如果要添加 data，它要是字节流编码格式的内容，即 bytes 类型，通过 bytes() 方法可以进行转化，另外如果传递了这个 data 参数，它的请求方式就不再是 GET 方式请求，而是 POST。

下面用一个实例来感受一下：

```python
import urllib.parse
import urllib.request

data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf8')
response = urllib.request.urlopen('http://httpbin.org/post', data=data)
print(response.read())
```

在这里我们传递了一个参数 word，值是 hello。它需要被转码成bytes（字节流）类型。其中转字节流采用了 bytes() 方法，第一个参数需要是 str（字符串）类型，需要用 urllib.parse 模块里的 urlencode() 方法来将参数字典转化为字符串。第二个参数指定编码格式，在这里指定为 utf8。 

- ##### timeout参数

timeout 参数可以设置超时时间，单位为秒，意思就是如果请求超出了设置的这个时间还没有得到响应，就会抛出异常，如果不指定，就会使用全局默认时间。它支持 HTTP、HTTPS、FTP 请求。 因此我们可以通过设置这个超时时间来控制一个网页如果长时间未响应就跳过它的抓取，利用 try except 语句就可以实现这样的操作，代码如下： 

```pyhon
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

- ##### 其他参数

还有 context 参数，它必须是 ssl.SSLContext 类型，用来指定 SSL 设置。 cafile 和 capath 两个参数是指定 CA 证书和它的路径，这个在请求 HTTPS 链接时会有用。 cadefault 参数现在已经弃用了，默认为 False。 以上讲解了 urlopen() 方法的用法，通过这个最基本的函数可以完成简单的请求和网页抓取，如需更加详细了解，可以参见官方文档：<https://docs.python.org/3/library/urllib.request.html>。 

- #### 2、Request

由上我们知道利用 urlopen() 方法可以实现最基本请求的发起，但这几个简单的参数并不足以构建一个完整的请求，如果请求中需要加入 Headers 等信息，我们就可以利用更强大的 Request 类来构建一个请求。 首先我们用一个实例来感受一下 Request 的用法： 

```python
mport urllib.request

request = urllib.request.Request('https://python.org')
response = urllib.request.urlopen(request)
print(response.read().decode('utf-8'))
```

可以发现，我们依然是用 urlopen() 方法来发送这个请求，只不过这次 urlopen() 方法的参数不再是一个 URL，而是一个 Request 类型的对象，通过构造这个这个数据结构，一方面我们可以将请求独立成一个对象，另一方面可配置参数更加丰富和灵活。 下面我们看一下 Request 都可以通过怎样的参数来构造，它的构造方法如下： 

```python
class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
```

第一个 url 参数是请求 URL，这个是必传参数，其他的都是可选参数。
第二个 data 参数如果要传必须传 bytes（字节流）类型的，如果是一个字典，可以先用 urllib.parse 模块里的 urlencode() 编码。
第三个 headers 参数是一个字典，这个就是 Request Headers 了，你可以在构造 Request 时通过 headers 参数直接构造，也可以通过调用 Request 实例的 add_header() 方法来添加, Request Headers 最常用的用法就是通过修改 User-Agent 来伪装浏览器，默认的 User-Agent 是 Python-urllib，我们可以通过修改它来伪装浏览器。
第四个 origin_req_host 参数指的是请求方的 host 名称或者 IP 地址。
第五个 unverifiable 参数指的是这个请求是否是无法验证的，默认是False。意思就是说用户没有足够权限来选择接收这个请求的结果。例如我们请求一个 HTML 文档中的图片，但是我们没有自动抓取图像的权限，这时 unverifiable 的值就是 True。
第六个 method 参数是一个字符串，它用来指示请求使用的方法，比如GET，POST，PUT等等。

写个例子：

```python
from urllib import request, parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

在这里我们通过四个参数构造了一个 Request，url 即请求 URL，在headers 中指定了 User-Agent 和 Host，传递的参数 data 用了 urlencode() 和 bytes() 方法来转成字节流，另外指定了请求方式为 POST。 通过观察结果可以发现，我们成功设置了 data，headers 以及 method。 另外 headers 也可以用 add_header() 方法来添加。

```python
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
```

 如此一来，我们就可以更加方便地构造一个 Request，实现请求的发送啦。 

#### 二、 处理异常

我们了解了 Request 的发送过程，但是在网络情况不好的情况下，出现了异常怎么办呢？这时如果我们不处理这些异常，程序很可能报错而终止运行，所以异常处理还是十分有必要的。+

Urllib 的 error 模块定义了由 request 模块产生的异常。如果出现了问题，request 模块便会抛出 error 模块中定义的异常。

主要有这两个处理异常类，`URLError`,`HTTPError`
下面写个例子：

```python
from urllib import request, error
try:
    response = request.urlopen('http://cuiqingcai.com/index.htm')
except error.URLError as e:
    print(e.reason)
```

我们打开一个不存在的页面，照理来说应该会报错，但是这时我们捕获了 URLError 这个异常，运行结果如下： 

```python
Not Found
```

```pytho
from urllib import request,error
try:
    response = request.urlopen('http://cuiqingcai.com/index.htm')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, seq='\n')
```

运行结果： 

```python
Not Found
404
Server: nginx/1.4.6 (Ubuntu)
Date: Wed, 03 Aug 2016 08:54:22 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: close
X-Powered-By: PHP/5.5.9-1ubuntu4.14
Vary: Cookie
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Pragma: no-cache
Link: <http://cuiqingcai.com/wp-json/>; rel="https://api.w.org/"
```

HTTPError,它有三个属性。

- code，返回 HTTP Status Code，即状态码，比如 404 网页不存在，500 服务器内部错误等等。
- reason，同父类一样，返回错误的原因。
- headers，返回 Request Headers。
  因为 URLError 是 HTTPError 的父类，所以我们可以先选择捕获子类的错误，再去捕获父类的错误，所以上述代码更好的写法如下：

```python
from urllib import request, error

try:
    response = request.urlopen('http://cuiqingcai.com/index.htm')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, sep='\n')
except error.URLError as e:
    print(e.reason)
else:
    print('Request Successfully')
```

这样我们就可以做到先捕获 HTTPError，获取它的错误状态码、原因、Headers 等详细信息。如果非 HTTPError，再捕获 URLError 异常，输出错误原因。最后用 else 来处理正常的逻辑，这是一个较好的异常处理写法。 











