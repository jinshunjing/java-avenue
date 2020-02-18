
## SSL证书

### 申请证书
* SSL Server生成私钥/公钥：server.key/server.pub
* SSL Server生成server.req/server.csr，包含域名/公钥/申请者
* CA用ca.key签名server.req，生成server.crt(server.pub)，
* SSL Client用ca.crt(ca.pub)解密server.crt

> server.crt无法伪造，因为它是由ca.key生成，并且只能被ca.pub解密

### 单向认证
* SSL Server: server.key, server.crt
* SSL Client: ca.crt

### 双向认证
* SSL Server: server.key, server.crt, ca.crt
* SSL Client: client.key, client.crt, ca.crt

### 认证流程
* client发送明文随机数random_c
* server发送明文随机数random_s，加密算法
* client生成随机数pre-master，用server.pub加密后传输；并用对称密钥(random_c, random_s, pre-master)签名之前所有接受信息的hash
* server计算对称密钥(random_c, random_s, pre-master), 并验证签名
* 握手结束，使用对称密钥加密

### 单向认证
- 客户端向服务端发送SSL协议版本号，加密算法种类，随机数
- 服务端给客户端返回SSL协议版本号，加密算法种类，随机数，同时也返回服务器的证书
- 客户端验证服务器的合法性：1. 证书是否过期；2. 发现证书的CA是否可信；3. 公钥是否能够解开证书中的签名；4. 服务器的域名是否与证书上的域名匹配
- 客户端向服务端发送自己能支持的对称加密方案
- 服务端选择加密程度最高的加密方案
- 服务端用明文的方式把加密方案返回给客户端
- 客户端使用选中的加密方案生成随机码，使用服务器的公钥加密，然后发送给服务器
- 服务端用私钥解密，获得对称密钥
- 之后使用对称密钥加密

### 双向认证
- 客户端向服务端发送SSL协议版本号，加密算法种类，随机数
- 服务端给客户端返回SSL协议版本号，加密算法种类，随机数，同时也返回服务器的证书
- 客户端验证服务器的合法性：1. 证书是否过期；2. 发现证书的CA是否可信；3. 公钥是否能够解开证书中的签名；4. 服务器的域名是否与证书上的域名匹配

- 客户端向服务端发送自己的证书
- 服务端验证客户端的证书

- 客户端向服务端发送自己能支持的对称加密方案
- 服务端选择加密程度最高的加密方案

- 服务端用客户端的公钥加密选中的加密方案，并返回给客户端
- 客户端用自己的私钥解密选择的加密方案

- 客户端使用选中的加密方案生成随机码，使用服务器的公钥加密，然后发送给服务器
- 服务端用私钥解密，获得对称密钥
- 之后使用对称密钥加密




