# TCP
https://www.jianshu.com/writer#/notebooks/33672046/notes/40868185

## Linux命令

### 查看server socket
netstat -ltnp
### 查看socket
netstat -tnp


## 监控工具
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

## TCP listen backlog
- 内核维护两个队列
- 未完成队列：接收到SYN建立连接请求，处于SYN_RECV状态
- 已完成队列：处于ESTABLISHED状态
- backlog曾被定义为两个队列总和的最大值，也曾将backlog的1.5倍作为未完成队列的最大长度

## accept
- accept 发生在三次握手之后，从已完成队列中取出一项返回

## TCP vs UDP 
- 有连接，无连接
- 可靠传输（确认和重传），不可靠
- 拥塞控制，滑动窗口，保证传输的质量
- TCP面向字节流，会分段；UDP不会

## TCP参数
net.ipv4.ip_local_port_range = 1024 65000   
表示用于向外连接的端口范围。缺省情况下很小：32768到60999，改为1024到65000。

net.ipv4.tcp_syn_retries = 3    
在内核放弃建立连接之前发送SYN包的数量  
net.ipv4.tcp_synack_retries = 2  
为了打开对端的连接，内核需要发送一个SYN并附带一个回应前面一个SYN的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK包的数量。  
net.ipv4.tcp_max_syn_backlog = 262144  
表示SYN队列长度，默认1024，改成8192，可以容纳更多等待连接的网络连接数。记录那些尚未收到客户端确认信息的连接请求的最大值。对于有128M内存的系统而言，缺省值是1024，小内存的系统则是128。  
net.ipv4.tcp_syncookies = 0   
表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭。此参数是为了防止洪水攻击的，但对于大并发系统，要禁用此设置  

net.ipv4.tcp_fin_timeout = 10  
表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间  
net.ipv4.tcp_tw_recycle = 1  
表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭  
net.ipv4.tcp_tw_reuse = 1  
表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭  
net.ipv4.tcp_max_tw_buckets = 5000  
表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默认为180000，改为5000。 

net.ipv4.tcp_keepalive_time = 30  
当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时  
net.ipv4.tcp_keepalive_probes = 3  
如果对方不予应答，探测包的发送次数  
net.ipv4.tcp_keepalive_intvl = 15  
keepalive探测包的发送间隔  

net.ipv4.tcp_max_orphans = 262144  
系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅是为了防止简单的DoS攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)  

net.ipv4.tcp_timestamps = 1  
时间戳可以避免序列号的卷绕。一个1Gbps的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。

