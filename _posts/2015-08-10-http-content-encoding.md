---
layout: post
title: http 内容编码
categories: http
tags: 笔记 http
---

HTTP 应用程序在发送之前需要对内容进行编码，如果我们返回的 HTML 内容很大，我们需要进行编码来减少传输的数据，从而提高效率。

## 编码过程

1. 网站服务器生成原始响应报文，其中有原始的 Content-Type 和 Content-Length
2. 内容编码服务器创建编码后的服务器。
3. 接受程序得到编码后的报文，进行解码，获得原始报文。

下面给出一个响应例子：

```
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 10 Aug 2015 13:53:10 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Content-Encoding: gzip
[...]
```

## 内容编码类型

HTTP 定义了一些标准的内容编码类型。

Content-Encoding 值|描述
:--|:--
gzip|GNU zip 编码
compress|unix文件压缩
deflate|zlib压缩
identity|表示没进行编码，当没有 Content-Encoding 响应头就默认这个

前三个都是无损压缩，gzip 效率最高，使用最广。

## Accept-Encoding

我们不希望服务端使用客户端不支持的方式进行编码，为了避免这样的情况发生，客户端就把自己支持的内容编码方式放在请求的 Accept-Encoding 头当中。如果 HTTP 请求中没有包含 Accept-Encoding 头，服务器就可以假设客户端能够接受任何编码方式。

下面是例子：

```
Accept-Encoding: compress, gzip
Accept-Encoding: *
Accept-Encoding: compress;q=0.5, gzip;q=1.0
Accept-Encoding: gzip;q=1.0, identity;q=0.5, *;q=0
```

客户端可以给每种编码附带 Q(质量) 值参数来说明编码的优先级。Q 值的范围在 0.0 - 1.0，0.0 说明客户端不想接受所说明的编码，1.0 说明最希望使用的编码。

`*` 表示其他方法，identity 编码代号只能出现在 Accept-Encoding 首部，客户端用它来说明相对于其他内容编码算法的优先级。

【参考资料】

1. [http 权威指南](http://book.douban.com/subject/10746113/)

---EOF---

