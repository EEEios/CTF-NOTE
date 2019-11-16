# IP协议

参考链接：https://blog.csdn.net/qq_42058590/article/details/82918678

1. 版本号(4)：IPv4版本就是IPv4

2. 首部长度(4)：4位(0~15)，代表行数，基本单位为4字节（不够的会进行填充），如图，每行4个字节最小长度为20字节，二进制为0101，最大为6，二进制为1111.

3. 服务(8)：用于设置数据传输的优先级

4. 总长度(16)：数据包总长度=首部长度+数据长度

5. 标识(16)：IP在存储器中维持一个计数器，每产生一个数据包，计数器+1并赋值给标识字段。当数据包长度超过MTU时，标识字段的值被复制到所有数据报片的标识字段中，相同标识符的数据报会被重新组装成一个数据报

6. 标志(3)：如下

> | x    | DF                | MF                                                 |
> | ---- | ----------------- | -------------------------------------------------- |
> | x    | 为1时标识不能分片 | 为1时标识后面还有若干个数据包，0标识最后一个数据包 |

7. 片偏移(13)：分片后，该片在元分组中的相对位置，基本单位为8字节（不够的会进行填充）

   > 标志和片偏移
   >
   > ![1573802238097](C:\Users\E10S\AppData\Roaming\Typora\typora-user-images\1573802238097.png)

8. 生存时间(ttl, time to live, 8)：跳数，没经过一个路由器 -1，为0时该包被丢弃

9. 协议(8)：数据部分携带的协议

10. 首部校验和(166)：计算方法如下

    > ![img](https://img-blog.csdn.net/20181001232754493?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDU4NTkw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

11. IP源地址(32)：需要注意字节顺序
12. 目的地址(32)：同上
13. 可变选项

![img](https://img-blog.csdn.net/20181001210317800?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDU4NTkw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![1573802279875](C:\Users\E10S\AppData\Roaming\Typora\typora-user-images\1573802279875.png)