# Http基础

### http协议

#### 概述

http协议本身的设计其实很简单，只是在TCP/IP上建立的一种传数据，返回数据的协议，并规定了相应部分数据格式的规则，网络上讲的一大堆什么详解、完全解密，都是过度的加入了自己的想法（就和语文考试一样，其实作者没有那么多想法，不需要强行加想法进去）以及把太多功能性相关内容加入，最终让人闹不清主次。

#### 过程

整个http的过程十分简单，就是客户端发送一堆数据给服务器，服务器解析出想要的内容，然后返回一堆数据，客户端解析出想要的内容（其中怎么发、怎么保证数据不丢失，就是底层TCP/IP的事，不需要在这里理解）

#### 格式

##### request

* 主要包含三个内容
* 1.请求行：方法（GET、POST、PUT、DELETE等等） URI 协议版本
* 2.header：key:value格式
* 3.body：数据内容，可以是任何内容

##### response

* 主要包含三个内容
* 1.响应行：协议版本 状态码 状态描述
* 2.header：key:value格式
* 3.body：数据内容，可以是任何内容

#### Get与Post对比

* 1.Get在Url拼接参数，只能字符类型；Post参数放在body，可以接受多种数据格式
* 2.Get参数长度受限于url长度，Post参数长度不受限
* 3.安全性get更差，在url中，都可见；Post放在body中，一般不可见

#### http协议内约定的功能

* **对一些通用的功能进行约定，形成一种默认标准**

* 方法-想干嘛：GET，body不传数据；POST：body传数据；等等功能
* 状态码-得到的结果：1xx：指示信息；2xx：成功；3xx：重定向；4xx：客户端错误；5xx：服务器端错误
* header：对传递的数据内容进行约定或者附带上一些必要的约定信息
* Content-Type：约定post数据格式，包括json、表单、key1=value1&key2=value2等等格式
* Cache-Control：缓存功能
* Connection：连接是否连续
* Accept：描述接受的格式，image/gif,image/x-xbitmap等等
* Accept-Language：语言，zh-cn (CRLF)
* Accept-Encoding：编码gzip,deflate (CRLF)
* If-Modified-Since：缓存相关时间，Wed,05 Jan 2007 11:21:25 GMT (CRLF)
* Host：地址
* Last-Modified：最后修改时间
* Expires：过期时间长度
