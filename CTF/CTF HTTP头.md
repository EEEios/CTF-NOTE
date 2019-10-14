# CTF HTTP头

#### ip伪造

X-*Forwarded*-For ；

client-ip ；

REMOTE_ADDR ；

> `REMOTE_ADDR`不可以显式的伪造，虽然可以通过代理将ip地址隐藏，但是这个地址仍然具有参考价值，因为它就是与你的服务器实际连接的ip地址。

#### 维持tcp通道

Connection:Keep-Alive;

Pipeline; （浏览器中默认不开启Pipeline，但服务器都提供了对Pipeline的支持）

### 请求体相关

Content-Length - 请求体长度

Transfer-Encoding - 报文格式

Content-Encoding - 

> TE和CE一般相辅相成