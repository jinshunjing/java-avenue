# Tomcat

## Tomcat Connector 原理
- accept队列：完成三次握手的连接，放入accept队列。该队列的大小由acceptCount决定。如果队列已满，则拒绝。
- 任意时刻接收和处理的最大连接数：通过maxConnections配置，默认是10000。
- 服务器可以同时连接的连接数为：maxConnections + acceptCount
- 线程池：最大的线程数通过maxThreads配置，默认是200。


## Tomcat Connector 配置
<Connector
 executor="tomcatThreadPool"
 port="8080"
 protocol="org.apache.coyote.http11.Http11Nio2Protocol"
 connectionTimeout="60000"
 maxConnections="10000"
 redirectPort="8443"
 enableLookups="false"
 acceptCount="100"
 maxPostSize="10485760"
 maxHttpHeaderSize="8192"
 compression="on"
 disableUploadTimeout="true"
 compressionMinSize="2048"
 acceptorThreadCount="2"
 compressableMimeType="text/html,text/plain,text/css,application/javascript,application/json,application/x-font-ttf,application/x-font-otf,image/svg+xml,image/jpeg,image/png,image/gif,audio/mpeg,video/mp4"
 URIEncoding="utf-8"
 processorCache="20000"
 tcpNoDelay="true"
 connectionLinger="5"
 server="Server Version 11.0"
 />

参数解释：

protocol：Tomcat 8 设置 nio2 更好：org.apache.coyote.http11.Http11Nio2Protocol
protocol：Tomcat 6 设置 nio 更好：org.apache.coyote.http11.Http11NioProtocol
protocol：Tomcat 8 设置 APR 性能飞快：org.apache.coyote.http11.Http11AprProtocol 更多详情：《Tomcat 8.5 基于 Apache Portable Runtime（APR）库性能优化》

acceptorThreadCount：用于接受连接的线程数量。增加这个值在多CPU的机器上,尽管你永远不会真正需要超过2。 也有很多非维持连接,您可能希望增加这个值。默认值是1。

maxConnections：这个值表示最多可以有多少个socket连接到tomcat上
acceptCount：当tomcat起动的线程数达到最大时，接受排队的请求个数，默认值为100。

processorCache：协议处理器缓存的处理器对象来提高性能。 该设置决定多少这些对象的缓存。-1意味着无限的,默认是200。 如果不使用Servlet 3.0异步处理,默认是使用一样的maxThreads设置。 如果使用Servlet 3.0异步处理,默认是使用大maxThreads和预期的并发请求的最大数量(同步和异步)。

connectionTimeout：Connector接受一个连接后等待的时间(milliseconds)，默认值是60000。
enableLookups：禁用DNS查询
maxPostSize：设置由容器解析的URL参数的最大长度，-1(小于0)为禁用这个属性，默认为2097152(2M) 请注意， FailedRequestFilter 过滤器可以用来拒绝达到了极限值的请求。
maxHttpHeaderSize：http请求头信息的最大程度，超过此长度的部分不予处理。一般8K。
compression：是否启用GZIP压缩 on为启用（文本数据压缩） off为不启用， force 压缩所有数据
disableUploadTimeout：这个标志允许servlet容器使用一个不同的,通常长在数据上传连接超时。 如果不指定,这个属性被设置为true,表示禁用该时间超时。
compressionMinSize：当超过最小数据大小才进行压缩
compressableMimeType：配置想压缩的数据类型
URIEncoding：网站一般采用UTF-8作为默认编码。
tcpNoDelay：如果设置为true,TCP_NO_DELAY选项将被设置在服务器套接字,而在大多数情况下提高性能。这是默认设置为true。
connectionLinger：秒数在这个连接器将持续使用的套接字时关闭。默认值是 -1,禁用socket 延迟时间。
server：隐藏Tomcat版本信息，首先隐藏HTTP头中的版本信息


## Tomcat 线程池配置
<Executor
 name="tomcatThreadPool"
 namePrefix="catalina-exec-"
 maxThreads="500"
 minSpareThreads="30"
 maxIdleTime="60000"
 prestartminSpareThreads = "true"
 maxQueueSize = "100"
/>
参数解释：

maxThreads：最大并发数，默认设置 200，一般建议在 500 ~ 800，根据硬件设施和业务来判断
minSpareThreads：Tomcat 初始化时创建的线程数，默认设置 25
maxIdleTime：如果当前线程大于初始化线程，那空闲线程存活的时间，单位毫秒，默认60000=60秒=1分钟。
prestartminSpareThreads：在 Tomcat 初始化的时候就初始化 minSpareThreads 的参数值，如果不等于 true，minSpareThreads 的值就没啥效果了
maxQueueSize：最大的等待队列数，超过则拒绝请求

