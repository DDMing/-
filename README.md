# SoftwareReuseDiscuss
##用户登录后始终在线，考虑低带宽、不稳定网络。
------------------
- **长连接心跳机制**:
- **消息不遗漏**:
- **消息不重复**:
- **消息压缩**:

##长连接心跳机制
  - **出现原因** ：在长连接下，有可能很长一段时间都没有数据往来。理论上说，这个连接是一直保持连接的，但是实际情况中，如果中间节点出现什么故障是难以知道的。。典型应用是app的推送的服务。
  - **机制原理** : 维护一个长连接都需要心跳机制。心跳机制便是客户端发送一个心跳给服务器，服务器给客户端一个心跳应答，这样就形成客户端服务器的一次完整的握手，这个握手是让双方都知道他们之间的连接是没有断开，客户端是在线的。如果超过一个时间的阈值，客户端没有收到服务器的应答，或者服务器没有收到客户端的心跳，那么对客户端来说则断开与服务器的连接重新建立一个连接，对服务器来说只要断开这个连接即可。
  - **通常实现的两种技术**
    - **1** 应用层自己实现的心跳包。
    - **2** 在TCP/IP协议层上，使用SO_KEEPALIVE套接字选项
  - **TCP keepalive** : [RFC1122#TCP Keep-Alives](https://tools.ietf.org/html/rfc1122#page-101)
    - **1** TCP Keepalive虽不是标准规范，但操作系统一旦实现，默认情况下须为关闭，可以被上层应用开启和关闭。
    - **2** TCP Keepalive必须在没有任何数据（包括ACK包）接收之后的周期内才会被发送，允许配置，默认值不能够小于2个小时
    - **3** 不包含数据的ACK段在被TCP发送时没有可靠性保证，意即一旦发送，不确保一定发送成功。系统实现不能对任何特定探针包作死连接对待
    - **4** 规范建议keepalive保活包不应该包含数据，但也可以包含1个无意义的字节，比如0x0。
  - **Http Keep-Alive** : HTTP协议采用“请求-应答”模式，当使用普通模式，即非KeepAlive模式时，每个请求/应答客户和服务器都要新建一个连接完成之后立即断开连接（HTTP协议为无连接的协议）；当使用Keep-Alive模式（又称持久连接、连接重用）时，Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。启用Keep-Alive模式肯定更高效，性能更高。因为避免了建立/释放连接的开销。![有无keep-alive的比较](https://www.byvoid.com/upload/wp/2011/07/450px-HTTP_persistent_connection.svg_.png)

##**消息不重复** 与 **消息不遗漏**
  - 以[`MQTT`](http://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels)为例。
  - **MQTT**协议是为大量计算能力有限，且工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议。
  - QoS(Quality Of Service)分为三个等级
    - At most once (0)
    - At least once (1)
    - Exactly once (2)
  - *消息不重复*以及*不遗漏*则采用最高等级的QoS 2，基本原理类似于TCP/IP的建立。![Exactly Once](http://www.hivemq.com/wp-content/uploads/publish_qos2_flow.png)

####**BTW** MQTT 是如何处理[长连接](http://www.hivemq.com/blog/mqtt-essentials-part-10-alive-client-take-over)的问题的。
  - MQTT是基于TCP连接的，TCP的链接是可靠的。
  - MQTT依然是利用心跳包的机制实现keep-alive

> It is the responsibility of the Client to ensure that the interval between Control Packets being sent does not exceed the Keep Alive value. In the absence of sending any other Control Packets, the Client MUST send a PINGREQ Packet.

## **消息压缩**
依然以MQTT为例，从[编码和压缩](https://source.sierrawireless.com/airvantage/av/reference/hardware/protocols/mqtt-ts/)两个方面诠释这个问题
  - **编码:**
    - MQTT在编码上采用[CBOR](https://tools.ietf.org/html/rfc7049)，是基于json数据格式的编码
    - *整数*采用[差分编码](https://en.wikipedia.org/wiki/Delta_encoding)
    - 采用类似GPS坐标的，修正坐标的方法，来减小*浮点数*的大小
    - *时间戳*只精确到毫秒，即去掉最后两位数的时间精度。
    
  - **压缩**
    - MQTT压缩算法采用[zlib](https://tools.ietf.org/html/rfc1950)

###END
--------------------------
#####`4月10日`前提交到Github上
