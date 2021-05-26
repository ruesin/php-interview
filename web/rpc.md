# RPC

RPC（Remote Procedure Call）：远程过程调用，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的思想。

比如两个不同的服务 A、B 部署在两台不同的机器上，如果服务 A 想要调用服务 B 中的某个方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

RPC 是一种设计思想而非一种规范或协议，是为了解决不同服务之间的调用问题，完整的 RPC 实现一般会包含有传输协议和序列化协议，可以通过TCP、UDP、Socket、HTTP等方式通信。

在一个典型 RPC 的使用场景中，包含了服务发现、负载、容错、网络传输、序列化等组件，其中“RPC 协议”就指明了程序如何进行网络传输和序列化。

![rpc](../images/web/rpc_1.jpg)

“RPC 协议”的核心功能主要有 5 个部分组成：客户端、客户端 Stub、网络传输模块、服务端 Stub、服务端。主要涉及到三个关键技术点：服务寻址、数据流的序列化和反序列化、网络传输。

## 服务寻址

服务寻址可以使用 Call ID 映射。在本地调用中，函数体是直接通过函数指针来指定的，但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。

所以在 RPC 中，所有的函数都必须有自己的一个 ID。这个 ID 在所有进程中都是唯一确定的。

客户端在做远程过程调用时，必须附上这个 ID。然后我们还需要在客户端和服务端分别维护一个函数和Call ID的对应表。两者的表不一定需要完全相同，但相同的函数对应的Call ID必须相同。

当客户端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

## 序列化和反序列化

在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。

这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。

序列化转换可以自己写，也可以使用 Protobuf、FlatBuffers 等，可以实现二进制流和对象之间转换即可。

## 网络传输

远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。

网络传输层需要把 Call ID 和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。


只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分RPC框架都使用TCP协议，但其实UDP也可以，而gRPC干脆就用了HTTP2。

TCP 的连接是最常见的，简要分析基于 TCP 的连接：通常 TCP 连接可以是按需连接(需要调用的时候就先建立连接，调用结束后就立马断掉)，也可以是长连接(客户端和服务器建立起连接之后保持长期持有，不管此时有无数据包的发送，可以配合心跳检测机制定期检测建立的连接是否存活有效)，多个远程过程调用共享同一个连接。

### 基于 TCP 协议的 RPC 调用

由调用方与提供方建立 Socket 连接，并由调用方通过 Socket 将需要调用的接口名称、方法名称和参数序列化后传递给提供方，提供方反序列化后再利用反射调用相关的方法，然后将结果返回给调用方。

### 基于 HTTP 协议的 RPC 调用

该方法更像是访问网页一样，只是它的返回结果更加单一简单。

由调用方向提供方发送请求，这种请求的方式可能是 GET、POST、PUT、DELETE 等中的一种，提供方可能会根据不同的请求方式做出不同的处理，或者某个方法只允许某种请求方式。

而调用的具体方法则是根据 URL 进行方法调用，而方法所需要的参数可能是对服务调用方传输过去的 XML 数据或者 JSON 数据解析后的结果，提供方返回 JOSN 或者 XML 的数据结果。

### TCP 和 HTTP 对比

由于 TCP 协议处于协议栈的下层，能够更加灵活地对协议字段进行定制，减少网络开销，提高性能，实现更大的吞吐量和并发数。

但是需要更多关注底层复杂的细节，实现的代价更高。同时对不同平台，如安卓，iOS 等，需要重新开发出不同的工具包来进行请求发送和相应解析，工作量大，难以快速响应和满足用户需求。

基于 HTTP 协议实现的 RPC 则可以使用 JSON 和 XML 格式的请求或响应数据。

而 JSON 和 XML 作为通用的格式标准(使用 HTTP 协议也需要序列化和反序列化，不过这不是该协议下关心的内容，成熟的 Web 程序已经做好了序列化内容)，开源的解析工具已经相当成熟，在其上进行二次开发会非常便捷和简单。

但是由于 HTTP 协议是上层协议，发送包含同等内容的信息，使用 HTTP 协议传输所占用的字节数会比使用 TCP 协议传输所占用的字节数更高。

因此在同等网络下，通过 HTTP 协议传输相同内容，效率会比基于 TCP 协议的数据效率要低，信息传输所占用的时间也会更长，当然压缩数据，能够缩小这一差距。