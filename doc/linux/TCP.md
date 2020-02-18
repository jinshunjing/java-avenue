## 基础知识
https://www.jianshu.com/writer#/notebooks/33672046/notes/40868185

## Linux命令
### 查看server socket
netstat -ltnp
### 查看socket
netstat -tnp

## 基础
- 抓包wireshark


## TCP状态
|状态 | 说明|
|------|------|
|CLOSED|关闭状态|
|LISTEN|服务端监听状态|
|SYN_SENT|客户端已发送SYN|
|SYN_RECV|服务端已接收SYN|
|ESTABLISHED|正常数据传输状态|
|FIN_WAIT_1|客户端已发送FIN，等待服务端确认|
|CLOSE_WAIT|服务端已接收FIN并且发送ACK，等待直接关闭连接|
|FIN_WAIT_2|客户端已接收ACK，等待服务端发送FIN|
|LAST_ACK|服务端已发送FIN，等待客户端确认|
|TIME_WAIT|客户端已接收FIN并且发生ACK，等待2MLS后关闭连接|
|CLOSING|两边同时发送FIN|

---

## 三次握手
+ 第一次握手：客户端发送连接请求报文（SYN=1，seq=x），进入状态SYN_SENT
+ 第二次握手：服务端接收到报文，发送确认报文（SYN=1，ACK=1, ack=x+1, seq=y)，进入状态SYN_RECV
+ 第三次握手：客户端接收到报文，发送确认报文（ACK=1, ack=y+1, seq=x+1），进入状态ESTABLISHED
+ 服务端接收到报文，进入状态ESTABLISHED

> 为什么还需要客户端再确认一次？
+ **防止失效的连接请求被服务端确认而导致的错误。**比如客户端发送了两次连接请求，后一个请求比前一个请求先到达服务端。服务端可能两个都确认，客户端可以忽略后一个确认。

> 服务端容易受到SYN攻击。服务端在第二次握手之后就分配资源。

## 四次挥手
+ 第一次挥手：客户端发送连接释放报文（FIN=1, seq=u），进入状态FIN_WAIT_1
+ 第二次挥手：服务端接收到报文，发送确认报文(ACK=1, ack=u+1, seq=v)，进入状态CLOSE_WAIT
+ 客户端接收到报文，进入状态FIN_WAIT_2
+ 第三次挥手：服务端发送连接释放报文(ACK=1, FIN=1, ack=u+1, seq=w)，进入状态LAST_ACK
+ 第四次挥手：客户端接收到报文，发送确认报文（ACK=1, ack=w+1），进入状态TIME_WAIT，等待2MLS后进入状态CLOSED
+ 服务端接收到报文，进入状态CLOSED

> 为什么还需要客户端再等待2MLS？
+ **保证双工通道正常关闭。**保证客户端的确认报文能够到达服务端。客户端可以超时重传。如果服务端接收不到ACK，则会再次发送FIN，导致错误。
+ **防止失效的报文出现。**如果客户端又发起了新连接，并且新连接与之前的老连接使用相同的端口，如果之前的连接还有数据滞留在网络，那么这些数据会与新连接的数据发生混淆。

---

## TCP KeepAlive
默认KeepAlive不开启。如果长时间不传输数据报文，不知道连接是否有效。
|参数|说明|
|------|------|
|tcpkeepalivetime|TCP连接在多少秒没有数据报文传输之后发送探测报文|
|tcpkeepaliveintvl|两次探测报文间隔多少秒|
|tcpkeepaliveprobes|探测的次数|

## 重传
- 超时重传：超时还未接收ACK，则重传数据包
- 快速重传：如果接收方收到了后面的数据包但是没有收到前面的，则发送3个ACK确认请求重传，发送方直接重传，不需要等待超时

## TCP报文段首部
- 源端口(16)，目的端口(16)
- 序号(32)
- 确认号(32)
- 数据偏移(4)，保留(6)，URG/ACK/PSH/RST/SYN/FIN, 窗口(16)
- 校验和(16)，紧急指针(16)
- 选项，填充

### TCP选项
- 选项结束字段EOP(0x00)
- 无操作字段NOP(0x01)
- 

## TCP listen backlog
- 内核维护两个队列
- 未完成队列：接收到SYN建立连接请求，处于SYN_RECV状态
- 已完成队列：处于ESTABLISHED状态
- backlog曾被定义为两个队列总和的最大值，也曾将backlog的1.5倍作为未完成队列的最大长度

- ss -l
- Recv-Q
- Send-Q 128

## accept
- accept 发生在三次握手之后，从已完成队列中取出一项返回

## TCP vs UDP 
- 有连接，无连接
- 可靠传输（确认和重传），不可靠
- 拥塞控制，滑动窗口，保证传输的质量
- TCP面向字节流，会分段；UDP不会











