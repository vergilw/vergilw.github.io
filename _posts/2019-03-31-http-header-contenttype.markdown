>终端开发少不了和服务端进行网络交互，在大部分平台成熟的框架里，对于API交互封装的非常好，但是如果你需要自己封装网络层的时候就需要了解HTTP Header里的一些基本概念。本文主要介绍ContentType和Accept。
    
## ContentType
Content-Type用于指示资源的MIME类型。在终端开发中常用的ContentType有三种类型：**application/x-www-form-urlencoded、multipart/form-data、application/json**。

HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。协议规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。服务端会根据的Request Header里的ContenType值，将请求资源进行相应的解码。
    
## HTTP Method
HTTP/1.1 协议规定的 HTTP 请求方法有 OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT。而常用的是**GET、POST、PUT、DELETE、PATCH**，不同的服务端开发框架支持不同，例如PHP和JAVA作为服务端语言，那么你大部分请求都是GET和POST，而如果是Python、Ruby等，那么上述常用的Method可能都会用到。这是由于该语言平台的主流框架所致，后者对于RESTful规范更加友好。
    
#### application/x-www-form-urlencoded
application/x-www-form-urlencoded主要用于GET请求，例如我们在GET请求里提交了一些中文字符，由于HTTP协议以ASCII码传输，并不支持中文字符，此时就会对中文进行encode后再发送给服务端，服务端接受数据decode后才能获取正确的中文字符。
    
#### multipart/form-data
multipart/form-data用于POST、PUT请求，协议规定POST提交的数据必须放在消息主体中，但协议并没有规定数据必须使用什么编码方式。multipart/form-data主要用于数据分片上传，当我们需要提交文件类型数据时使用此方式，当文件过大时，会分成多段上传提高效率。

#### application/json
application/json用于多种Method，在RESTful普及的今天特别常用，因为它能够组装复杂的键值对数据，将数据模型通过JSON序列化以对象的方式传给服务端。典型的使用场景是终端以POST、PUT方式发送JSON序列化对象直接创建数据。

## Accept
Accept 请求头用来告知服务端终端可以处理的内容类型，这种内容类型用MIME类型来表示。它与ContentType不同的是，它可以同时指定多个值。



[Header Content-Type参考文章](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)
[Header Accept参考文章](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)
