# 在浏览器中输入URL之后的返回流程。
    + redirect跳转
    + App Cache 应用缓存
    + DNS 域名解析
    + TCP 创建TCP链接
    + Request 发送请求
    + Response 接受响应
---

# http的三次握手
### 在客户端和服务器之间进行一个http 的请求和发送的过程中，是需要创建一个TCP connection
    + 首先客户端发起一个我要创建一个链接的一个数据包，发送到服务端，这里面会有一个标志位SYN=1，后面会发送一个Seq=x 一般x=1。
    + 服务端接收到这个链接之后，会开启一个TCP socket的端口，之后返回给客户端里面第一个也是标志位 SYN=1，然后ACK=X+1，Seq=Y
    + 然后客户端再次发给ACK=Y+1,Seq=Z
    #### 三次握手主要是为了规避网络延时所导致的服务器开销问题

# URL URI URN
    + URI：Uniform Resource Identifier/统一资源标志符 定位某一个特定的资源而去设置的，用来标识互联网唯一的资源他包含了URL 和URN
    + URL：Uniform Resource Locator/统一的资源定位器
    + URN：永久统一定为符号

# html 页面重定向
    + <meta http-equiv="refresh" content="0;url=http://www.baidu.com" />

# nginx
    + nginx 所在目录 /usr/local/etc/nginx

    + 重启动nginx服务
        brew services restart nginx
    + 停止nginx服务
        brew services stop nginx
    + 生成https密钥公钥命令
        openssl req -x509 -newkey rsa:2048 -nodes -sha256 -keyout localhost-privkey.pem -out localhost-cert.pem
