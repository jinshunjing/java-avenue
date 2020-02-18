# Http

## 错误码
401: Unauthorized
500: Internal server error
502：Bad Gateway 
503: Service Unavailable
504: Gateway timeout

## Http 2.0

## 多路复用
> 只有一个tcp连接？

### 并行双向字节流

## 二进制

## 帧
Frame | 说明
:------ | :------
SETTINGS | 设置帧
WINDOW_UPDATE | 流量控制帧
HEADER | 头帧
DATA | 数据帧
PUSH_PROMISE | 服务端推送消息帧
GOAWAY | 连接关闭帧

* Settings
SETTINGS_ENABLE_PUSH: 0禁止服务器推送
SETTINGS_MAX_CONCURRENT_STREAMS：允许打开流的最大值

* Header
END_STREAM：表示流的结束

### 消息头压缩
不仅仅Data帧，Header帧也可以gzip压缩

## 服务器推送
