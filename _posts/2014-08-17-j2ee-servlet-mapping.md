---
layout: post
title: 映射请求到servlet
categories: j2ee
tags: j2ee sevlet
---

### servlet规范当中对映射请求的描述：

在收到客户端请求时，web 容器确定转发到哪一个Web应用。选择的Web应用必须具有最长的上下文路径匹配请求URL的开始。
当映射到Servlet时，URL匹配的一部分是上下文。Web 容器接下来必须用下面描述的路径匹配步骤找出servlet来处理请求。
用于映射到Servlet的路径是请求对象的请求URL减去上下文和路径参数部分。
下面的URL路径映射规则按顺序使用。
使用第一个匹配成功的且不会进一步尝试匹配：

1. 容器将尝试找到一个请求路径到servlet路径的精确匹配。成功匹配则选择该servlet。
2. 容器将递归地尝试匹配最长路径前缀。这是通过一次一个目录的遍历路径树完成的，使用‘/’字符作为路径分隔符。最长匹配确定选择的servlet。
3. 如果URL最后一部分包含一个扩展名（如.do），servlet容器将视图匹配为扩展名处理请求的Servlet。
4. 扩展名定义在最后一部分的最后一个‘.’字符之后。

如果前三个规则都没有产生一个servlet匹配，容器将试图为请求资源提供相关的内容。如果应用中定义了一个“default”servlet，它将被使用。许多容器提供了一种隐式的default servlet用于提供内容。

容器必须使用区分大小写字符串比较匹配。

下面详细描述容器的匹配过程：

1. 精确路径匹配。

例子：比如servletA 的url-pattern为 /test，servletB的url-pattern为 /* ，这个时候，如果我访问的url为http://localhost/test ，这个时候容器就会先进行精确路径匹配，发现/test正好被servletA精确匹配，那么就去调用servletA，也不会去理会其他的 servlet了。 

2. 最长路径匹配。

例子：servletA的url-pattern为/test/*，而servletB的url-pattern为/test/a/*，此时访问http://localhost/test/a时，容器会选择路径最长的servlet来匹配，也就是这里的servletB。 

3. 扩展匹配，如果url最后一段包含扩展，容器将会根据扩展选择合适的servlet。

例子：servletA的url-pattern：*.action 

4. 如果前面三条规则都没有找到一个servlet，容器会根据url选择对应的请求资源。
如果应用定义了一个default servlet，则容器会将请求丢给default servlet（什么是default servlet？后面会讲）。 

根据这个规则表，就能很清楚的知道servlet的匹配过程，所以定义servlet的时候也要考虑url-pattern的写法，以免出错。 
对于filter，不会像servlet那样只匹配一个servlet，因为filter的集合是一个链，所以只会有处理的顺序不同，而不会出现只选择一个filter。Filter的处理顺序和filter-mapping在web.xml中定义的顺序相同。

url-pattern详解，在web.xml文件中，以下语法用于定义映射： 

- 以”/’开头和以”/*”结尾的是用来做路径映射的。 （对应于第2条匹配规则）
- 以前缀”*.”开头的是用来做扩展映射的。 （对应于第3条匹配规则）
-  “/” 是用来定义default servlet映射的。 （对应于第4条匹配规则）
- 剩下的都是用来定义详细映射的。比如： /aa/bb/cc.action （对应于第1条匹配规则）

所以，为什么定义”/*.action”这样一个看起来很正常的匹配会错？因为这个匹配即属于路径映射，也属于扩展映射，导致容器无法判断。

下面详细讲述对于上面比配情况Servlet API当中方法的值情况：

```
                        |-- Context Path --|-- Servlet Path -|--Path Info--|
http://www.myserver.com     /mywebapp        /helloServlet      /hello
                        |-------- Request URI  ----------------------------|
```

记住下面三点:

Request URI = context path + servlet path + path info.
Context paths 和 servlet paths 以 '/' 开始，但是不以'/'结尾.
HttpServletRequest 提供3个方法 getContextPath(),getServletPath() 和getPathInfo() 来分别获取 context path, servlet path,  path info。
识别servlet路径 :

为了匹配一个servlet请求URI，servlet容器遵循一个简单的算法。 一旦确定了上下文路径，如果有的话，它会评估该请求URI的其余部分与在部署描述符中指定的servlet映射，按下列顺序。如果它找到一个匹配在任何步骤，它不采取下一个步骤。 

精确路径匹配（上面匹配的第一条规则）：在这种情况下，getPathInfo()为空。
最长路径匹配（上面匹配的第二条规则）： 如果有一个匹配，请求URI的匹配部分是getServletPath() 结果，其余部分是getPathInfo()。
扩展匹配（上面匹配的第三条规则）： 在这种情况下，完整的请求URI是getServletPath()和getPathInfo()为空。
默认servlet匹配（上面匹配的第四条规则）： 如果没有默认的servlet，它会发送未找到指示的servlet的错误消息。
通过例子来熟悉一下：

```
<servlet-mapping>
    <servlet-name>RedServlet</servlet-name>
    <url-pattern>/red/*</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>RedServlet</servlet-name>
    <url-pattern>/red/red/*</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>RedBlueServlet</servlet-name>
    <url-pattern>/red/blue/*</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>BlueServlet</servlet-name>
    <url-pattern>/blue/</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>GreenServlet</servlet-name>
    <url-pattern>/green</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>ColorServlet</servlet-name>
    <url-pattern>*.col</url-pattern>
</servlet-mapping>

```

```
Request URI                Servlet Used            Servlet Path        Path Info
/colorapp/red                RedServlet              /red                 null
/colorapp/red/               RedServlet              /red                 /
/colorapp/red/aaa            RedServlet              /red                 /aaa
/colorapp/red/blue/aa        RedBlueServlet          /red/blue            /aa
/colorapp/red/red/aaa        RedServlet              /red/red             /aaa
/colorapp/aa.col             ColorServlet            /aa.col              null
/colorapp/hello/aa.col       ColorServlet            /hello/aa.col        null
/colorapp/red/aa.col         RedServlet              /red                 /aa.col
/colorapp/blue               NONE(Error message)                          
/colorapp/hello/blue/        NONE(Error message)                          
/colorapp/blue/mydir         NONE(Error message)     
/colorapp/blue/dir/aa.col    ColorServlet            /blue/dir/aa.col     null  
/colorapp/green              GreenServlet            /green               null
```
 
